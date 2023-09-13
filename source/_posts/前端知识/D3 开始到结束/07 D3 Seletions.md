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

您可以使用 .classed 在选择的元素中添加和删除类属性。语法是

```js
s.classed(className, value);
```

### text

.text 方法允许您设置选择中每个元素的文本内容。它是通常用于选择 HTML 元素（例如 div、p 和 span）和 SVG 文本元素。语法是
```js
s.text(value);
```

## 多次更新

.style、.attr、.classed 和 .text 方法可以链接在一起，从而允许您可以在一条语句中设置多个样式、属性和类。例如：

```js
d3.selectAll('circle')
.attr('cx', 100)
.attr('cy', 100)
.attr('r', 50)
.style('fill', 'red')
.classed('item', true);
```

###  .select and .selectAll

style、.attr、.classed 和 .text 方法可用于使用.select 和.selectAll。上面所有的例子都使用了.selectAll。如果使用.select，则仅将选择第一个匹配的元素。尝试将 .selectAll 更改为 .select CodePen 示例，您应该看到只有页面上的第一个元素被修改。

## 链式选择

您可以通过链接调用来选择另一个选择中的元素.select 和.selectAll。这个概念最好通过一个例子来解释。假设你有 HTML：


```js
d3.select('div.first')

.selectAll('p');
```