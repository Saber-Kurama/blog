# 解决TypeError Right-hand side of 'instanceof' is not callable

https://blog.csdn.net/weixin_42752574/article/details/105377140

https://codesource.io/how-to-fix-right-hand-side-of-instanceof-is-not-callable-in-vuejs/

在开发环境中

因为
```
TypeError: Right-hand side of 'instanceof' is not callable

```

造成界面路由无法跳转
一度以为是 v-for 和 router-link的问题

最终还是 组件的 props 写的不对
