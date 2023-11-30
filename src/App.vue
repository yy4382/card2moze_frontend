<template>
  <div id="app">
    <div><label for="backend-url">Backend URL:</label>
      <input id="backend-url" v-model="backend_url" type="text">
    </div>
    <button @click="toggleSelectionMode">
      {{ isRangeSelectMode ? 'Single Select Mode' : 'Range Select Mode' }}
    </button>
    <button @click="downloadSelected">Submit Selected Items</button>
    <button @click="updateEntries">Update Entries</button>
    <button @click="getEntries">Get Entries</button>
    <table class="custom-table">
      <thead>
        <tr>
          <th>Name</th>
          <th>Amount</th>
          <th>Time</th>
          <th>Balance</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in items" :key="index" :class="{ 'selected-row': isSelected(index) }"
          @click="handleRowClick(index, $event)" @touchstart="startLongPressTimer(index, $event)"
          @touchend="clearLongPressTimer" @touchmove="clearLongPressTimer">
          <!-- <td><input type="checkbox" :checked="isSelected(index)" @click.stop /></td> -->
          <td>{{ item.name }}</td>
          <td>{{ item.amount }}</td>
          <td>{{ item.time }}</td>
          <td>{{ item.balance }}</td>
        </tr>
      </tbody>
    </table>
    <VueDatePicker v-model="range" range />
    <button @click="downloadByDate">获取选定时间内的消费记录csv</button>
  </div>
</template>

<script>
import axios from 'axios'
import VueDatePicker from '@vuepic/vue-datepicker';
import '@vuepic/vue-datepicker/dist/main.css'

export default {
  components: {
    VueDatePicker
  },
  data() {
    return {
      items: [],
      selectedItems: [],
      selectedIndices: [],
      lastSelectedIndex: null,
      range: [new Date(), new Date()],
      longPressTimer: null,
      longPressDelay: 1000,
      isRangeSelectMode: false,
      rangeStartIndex: null,
      backend_url: null
    }
  },
  watch: {
    items() {
      this.selectedIndices = [];
      this.selectedItems = [];
    }
  },
  methods: {
    isSelected(index) {
      return this.selectedIndices.includes(index);
    },
    handleRowClick(index, event) {
      if (this.isRangeSelectMode) {
        // 如果已经开始一个范围选择，就选择从开始到当前行之间的所有行
        // 否则，设置当前行为范围选择的开始
        if (this.rangeStartIndex !== null) {
          this.selectedIndices = [];
          for (let i = Math.min(this.rangeStartIndex, index); i <= Math.max(this.rangeStartIndex, index); i++) {
            this.selectedIndices.push(i);
          }
          // 清除范围选择开始
          this.rangeStartIndex = null;
        } else {
          this.rangeStartIndex = index;
          this.selectedIndices = [index];
        }
      } else {
        if (event.shiftKey && this.lastSelectedIndex !== null) {
          let start = Math.min(index, this.lastSelectedIndex);
          let end = Math.max(index, this.lastSelectedIndex);
          for (let i = start; i <= end; i++) {
            if (!this.selectedIndices.includes(i)) {
              this.selectedIndices.push(i);
            }
          }
        } else if (event.ctrlKey || event.metaKey) {
          const position = this.selectedIndices.indexOf(index);
          if (position > -1) {
            this.selectedIndices.splice(position, 1);
          } else {
            this.selectedIndices.push(index);
          }
        } else {
          this.selectedIndices = [index];
        }
        this.lastSelectedIndex = index;
      }
      this.selectedItems = this.selectedIndices.map(i => this.items[i]);
    },
    startLongPressTimer(index, event) {
      this.clearLongPressTimer();
      this.longPressTimer = setTimeout(() => {
        event.preventDefault();
        this.handleLongPress(index);
      }, this.longPressDelay);
    },
    clearLongPressTimer() {
      clearTimeout(this.longPressTimer);
      this.longPressTimer = null;
    },
    handleLongPress(index) {
      const position = this.selectedIndices.indexOf(index);
      if (position > -1) {
        this.selectedIndices.splice(position, 1);
      } else {
        this.selectedIndices.push(index);
      }
      this.selectedItems = this.selectedIndices.map(i => this.items[i]);
    },
    toggleSelectionMode: function () {
      this.isRangeSelectMode = !this.isRangeSelectMode;
      // 清空选择
      this.selectedIndices = [];
      this.selectedItems = [];
      this.rangeStartIndex = null;
    },
    getEntries() {
      axios.get(`${this.backend_url}/get_entries`)
        .then(response => {
          this.items = response.data;
        })
        .catch(error => {
          console.log('Error retrieving data', error);
        });
    },
    downloadSelected() {
      const payload = this.selectedItems;
      axios({
        url: `${this.backend_url}/get_csv`,
        method: 'POST',
        responseType: 'blob', // important
        data: payload,
      })
        .then((response) => {
          const url = window.URL.createObjectURL(new Blob([response.data]));
          const link = document.createElement('a');
          link.href = url;
          const contentDisposition = response.headers['content-disposition'];
          let fileName = 'unknown';
          if (contentDisposition) {
            const matches = contentDisposition.match(/filename\*=(UTF-8''|'')(.*?)$/);
            if (matches && matches.length > 2) {
              fileName = decodeURIComponent(matches[2]);
            } else {
              fileName = contentDisposition.split('filename*=')[1];
            }
            console.log(contentDisposition);
            console.log(fileName);
          }
          link.setAttribute('download', fileName);
          document.body.appendChild(link);
          link.click();
        })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
    updateEntries() {
      axios.get(`${self.backend_url}/update_entries`)
        .then(response => {
          if (response.data.success) {
            this.getEntries();
          } else {
            console.log(response.data.error);
          }
        })
        .catch(error => {
          console.log('Error retrieving data', error);
        });
    },
    downloadByDate() {
      const start_time = this.range[0].getTime();
      const end_time = this.range[1].getTime();
      axios.get(`${self.backend_url}/get_csv`, {
        responseType: 'blob', // important
        params: {
          start_time: start_time,
          end_time: end_time,
          timestamp: true
        }
      }).then((response) => {
        const url = window.URL.createObjectURL(new Blob([response.data]));
        const link = document.createElement('a');
        link.href = url;
        const contentDisposition = response.headers['content-disposition'];
        let fileName = 'unknown';
        if (contentDisposition) {
          const matches = contentDisposition.match(/filename\*=(UTF-8''|'')(.*?)$/);
          if (matches && matches.length > 2) {
            fileName = decodeURIComponent(matches[2]);
          } else {
            fileName = contentDisposition.split('filename*=')[1];
          }
          console.log(contentDisposition);
          console.log(fileName);
        }
        link.setAttribute('download', fileName);
        document.body.appendChild(link);
        link.click();
      })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
  },
  created() {
    this.getEntries();
  }
}
</script>

<style scoped>
.custom-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 16px;
  user-select: none;
}

.custom-table th,
.custom-table td {
  border: 1px solid #DDD;
  padding: 8px;
}

.custom-table th {
  background-color: #f2f2f2;
  color: black;
  text-align: left;
}

.selected-row {
  background-color: #aaa;
}
</style>