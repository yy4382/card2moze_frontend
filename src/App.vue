<template>
  <div class="page-container">
    <div class="be-input"><label for="backend-url">Backend URL:</label>
      <input id="backend-url" v-model="backend_url" type="text">
    </div>
    <div class="content" >
      <TableSelect v-if="selectedTab == 0" :backend_url="backend_url" @csvDownload="downloadCSV" />
      <DateSelect v-if="selectedTab == 1" :backend_url="backend_url" @csvDownload="downloadCSV" />
      <TypeUpdate v-if="selectedTab == 2" :backend_url="backend_url" />
      <SettingsPage v-if="selectedTab == 3" :backend_url="backend_url" />
    </div>
    <TabBar class="tab-bar" @tabSelected="handleTabSelected" />
  </div>
</template>

<script>
import TabBar from './components/TabBar.vue';
import TableSelect from './components/TableSelect.vue';
import DateSelect from './components/DateSelect.vue';
import SettingsPage from './components/SettingsPage';
import TypeUpdate from './components/TypeUpdate.vue';

export default {
  components: {
    TabBar,
    TableSelect,
    DateSelect,
    SettingsPage,
    TypeUpdate
},
  data() {
    return {
      selectedTab: '',
      backend_url: ""
    };
  },
  methods: {
    handleTabSelected(tab) {
      console.log('Tab selected:', tab);
      this.selectedTab = tab;
    },
    downloadCSV(data) {
      const csv = data.csv;
      const start_time = data.start_time;
      const end_time = data.end_time;
      // Create a Blob object from the CSV string
      const blob = new Blob([csv], { type: 'text/csv' });

      // Create a URL for the Blob object
      const url = URL.createObjectURL(blob);

      // Create a link element
      const link = document.createElement('a');
      link.href = url;
      link.download = `${start_time} -- ${end_time}.csv`;

      // Append the link to the document body
      document.body.appendChild(link);

      // Programmatically click the link to trigger the download
      link.click();

      // Remove the link from the document body
      document.body.removeChild(link);

      // Revoke the URL to free up memory
      URL.revokeObjectURL(url);
    
    }
  }
};
</script>

<style scoped>
.page-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  /* 使用视窗高度来确保容器填满整个高度 */
}

.content {
  flex-grow: 1;
  /* 这将确保内容区域占据所有除了TabBar所需空间的空间 */
  padding-bottom: 70px;
  /* 这将确保内容底部和TabBar之间有足够的间隙 */
}

.tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  z-index: 10;
  /* 任意高于内容区域z-index的值确保TabBar在最上层 */
  /* TabBar的高度，如果已知 */
  height: 50px;
  /* 底部TabBar背景颜色，如果需要 */
  background-color: #fff;

  /* 确保内容底部和TabBar之间有足够的间隙 */
  box-shadow: 0px -1px 5px 0px rgba(0, 0, 0, 0.2);
  /* 如果需要阴影效果 */
}

.be-input {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  padding: 10px;
}

</style>