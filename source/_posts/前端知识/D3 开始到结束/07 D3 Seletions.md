D3 选择基本上是 HTML 或 SVG 元素的数组。他们让你修改您选择的 HTML 或 SVG 元素的样式和属性。例如你可以使用D3选择一些SVG圆圈并将其颜色更改为红色。也使用选择将数组连接到 HTML 或 SVG 元素时（请参阅下一章）。

##  Creating a selection

D3 有两个用于进行选择的函数：d3.select 和 d3.selectAll。d3.select 选择单个元素，而 d3.selectAll 选择多个元素。
###  d3.select

d3.select 的语法是：
```js
d3.select(selector);
```
其中选择器是包含 CSS 选择器字符串的字符串（例如“#chart”）。

3.select 选择页面上与选择器字符串匹配的第一个元素。它返回具有许多修改方法（例如 .attr 和 .style）的对象选定的 HTML/SVG 元素。 （方法是分配给属性的函数一个物体的。）
例如，如果页面上有一些 SVG 圆圈：

```html
<circle></circle> <circle></circle> <circle></circle>
```
以下语句进行仅包含第一个圆圈的选择：
```js
d3.select('circle');

```

### d3.selectAll


## 更新选择的元素

使用 d3.select 或 d3.selectAll 创建 HTML/SVG 选区后元素 您可以使用多种方法修改元素。其中四个最多常用的选择方法有.style、.attr、.classed 和.text。

### style

.style 方法为选择中的每个元素设置 CSS 样式属性。语法是：

```js
s.style(property, value);
```

### attr

.attr 在选择中的每个元素上设置属性。语法是：

```js
s.attr(name, value);
```

### .classed

