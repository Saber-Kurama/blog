
## 背景

希望不同产品直接可以共享，第三方库代码，减少不必要的请求


可以想到的方案

### 方案1

CDN 方案

优点：

1. 最简单
   
缺点：

1. 必须有CDN


### 方案2

使用基座的中的资源方案或者使用一个公共服务的资源方案

优点：
1. 相对简单 （CDN的变体）
缺点：
2. 耦合度相对高，必须有基座或者公共服务

### 方案3

使用 `cacheStore` 来共享

优点：
1. 独立性
缺点：
1. 部署方案是在一个域下
2.  `cache` 有大小限制
3.  `https` 限制


不同的浏览器对于 Cache Storage（缓存存储）的大小限制是不一样的，一般来说，它是由浏览器厂商所设定并实现的，而且常常会发生变化。以下是不同浏览器 cache store 大小限制的一些估计值：

1. Chrome：Chrome浏览器通常会将 Cache Storage 的限制设置为大约80%的硬盘可用空间。但是，对于单个 Cache Storage 来说，通常将其大小设置为约 6% ~ 10% 的硬盘可用空间。
    
2. Firefox：Firefox的缓存存储一般最大为 350MB 到 2.5GB 不等。默认情况下它会将总磁盘空间的 350MB 用于缓存。
    
3. Safari：Safari浏览器的缓存大小限制取决于系统的磁盘空间，通常会将磁盘空间的 5% ~ 20% 留作缓存容量。
    
4. Edge：Microsoft Edge浏览器对于 cache store 的大小限制是不明确的，但通常限制在几百MB至2GB之间。
    

需要注意的是，在实际的应用中，许多网络资源都被缓存起来，一旦浏览器的 cache store 容量达到限制，就会影响性能和用户体验，建议在定期清除缓存或调整缓存大小的同时，进行缓存策略的优化和设置


## 思考 todo


### 缓存原理

缓存依靠的 `cacheStore`  那么猜测域应该和 `localStore是一致的`

所以理论上讲 是可以 从 `CD` 拿到 `HM` 的缓存

为了实现共享缓存方案


`vite-plugin-pwa` 的原理

问题
1. 怎么配置资源清单
2. 怎么定制server-works
3. 如何在开发环境中使用 hashRouter
4. 缓存方案的定制
5. 如何更新缓存
6. 如何删除缓存
7. 缓存的配置和优化
8. 如何发布更新提示


##  官方文档

###  入门

``` ts
import { VitePWA } from 'vite-plugin-pwa' 
export default defineConfig({ 
	plugins: [ VitePWA({ injectRegister: 'auto' }) ] 
})

```

如果想在 `dev` 中使用的话，需要配置 `devOptions`

### 注册服务工作者


#### 内联注册

#### 脚本注册

#### 人工注册

#### 自动注册

如果您的应用程序代码库没有导入插件公开的任何虚拟模块，插件将回退到[脚本注册](https://vite-pwa-org.netlify.app/guide/register-service-worker.html#script-registration)，否则，导入的虚拟模块将为您注册Service Worker。

### Service Worker Precache

配置Service Worker的preache清单

由于`vite-plugin-pwa`plugin使用[workbox-build](https://developer.chrome.com/docs/workbox/modules/workbox-build/)节点库来构建Service Worker，它只会在清单预缓存中包含`css`、`js`和`html`资源（检查[GlobPartical](https://developer.chrome.com/docs/workbox/reference/workbox-build/#type-GlobPartial)中的`globPatterns`条目）。

`workbox-build`节点库是基于文件的，也就是说，它将遍历应用程序的构建输出文件夹。`Vite`将在`dist`文件夹中生成构建，因此，`workbox-build`将遍历`dist`文件夹，将其中找到的所有资源添加到Service Worker的预缓存清单中。

所以默认 默认清单是dist目录下的 `css`、`js`、和`html`

可以通过配置`globalPatterns` 来调整 preache的清单

``` ts
import { VitePWA } from 'vite-plugin-pwa' 
export default defineConfig({ 
	plugins: [ 
		VitePWA({ 
		registerType: 'autoUpdate', 
		workbox: { globPatterns: ['**/*.{js,css,html,ico,png,svg}'] } 
		}) 
	]
})
```

那么来了 `preach` 和 `cache`的区别
``