本篇是 [Defensive CSS](https://defensivecss.dev) 网站提供的 tips 的备忘录。这里去除掉了一些 grid 相关的和比较常见的内容，并且有些内容做了一些合并。

## Flex 换行

在 Flex 布局中，如果里面的元素超过了容器宽度，默认配置是不会换行的。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8j33uxvj31cm0f40su.jpg)

可以通过添加 flex-wrap 解决：

```css
.options-list {
  display: flex;
  flex-wrap: wrap;
}
```

还有，在 Flex 中的文字是不会自动换行的。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8km9pa6j30z60g8jrn.jpg)

需要加上 min-width：

```css
.card__title {
  overflow-wrap: break-word;
  min-width: 0;
}
```

## 图片变形

object-fit 默认值是 fill。会强制拉伸图片到外部确定高度和宽度的容器中。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8ljxb0uj31cm0lw75q.jpg)

需要通过设置 object-fit 来消除：

```css
.card__thumb {
  object-fit: cover;
}
```

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8lou37lj31cm0lw0tj.jpg)

另外，可以作为**全局属性**，**但不要忘记添加 max-width**。

```css
img {
  max-width: 100%;
  object-fit: cover;
}
```

## 长文本

长文本默认会换行，有时候会影响我们的布局。通过：

```css
.username {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

如果文本超出容器宽度则会显示省略号。另外，建议通过 HTML 标签的 title 属性展示完整内容。

## 组件间距

组件之间需要留有空间，尽管有时候看着不需要，但需要有这个意识。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8opbn76j31cm0haweo.jpg)

```css
.section__title {
  margin-right: 1rem;
}
```

## 图片背景重复

众所周知，image 默认情况下是会开启 backround-repeat 的，这是一个坑。大部分情况下，我们并不希望图片背景重复：

```css
.hero {
  background-image: url("..");
  background-repeat: no-repeat;
}
```

## CSS 变量默认值

css 变量越来越风靡。在 css 变量没有找到的时候，加上缺省值。

```css
.message__bubble {
  max-width: calc(100% - var(--actions-width, 70px));
}
```

## 固定尺寸

很多时候，用固定的 width 和 height 不如使用 min-width 和 min-height。

## 组合选择器

在遇到一些浏览器兼容性相关的 css 属性，尽量不要使用组合选择器，因为[有些属性是整个弃用的](https://www.w3.org/TR/selectors/#grouping)。

🙅：

```css
/* Don't do this, please */
input::-webkit-input-placeholder,
input:-moz-placeholder {
  color: #222;
}
```

🙆：

```css
input::-webkit-input-placeholder {
  color: #222;
}

input:-moz-placeholder {
  color: #222;
}
```

## 滚动链

如果一个网页里嵌入一个 Modal，你会发现滚动到 Modal 底部之后，整个页面也会进行滚动。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8r6yvycj318g0ni3z0.jpg)

之前有很多 hack 方法来避免这个问题，现在可以通过 css 属性来实现：

```css
.modal__content {
  overscroll-behavior-y: contain;
  overflow-y: auto;
}
```

## 图片里的文字

当图片里存在文字描述时，如果图片没加载出来，文字就已经存在了，视觉效果不好。

最好为图片容器添加一个背景色：

```css
.card__img {
  background-color: grey;
}
```

## 响应式高度设计

篇幅较长，简单来说就是根据不同高度实现不同的 css。

具体可以看作者的[另外一篇 blog](https://ishadeed.com/article/responsive-design-height/)。

## 移动设备的 hover

在移动浏览器中浏览网页，滑动页面的时候有概率会触发元素 hover 效果，会影响用户体验。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8sszr2jj318g0mm74k.jpg)

解决方法是加入移动设备的 query media：

```css
@media (hover: hover) {
  .card:hover {
    /* Add hover styles.. */
  }
}
```

## 图片内边框

很多网站都设置有用户头像，如果没有边框的话，用户上传的图片背景可能会和网页背景色融为一体，对用户会造成困惑。可以加入边框和阴影解决：

```html
<div class="card__avatar">
  <img src="assets/shadeed.jpg" alt="" />
  <div class="border"></div>
</div>
```

```css
.card__avatar {
  position: relative;
}

.card__avatar img {
  width: 56px;
  height: 56px;
  border-radius: 50%;
}

.border {
  position: absolute;
  width: 56px;
  height: 56px;
  border: 2px solid #000;
  border-radius: 50%;
  opacity: 0.1;
}
```

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8t89lokj30zs0oc3yw.jpg)

## Flex 拉伸

flex 布局中，默认会把子项 y 轴拉长以适配 flex 容器，我们可以通过调整 align-items 来避免这个问题出现：

```css
.person {
  display: flex;
  align-items: center;
}
```

## iOS Safari 放大

默认情况下，在 iOS 端使用 Safari 浏览器输入框时会放大聚焦。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8tqqwahj318g0s4750.jpg)

解决方法很简单，就是加上：

```css
input {
  font-size: 16px;
}
```

## 按钮的最小宽度

在多语言环境下，同样意思的 Button 组件可能宽度会出现不一致的情况，这时候可以设置一个最小宽度保证画面的一致性。

![](https://tva1.sinaimg.cn/large/008vxvgGgy1h8g8hvh5n8j31gj0u0q3i.jpg)

```css
.button {
  min-width: 90px;
}
```
