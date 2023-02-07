
[# 谷歌翻译中国版停服！教你解决修复 Chrome 浏览器无法翻译网页的问题方法教程](https://www.iplaysoft.com/fix-chrome-translate.html)


[# 谷歌浏览器内置翻译器和谷歌翻译 API 可用服务器 IP 地址](https://hexingxing.cn/google-chrome-built-in-translator-and-google-translate-api-available-server-ip-address/)


## 图书
https://zh.singlelogin.me/
http://loginzlib2vrak5zzpcocc3ouizykn6k5qecgj2tzlnab5wcbqhembyd.onion/


### `<edit-text>` Props

|参数名|描述|类型|默认值|
|---|---|---|:---:|
|type|组件类型|`'number'\| "date" \| "text" \| "password" \| "select" \| "textarea"` | `text`|
|text **(v-model)**|文本编辑内容| `string \| number` | `''`|
|placeholder| 提示文案| `string` | `''`|
|editButtonType| 编辑操作类型| `icon \| text`| `icon`|
|component| 自定义编辑组件| `Object`| `-` |
|componentProps| 编辑组件的props| `Object` | `-`|
|editing **(v-model)**|是否是编辑状态| `boolean`| `false`|
|autoSave|是否自动保存| `boolean`| `false`|
|hideSaveButton|是否隐藏保存按钮| `boolean`| `false`|
|hideCancleButton|是否隐藏取消按钮| `boolean`| `false`|
|renderValue|自定义文字渲染| `(text: string \| number) => string | any`| `-`|

### `<edit-text>` Events
|事件名|描述|参数|
|---|:---:|---|
|editStart| 开始编辑|  |
|editChange| 编辑内容更改| text: `string | number` |
|editEnd| 编辑结束| text: `string | number` |
|save| 保存按钮事件|  text: `string | number`|
|cancle| 取消按钮事件| text: `string | number` |
### `<edit-text>` Slots

|插槽名|描述|参数|
|---|:---:|---|