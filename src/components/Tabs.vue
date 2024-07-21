<template>
  <div class="tabsContainer">
    <div class="headerContainer" :class="{ fixed: y > barTop }">
      <div class="tabsHeader" ref="tabsHeaderRef">
        <div
          v-for="(tab, index) in tabList"
          :key="tab.title"
          class="tab"
          :class="{ active: activeIndex === index }"
          @click="handleClickTab(index, $event)"
        >
          <span>{{ tab.title }}</span>
        </div>
      </div>
    </div>

    <div class="tabsContent">
      <slot name="content"></slot>
    </div>
  </div>
</template>
<script setup>
import { ref, defineProps, onMounted } from "vue";

import { useScroll } from "@vueuse/core";

const tabsHeaderRef = ref();
const activeIndex = ref(0);
const props = defineProps({
  tabList: {
    type: Array,
    default: () => [],
  },
});

const { y } = useScroll(window); //滚动距离

const handleClickTab = (val, event) => {
  if (activeIndex.value != val) {
    activeIndex.value = val;
    //tab栏左右划
    let tabLeft = event.clientX;
    let tabBox = document.querySelector(".tab").clientWidth / 2;
    let totalWidth = document.body.clientWidth;
    let width = totalWidth / 2;
    let tabRight = totalWidth - tabLeft;
    let scrollBox = document.querySelector(".tabsHeader");
    // console.log(scrollBox.scrollLeft)
    let scrollL = scrollBox.scrollLeft;
    if (tabLeft > 300 || tabRight > 300) {
      scrollBox.style.scrollBehavior = "smooth";
      scrollBox.scrollLeft = scrollL + (tabLeft - width) + tabBox;
    }

    // 与标签内容联动  根据元素判断页面滚动距离
    let tabbar = document.querySelector(".tabsContainer").offsetTop;
    let tabbarHeight = tabsHeaderRef.value.clientHeight;
    console.log(tabbar);
    window.scrollTo({
      left: 0,
      top: tabbar,
      behavior: "smooth",
    });
    let div = document.querySelector(".tabsContent");
    let targetChild = div.children[val];
    let tabTop = div.getBoundingClientRect().top;
    let targetTop = targetChild.getBoundingClientRect().top;
    console.log(tabTop, targetTop);
    if (targetTop - tabTop > 20 || tabTop - targetTop < 20) {
      window.scrollTo({
        left: 0,
        top: tabbar + targetTop - tabTop - tabbarHeight,
        behavior: "smooth",
      });
    }
  }
};

// 监听windows的滚动事件，但是效果不佳
// window.onscroll = function () {
//   let div = document.querySelector('.tabsContent').children
//   console.log(div)
//   const tabbar = document.querySelector('.tabsContainer').offsetTop
//   const tab = document.querySelector('.tabsHeader')
//   // console.log(tabbar, window.scrollY)
//   let height = 0
//   for (let i = 0; i < div.length; i++) {
//     if (window.scrollY - tabbar >= height + tab.clientHeight - 30) {
//       activeIndex.value = i
//     }
//     height += div[i].clientHeight
//   }
//   let tabLeft = tab.children[activeIndex.value].getBoundingClientRect().left
//   // console.log(tabLeft, activeIndex.value)
//   let tabBox = document.querySelector('.tab').clientWidth / 2
//   let totalWidth = document.body.clientWidth
//   let width = totalWidth / 2
//   let tabRight = totalWidth - tabLeft
//   let scrollBox = document.querySelector('.tabsHeader')
//   // console.log(scrollBox.scrollLeft)
//   let scrollL = scrollBox.scrollLeft
//   if (tabLeft > 300 || tabRight > 300) {
//     scrollBox.style.scrollBehavior = 'smooth'
//     scrollBox.scrollLeft = scrollL + (tabLeft - width) + tabBox
//   }
// }
const domMap = new Map();

// 设置map属性
const setDomMap = (dom, obj) => {
  const element = domMap.get(dom);
  const value = {
    key: element?.key,
    top: element?.top,
    height: element?.height,
    index: element?.index,
    isIntersecting: element?.isIntersecting,
    ...obj,
  };
  domMap.set(dom, value);
};

const barTop = ref(0);

