<template>
  <div class="template-select">
    <select v-model="selectedNoType">
      <option v-for="noType in noTypes" :key="noType" :value="noType">{{ noType }}</option>
    </select>
    <table class="template-table">
      <thead>
        <tr>
          <th>选择</th>
          <th>主类别＊</th>
          <th>子类别＊</th>
          <th>名称</th>
          <th>商家</th>
          <th>项目</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(template, index) in templates" :key="hash(template)">
          <!-- <td><input type="checkbox" :checked="isSelected(index)" @click.stop /></td> -->
          <td><input type="radio" :value="index" v-model="selectedRow" name="templateSelection" /></td>
          <td>{{ template["主类别＊"] }}</td>
          <td>{{ template["子类别＊"] }}</td>
          <td>{{ template["名称"] }}</td>
          <td>{{ template["商家"] }}</td>
          <td>{{ template["项目"] }}</td>
        </tr>
      </tbody>
    </table>
  </div>
  <button @click="updateType">更新类别</button>
</template>
  
<script>
import axios from 'axios'
import CryptoJS from 'crypto-js';
export default {
  props: { backend_url: { type: String, default: "" } },
  data() {
    return {
      noTypes: [],
      templates: [],
      selectedRow: null,
      selectedNoType: null,
    }
  },
  methods: {
    hash(obj) {
      const str = JSON.stringify(obj);
      return CryptoJS.SHA256(str).toString();
    },
    getNoTypes() {
      axios.get(`${this.backend_url}/get_no_types`)
        .then((response) => {
          if (!response.data.success) {
            alert(response.data.error);
            return;
          }
          this.noTypes = response.data.no_type;
          this.templates = response.data.templates;
        })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
    updateType() {
      if (this.selectedRow == null) {
        alert("请选择模板");
        return;
      }
      if (this.selectedNoType == null) {
        alert("请选择类别");
        return;
      }
      const template = this.templates[this.selectedRow];
      axios.post(`${this.backend_url}/update_types`, {
        [this.selectedNoType]: template
      }).then((response) => {
        if (response.data.success) {
          this.getNoTypes();
          this.selectedNoType = null;
          this.selectedRow = null;
        } else {
          alert(response.data.error);
        }
      })
        .catch(err => {
          console.error('Error submitting data', err)
        });
    },
  },
  created() {
    this.getNoTypes();
  }
}

</script>

<style scoped>
.template-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 16px;
  user-select: none;
}

.template-table th,
.template-table td {
  border: 1px solid #DDD;
  padding: 8px;
}

.template-table th {
  background-color: #f2f2f2;
  color: black;
  text-align: left;
}
</style>