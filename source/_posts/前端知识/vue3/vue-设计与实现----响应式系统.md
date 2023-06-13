
🤔：如何设计一个响应式系统

所谓响应式系统，其实就是将副作用方法和和其依赖的数据进行关联
也就是在副作用中收集依赖数据，当依赖数据发生修改的时候，副作用发生修改

🤔：如何收集依赖数据

副作用中访问依赖数据，那么我们就可以根据 `proxy`的getter 来收集依赖，并将副作用放到一个桶中, 当对数据赋值，触发`setter`就会调用 副作用函数

🤔：副作用中没有的key发生修改不应该触发副作用

定义一个全局 `activeEffect`变量来解决 `effect` 名称固定的问题 
为了解决上面问题，必须修改数据的存储方式
使用 `weakmap` 来存储响应式对象，好处是弱引用，对象销毁，`weakmap`也会销毁
使用 `map` 来存储对象的`key`, 副作用函数依然是`Set`

🤔： 分支和cleanup

某些情况下，副作用有 if 等分支条件语句，会造成副作用和之前的数据不再关联
解决方案，是每次重新执行副作用之前先删除之前的关联，重新建立连接

为了可以清除旧链接，可以采用 给副作用函数 添加`deps`依赖。这样就可以清除

🤔： 如何解决嵌套 `effect`呢

`effect`的嵌套解决方案其实就是创建 `effect`栈

🤔：`effect` 和 `proxy`进行关联之后，是否可以控制`effect`的执行时间或者次数？

为了解决这个问题，我们需要引入 `scheduler`调度器来解决这个问题

🤔： `computed` 和`lazy` 

通过传递属性`options`, `effect(fn, options)` 就可以控制是否立即执行`fn`, 返回副作用函数

```js
const effectFn =  effect(fn, {lazy: true})
/**
_effectFn 大致就是
const  _effectFn = () => {
	cleanup()
	activeEffect = effect
	fn()
	activeEffect
} 

*/
effectFn(); // 这个时候就开始创建连接
```

`computed` 返回一个响应式对象，只有当 获取 `value` 值的时候才会触发 `fn`
简单的实现
```js
const computed = (fn) => {
	const effectFn = effect(fn, {lazy: false})
	const cProxy = new Proxy( {
		get() {
			return effectF()
		}
		
	})
}
```

如何实现缓存呢

```js
const computed = (fn) => {
	const effectFn = effect(fn, {lazy: false})
	let val;
	let dirty = true
	const cProxy = new Proxy( {
		get() {
			if(dirty) {
				 val = effectFn()
				 dirty = false
				 return val
			}else {
				return val
			}
		}
		
	})
}
```

如何更新缓存, 使用`scheduler` 调度器

```js
const computed = (fn) => {
	
	let val;
	let dirty = true
	const effectFn = effect(fn, {lazy: false, scheduler() {
		dirty = true
	}})
	const cProxy = new Proxy( {
		get() {
			if(dirty) {
				 val = effectFn()
				 dirty = false
				 return val
			}else {
				return val
			}
		}
		
	})
}
```

当`computed`中的数据发生修改，如何触发 `computed`的副作用发生修改呢
因为`computed` 只有`getter` ,所以可以在`getter`中触发依赖收集`track(obj，'value')`
那么触发是在什么时候呢，那就是副作用发生修改`trigger`

```js
const computed = (fn) => {
	
	let val;
	let dirty = true
	const effectFn = effect(fn, {lazy: false, scheduler() {
		dirty = true
		trigger(cProxy, 'value')
	}})
	const cProxy = new Proxy( {
		get() {
			if(dirty) {
				 val = effectFn()
				 dirty = false
				 return val
			}else {
				return val
			}
			track(cProxy，'value')
		}
		
	})
}
```

🤔： `watch` 是怎么实现的呢？ `watch` 是基于`effect`

可以认为`watch`的一个参数是 `getter`函数或者一个响应式对象，第二个是 `scheduler`调度器。旧值和新值
```js

const watch = (w, fn, options) => {
	let getter;
	let oldValue, newValue;
	if(typeof w === 'function') {
		getter = w
	}else {
		// 递归响应对象
		getter = () => { tr(w)}
	}
	const  effectFn = effect(getter, {lazy: false,scheduler: () => {
		fn()
	})
	oldValue = effectFn()
})

```

`watch`立即执行和执行时机

🤔： 过期的副作用。如何解决数据竞争等问题呢 解决`watch`的副作用


