---
title: 工作内容-选项卡loading
abbrlink: 337da9c4
date: 2024-07-08 09:05:01
categories:
  - 总结
tags:
  - 工作内容
---

# 场景


`选项卡`切换的常见是很常见的，就比如说：不同商品种类，切换显示该对应的商品信息；但是如果在不做页面缓存的情况下，是每次切换或者点击的时候都会去请求接口的，但是对于这个切换的时候无非就是拿到对应的`id`请求；


但是有没有一个比较好的方案呢，一个功能页面，展示的状态跟别为：请求时的`laoding`状态，请求成功渲染页面的结果页(`success`)，因为其他原因导致请求失败的失败效果(`error`)，还要有一个`重新请求或者刷新的功能`，但这个功能对于`h5`页面来讲是非常不错的一个优化方案；

对于这样的功能，常规的写法就是，写一个方法用于请求数据，并且接收一个`id`的参数，拿到参数去请求，每次切换的时候重新调用该方法；页面中渲染的话，就三种情况，失败的情况渲染一个按钮，用于重新请求或者刷新；

虽然这样的方案是可行的，但是有一个微小的问题，就是一个项目中有重复类似的功能，岂不是都要重新写一遍；

对于企业级项目来讲， 需要封装统一一下，不然类似页面类似功能代码太多太多了；在常规方案的函数上进行优化即可；

## Handler

解析过程参考下图：
![解析图](https://wangxiaoze-view.github.io/picx-images-hosting/images/tabs_loading.png)

这里的案例针对于`vue`来讲的，如图，根据切换的方法，我们可以对其进行优化；优化的方案其中关键的是`对于url或者id依赖`的监听；这里可使用`computed`,为什么不同`watch`呢，`computed`是依赖缓存值的变换进行更新的，消耗小，解析`dom`元素加载后立马执行的，内部缓存机制复用，效率高，调试方便； 而`watch`在首次加载的时候默认是不进行更新，用于观察`vue`的数据变动，监听值是必须存在的；

在触发某一事件后，先 `computed` 再 `methods` 再到 `watch，computed` 计算属性是基于它们的依赖进行缓存的

对于`url`进行`computed`属性，发生变化自动执行请求函数，这样就不用每次都调用方法了；

对于`computed`如何监听是不是就是更新了的，配合`watchEffect`搭配使用，`watchEffect`是一个监听器，会自动监听数据类型的属性，不需要具体到某个属性，一旦运行就会立即监听，组件卸载的时候会停止监听。如下：

```js
import {computed, watchEffect} from 'vue'

cont id = ref(1)

const changeId = () => id.value++;

const url = computed(() => 'https://api-xxxxx.com' + id.value);

watchEffect(() => {
    console.log('watchEffect to url', url);
})

```

每次执行`changeId`， `id`的依赖就会进行更新，而`watchEffect`可以监听到`id` 和`url`的最新值；不过`watchEffect`是不能获取上一次的值的，`watch`是可以的；

接着于`url`参数进行兼容判断，如果是一个`ref`类型的值，进行监听`watchEffect`，不是的情况执行自身的请求函数；如下：

```js
export function useFetch(url) {
	function doFetch() {
		// 请求api接口...
	}

	// isRef: 是否是ref的值；
	if (isRef(url)) {
		watchEffect(doFetch);
	} else {
		doFetch();
	}
}
```

对于成功，失败，加载状态也很简单，如下：

```js
export function useFetch(url) {
	// 数据
	const data = ref(null);
	// 失败
	const error = ref(null);

	async function doFetch() {
		data.value = null;
		error.value = null;

		// 坐下兼容，可能是ref的值，也可能是普通值
		// unref： 如果参数是 ref，则返回内部值，否则返回参数本身
		let urlVal = unRef(url);
		try {
			const res = await fetch(urlVal);
			data.value = res.json();
		} catch (err) {
			error.value = err;
		}
		// 请求api接口...
	}

	// isRef: 是否是ref的值；
	if (isRef(url)) {
		watchEffect(doFetch);
	} else {
		doFetch();
	}

	return { data, error, doFetch };
}
```

而对于页面的渲染进程直接判断，不过这里建议封装一个公共组件；

```html
<div v-if="error">错误：xxxx</div>

<div v-else-if="data">数据： xxxx</div>

<div v-else>loading...</div>
```
