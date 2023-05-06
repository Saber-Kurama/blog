

### 报错指示

vite does not provide an export named 'default'

### 报错原因

没有将 commonjs 模块转成 es module

### 处理办法

1.  在 vite.config.ts 配置 optimizeDeps.include。这个方法用来解决_引入正常的模块名_但仍出错的场景。
2.  如果模块提供了 es module 文件，如 has/src/index.esm.js，则可以通过别名，将 has/src/index.js 指向 has/src/index.esm.js，这个方案可以用来解决大部分 .umi 文件夹下使用绝对路径的问题，如 @umijs/runtime、dva-immer 等
3.  自己手动转换。如果上面两个方案仍没有解决，则需要实现 vite plugin 自己将代码进行转换。通过字符串替换，直接修改了返回给浏览器的文件，浏览器自然不会报错了。