

[ChatGPT next web](https://flowus.cn/yifei/share/7a955395-c429-4228-a3a1-86c23a201ea7)

[部署](https://askopenai.feishu.cn/docx/XtrdduHwXoSCGIxeFLlcEPsdn8b)

[【实操】用 Docker 快速部署 Next-Web](https://askopenai.feishu.cn/docx/PZvvdLkwXobVqJxmeb8cVZ9LnIh)


https://github.com/nodejs/docker-node/issues/2009
## 前置环境

宝塔安装了docker

软件商店->点击安装->选择3.9.2 Stable版本->安装。

## docker部署

### 拥有线上镜像

拉取镜像
``` sh
docker pull hzt2023/airabbit:v1.2
```

启动容器

```sh 

docker run -itd --name hanzaitu-airabbit -p 81:80 -p 85:85 -v /config/:/config/ hzt2023/airabbit:v1.2
```


### 配置域名

宝塔添加网站

配置ssl 证书

配置反向代理

docker build --build-arg NODE_ENV=production -t strapiapp:latest -f Dockerfile.prod . san'er

