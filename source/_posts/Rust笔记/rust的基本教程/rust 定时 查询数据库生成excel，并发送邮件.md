

## rust的定时任务

采用了 rust的 `tokio` 和 `cron` `tokio-cron-scheduler`

``` toml
[dependencies] 
cron= "0.12.0"
tokio = { version = "1", features = ["full"] }
tokio-cron-scheduler="0.10.0"
```


[git地址](https://github.com/mvniekerk/tokio-cron-scheduler/tree/main)

## rust 连接数据库

[# rust简单操作mysql增删查改](https://juejin.cn/post/7129750847512117256)

mysql 的密码特殊字符转码



## rust 生成Excel 文件

`calamine` 和 `xlsxwriter`


## rust 发邮件

[# 用Rust发一封图文并茂的邮件](https://youerning.top/post/rust/rust-email-tutorial/)

[# rust：使用lettre库实现邮件发送](https://juejin.cn/post/7271659132380299283)


## rust 的时间处理

`chrono`