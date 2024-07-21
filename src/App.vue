<template>
  <div class="container">
    <div class="content">页面内容</div>
    <Tabs :tabList="config">
      <template #content>
        <template v-for="item in config" :key="item.value">
          <Panel :title="item.title" class="panel" header-class="panelHeader">
            <div class="panelContent">内容内容</div>
          </Panel>
        </template>
      </template>
    </Tabs>
  </div>
</template>
<script setup>
import Panel from "./components/Panel.vue";
import Tabs from "./components/Tabs.vue";

const config = [
  { title: "标签1", value: "1" },
  { title: "标签2", value: "2" },
  { title: "标签3", value: "3" },
  { title: "标签4", value: "4" },
  { title: "标签5", value: "5" },
  { title: "标签6", value: "6" },
  { title: "标签7", value: "7" },
  { title: "标签8", value: "8" },
  { title: "标签9", value: "9" },
  { title: "标签10", value: "10" },
  { title: "标签11", value: "11" },
  { title: "标签12", value: "12" },
  { title: "标签13", value: "13" },
  { title: "标签14", value: "14" },
];
</script>
<style lang="scss" scoped>
.container {
  width: 100%;
  :deep(.tabsContainer) {
    .tabsHeader {
      margin-bottom: 8px;
      border-radius: 20.5px 20.5px 0 0;
      background: linear-gradient(to bottom, #ffffff, #f4f5fa);
    }
  }

  .content {
    height: 500px;
  }

  .panel {
    padding: 0 14px 14px 14px;
    margin: 0 12px 12px 12px;
    :deep(.header) {
      padding: 0;
      margin-bottom: 0;
      background: #fff;
      .title {
        font-size: 15px;
        font-weight: bold;
      }
    }

    .panelContent {
      font-size: 13px;
    }
  }
  .panel:nth-of-type(1) {
    margin-top: 8px;
  }
}
</style>
