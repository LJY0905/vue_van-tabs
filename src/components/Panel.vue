<template>
  <div class="panel">
    <div class="header" :class="headerClass">
      <div class="title">
        {{ title }}
      </div>
    </div>
    <slot></slot>
  </div>
</template>
<script setup>
const props = defineProps({
  title: {
    type: String,
    default: "",
  },
  headerClass: {
    type: String,
    default: "",
  },
});
</script>
<style lang="scss" scoped>
* {
  box-sizing: border-box;
}
.panel {
  border-radius: 14px;
  background: #ffffff;
  overflow: hidden;
  padding-bottom: 16px;
  .header {
    width: 100%;
    height: 44px;
    display: flex;
    align-items: center;
    padding: 16px 16px 0;
    margin-bottom: 10px;
    border-radius: 14px 14px 0px 0px;
    border: 1px solid #ffffff;
    background: linear-gradient(0.43deg, #ffffff 14.82%, #ffebf0 137.01%);
    .title {
      flex-shrink: 0;
      display: flex;
      align-items: center;
      line-height: 20px;
      color: #15161f;
      font-weight: 500;
      font-size: 16px;
    }
  }
}
</style>
