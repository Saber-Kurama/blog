
[# VS Code + ESLint 自动格式化和快捷键配置](https://www.iszy.cc/posts/12/)

[# 纯干货，插件实现按照 ESLint 规则自动格式化](https://juejin.cn/post/7052289253761368077)



## 配置自动格式化

```json
{  
  "editor.codeActionsOnSave": {  
    "source.fixAll.eslint": true  
  },  
  "eslint.validate": ["javascript", "vue", "html"]  
}
```