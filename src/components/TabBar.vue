<template>
  <div class="tab-bar">
    <div :class="['tab', { 'selected': selectedTab == index }]" v-for="(tab, index) in tabs" :key="index"
      @click="selectTab(index)">
      <p>
        {{ tab.title }}
      </p>
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
      this.selectedTab = index;
      this.$emit('tabSelected', index);
    }
  }
};
</script>

<style scoped>
.tab {
  cursor: pointer;
  /* padding: 5px; */
  /* border-radius: 5px;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); */
  /* margin: 5px; */
  text-align: center;
  flex-grow: 1;
  flex-basis: 0;
}

.selected {
  background-color: #fff;
  /* box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); */
  border-radius: 10%;
}
</style>
