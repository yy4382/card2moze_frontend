<template>
  <div>
    <button @click="getEntries" class="functionalButtons">获取条目</button>
    <button @click="updateEntries" class="functionalButtons">请求更新条目</button>
    <button @click="downloadSelected" class="functionalButtons">下载所选条目的CSV</button>
  </div>
  <div style="display: flex; justify-content: right; margin-bottom: 5px;">
    <label for="toggleMode">
      {{ isRangeSelectMode ? '多选模式（移动端）' : '单选模式（仍可使用⌘和⇧）' }}
    </label>
    <button @click="toggleSelectionMode" id="toggleMode">
      切换模式
    </button>
  </div>
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
</template>
  
<script>
import axios from 'axios'

export default {
  emits: ['csvDownload'],
  props: { backend_url: { type: String, default: "" } },
  // components: {
  //   VueDatePicker
  // },
  data() {
    return {
      items: [],
      selectedItems: [],
      selectedIndices: [],
      lastSelectedIndex: null,
      longPressTimer: null,
      longPressDelay: 1000,
      isRangeSelectMode: false,
      rangeStartIndex: null,
      // backend_url: ""
    }
  },
  watch: {
    items() {
      this.selectedIndices = [];
      this.selectedItems = [];
      localStorage.setItem('items', JSON.stringify(this.items));
    }
  },
  methods: {


    // 表格选择处理

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
      axios.post(`${this.backend_url}/get_csv`, payload)
        .then(response => {
          if (response.data.success) {
            this.$emit('csvDownload', response.data);
            // this.downloadCsv(response)
          } else if (response.data.error == 'No entry found') {
            console.log(response.data.error);
          } else if (response.data.error == 'Request for types') {
            const noTypeStores = response.data.stores;
            const noTypeStoresString = noTypeStores.join(", ");
            alert("need types:" + noTypeStoresString);
          } else {
            console.log(response.data.error);
          }
        })
        .catch(error => {
          console.log('Error submitting data', error);
        });
    },
    updateEntries() {
      axios.get(`${this.backend_url}/update_entries`)
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
  },
  created() {
    const items = localStorage.getItem('items');
    console.log(items)
    if (items === null) {
      // console.log(this.items)
      this.getEntries();
    } else {
      this.items = JSON.parse(items);
    }
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