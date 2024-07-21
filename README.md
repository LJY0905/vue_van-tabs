# Vue 3 + Vite

This template should help get you started developing with Vue 3 in Vite. The template uses Vue 3 `<script setup>` SFCs, check out the [script setup docs](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) to learn more.

Learn more about IDE Support for Vue in the [Vue Docs Scaling up Guide](https://vuejs.org/guide/scaling-up/tooling.html#ide-support).

# Notes:

## vue.js 实现 van-tabs 功能(样式针对移动端)

### 1.实现 tab 栏的左右滑动

```
display: flex;
flex-wrap: nowrap;
overflow-x: auto;
```

并隐藏滚动条：`scrollbar-width: none`;

### 2.实现吸顶

`position: sticky; top: 0`;

### 3.点击 tab 高亮

定义`activeIndex`，监听点击事件，更新`activeIndex`，绑定属性：`:class="{ active: activeIndex === index }"`，定义`.active`下`font-weight:bold`；
同时设置底部高亮线：

```
.active::after {
      content: '';
      position: absolute;
      width: 60%;
      bottom: 0;
      left: 20%;
      height: 2px;
      background-color: #ff397e;
      animation: high-light 0.3s ease-out;
    }
```

### 4.实现 tab 栏左右滑动

根据选择的 `tab` 的位置，判断是否需要滑动并滑动相应的距离。
获取点击位置距窗口左边的距离以及点击 `tab` 右边距离窗口右边的距离，获取 `tab` 小盒子的宽度，获取整个窗口的宽度，根据 `tab` 左右的距离，判断是否需要滑动，并计算滑动距离：已经滑动的距离+左边距-页面一半宽+`tab` 盒子宽

### 5.点击 `tab` 与标签页联动，根据点击的 `tab` 将对应的标签页滑动至页面顶部：

获取在整个页面中 `tab` 栏距离顶部的距离已经对应的标签元素距离顶部的距离，当点击 `tab` 时，默认 `tab` 栏滚动到页面顶部，滚动距离为 tab 栏到顶部距离，然后再滚动标签页到顶部距离-`tab` 栏高度。注意页面滚动时由于元素高度问题，`scrollTop` 可能不生效，可以用

```window.scrollTo({
left: x,
top: y,
behavior: 'smooth'
})
```

### 6.页面滚动时，动态高亮对应的 tab

使用 `window.onscroll` 事件监听滚动事件，判断个个元素距离顶部的距离来判断高亮 `tab`，可行但是动效卡顿，交互不友好。
改用 `intersectionObserver`。首先为各个标签元素设置自定义属性，方便确定 `index` 值，高亮对应的 `tab`。初始化 `map`，`key` 是各个标签元素，`value` 记录对应的属性，包括是否出现在页面，高度，对应的 `key` 值等。获取标签元素伪数组，并转化成数组，对元素依次 `observe`。定义 `intersectionObserver` 实例，对于每一个 `entries`，更新 `isIntersecting` 属性，确定在页面中出现，然后遍历所有的元素，更新其距离顶部的高度。对于每一个出现在页面中的元素，选择一个 `top` 属性（距离顶部距离）最小的，记录其 `key` 值，并赋值给 `activeIndex`，对于该实例，设置 `option：rootMargin` 偏移量，设置为 `tab` 栏的高度。同时实现 `tab` 栏的左右滑动。注意由于懒加载，这一部分最好在 `onMounted` 生命周期中进行，不然可能会拿不到对应的 dom 元素。

### 优化：

设置 tab 栏的标签动画以及`sticky`吸顶的抖动改善`z-index：999`

### 改善 tab 栏滚动时的抖动问题

`sticky` 改为 `fixed`；
由于 `fixed` 之后 `tab` 栏无法左右滑动的，解决此问题，在 `tabsHeader` 外再套一个 `div`，给定一个宽度，并将 `fixed` 样式加在该元素上
