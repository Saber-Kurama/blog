
使用CSS网格时，首先要设置`display: grid` 然后，我们使用以下组合显式定义网格：[grid-template-columns](https://css-tricks.com/almanac/properties/g/grid-template-columns/)，[grid-template-rows](https://css-tricks.com/almanac/properties/g/grid-template-rows/) 和 [grid-template-areas](https://css-tricks.com/almanac/properties/g/grid-template-areas/)。然后，下一步是在网格中放置项目。

这是应该使用的经典方法，我也推荐它。**没有任何明确定义**。我们称之为**隐式网格**。

## 显示，隐式？这到底是怎么回事？

奇怪的术语？[马图佐维奇已经](https://css-tricks.com/difference-explicit-implicit-grids/) [具有](https://css-tricks.com/difference-explicit-implicit-grids/) [很好的解释](https://css-tricks.com/difference-explicit-implicit-grids/) 在CSS网格中我们可以通过“隐式”和“显式”来定义什么，但是让我们直接深入了解[的](https://www.w3.org/TR/css-grid-1/#implicit-grids) [的](https://www.w3.org/TR/css-grid-1/#implicit-grids)[规格](https://www.w3.org/TR/css-grid-1/#implicit-grids) 说道：

[`grid-template-rows`](https://www.w3.org/TR/css-grid-1/#propdef-grid-template-rows)，[`grid-template-columns`](https://www.w3.org/TR/css-grid-1/#propdef-grid-template-columns)的[`grid-template-areas`](https://www.w3.org/TR/css-grid-1/#propdef-grid-template-areas) 属性定义了固定数量的轨道**显式网格**。 当网格项位于这些边界之外时，网格容器通过向网格添加隐式网格线来生成隐式网格轨迹。这些线与显式网格形式**隐式网格**。