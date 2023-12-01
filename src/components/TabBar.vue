<template>
  <div class="tab-bar">
    <div class="tab" v-for="(tab, index) in tabs" :key="index" @click="selectTab(index)">
      {{ tab.title }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tabs: [
        { title: '表格选择' },
        { title: '时间选择' },
        { title: '更新类别' },
        { title: '设置' }
      ],
      selectedTab: 0
    };
  },
  methods: {
    selectTab(index) {
      // console.log('Tab selected(tabbarvue):', index);
      this.selectedTab = index;
      this.$emit('tabSelected', index);
    }
  }
};
</script>

<style scoped>
.tab-bar {
  display: flex;
  justify-content: space-around;
  background-color: #f0f0f0;
  padding: 10px;
}

.tab {
  cursor: pointer;
  padding: 10px;
  border-radius: 5px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.tab:hover {
  background-color: #f0f0f0;
}
</style>