onMounted(() => {
  barTop.value = document.querySelector(".tabsHeader").offsetTop;
  const options = {
    rootMargin: `-${
      document.querySelector(".tabsHeader").clientHeight
    }px 0px 0px 0px`, // 计算交叉值时添加至根的边界盒中的一组偏移量，marginTop 是头部区域+选项卡的高度
    // threshold: 0 // 规定了一个监听目标与边界盒交叉区域的比例值
  };

  const intersectionObserver = new IntersectionObserver((entries) => {
    // console.log(entries);
    const tab = document.querySelector(".tabsHeader");
    entries.forEach((entry) => {
      // 更新 isIntersecting 属性，是否相交
      setDomMap(entry.target, { isIntersecting: entry.isIntersecting });
    });

    // 遍历所有属性，更新距离顶部高度
    Array.from(domMap.keys()).forEach((dom) => {
      const rect = dom.getBoundingClientRect();
      setDomMap(dom, { top: rect.top, height: rect.height });
    });

    let min = 1000;
    let key = null;
    // console.log(domMap);
    // 遍历domMap，根据每个dom元素存储的top值，找到距离父元素最近的一个dom元素
    for (const [, value] of domMap) {
      if (value.isIntersecting) {
        if (value.top < min) {
          min = value.top;
          key = value.key;
        }
      }
    }
    if (key) {
      // console.log(key);
      activeIndex.value = Number(key);
      let tabLeft =
        tab.children[activeIndex.value].getBoundingClientRect().left;
      let tabBox = document.querySelector(".tab").clientWidth / 2;
      let totalWidth = document.body.clientWidth;
      let width = totalWidth / 2;
      let tabRight = totalWidth - tabLeft;
      let scrollBox = document.querySelector(".tabsHeader");
      let scrollL = scrollBox.scrollLeft;
      if (tabLeft > 300 || tabRight > 300) {
        scrollBox.style.scrollBehavior = "smooth";
        scrollBox.scrollLeft = scrollL + (tabLeft - width) + tabBox;
      }
    }
  }, options);

  const headerEle = document.querySelector(".tabsHeader");
  const contentEle = document.querySelector(".tabsContent");
  const eleLen = contentEle.children.length;
  for (let i = 0; i < eleLen; i++) {
    headerEle.children[i].setAttribute("data-nav-id", `nav-${i}`);
    contentEle.children[i].setAttribute("data-content-id", `content-${i}`);
  }
  const observerNodes = Array.from(contentEle.children);

  observerNodes.forEach((el, index) => {
    intersectionObserver.observe(el);
    const attr = el.getAttribute(`data-content-id`);
    // console.log(attr);
    const key = attr?.split("-")?.[1];
    setDomMap(el, {
      isIntersecting: false,
      key,
      index,
      top: -1,
      height: -1,
    });
  });
});
</script>
<style lang="scss" scoped>
.tabsContainer {
  // 解决fixed之后无法左右滑动的问题，在tabsHeader外再套一个div，给定一个宽度，并将fixed样式加在该元素上
  .headerContainer {
    width: 100%;

    &.fixed {
      position: fixed;
      top: 0;
      z-index: 999;
    }
  }
  .tabsHeader {
    // 吸顶
    // position: -webkit-sticky;
    // position: sticky;
    // z-index: 999;
    // top: 0;
    width: 100%;
    display: flex;
    flex-wrap: nowrap;
    font-size: 16px;
    overflow-x: auto;
    overflow-y: visible;
    scrollbar-width: none;
    background-color: #fff;

    .tab {
      line-height: 40px;
      margin: 0 10px;
      padding-bottom: 2px;
      position: relative;
      font-size: 14px;
      span {
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow-y: overlay;
        animation: font-fade 0.3s ease-out;
      }
    }
    .tab.active {
      span {
        font-weight: bold;
        animation: font-change 0.3s ease-out;
      }
    }

    .active::after {
      content: "";
      position: absolute;
      width: 60%;
      bottom: 0;
      left: 20%;
      height: 2px;
      background-color: #ff397e;
      animation: high-light 0.3s ease-out;
    }
  }
}

@keyframes font-change {
  from {
    font-weight: normal;
  }
  to {
    font-weight: bold;
  }
}
@keyframes font-fade {
  from {
    font-weight: bold;
  }
  to {
    font-weight: normal;
  }
}

@keyframes high-light {
  from {
    width: 0;
    background-color: #fff;
  }
  to {
    width: 60%;
    background-color: #ff397e;
  }
}
</style>
