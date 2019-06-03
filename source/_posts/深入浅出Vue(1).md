---
title: 深入浅出Vue(1)
date: 2019-05-19 21:30:10
tags: VUE
---

## 深入响应式原理

### Vue.js响应式原理
---
当你把一个普通的 JavaScript 对象传入 Vue 实例作为 data 选项，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。这些 getter/setter 对用户来说是不可见的，但是在内部它们让 Vue 能够追踪依赖，在属性被访问和修改时通知变更。

---

以上是Vue中为了文档给出的解释，本文将会一个非常简单的例子出发，一步一步分析响应式原理的具体实现思路。

### 数据对象“可观测”
首先，我们定义一个数据对象，就以橘猫为例子：
``` js
const cat = {
  weight: 3,
  color: 'orange'
}
```
我们定义了一只3kg重的橘猫。
现在我们可以通过cat.weight和cat.color直接读写对应的属性值。但是当猫的属性被读取或修改时，我们不能第一时间获知。那么应该如何做才能够让猫主动告诉我们，它的属性被修改了呢？这时候就需要Object.defineProperty的力量了。

MDN上关于Object.defineProperty的介绍：

> Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。

在本文中，我们只使用这个方法使对象变得“可观测”，对defineProperty有兴趣的话可以参考[MDN Web 文档](https://developer.mozilla.org)。

改写一下上面的例子,让猫主动通知我们其属性的读写情况：

``` js
let cat = {}
let val = 3
Object.defineProperty(cat, 'weight', {
  get () {
    console.log('我的weight属性被读取了！')
    return val
  },
  set (newVal) {
    console.log('我的weight属性被修改了！')
    val = newVal
  }
})
```
我们通过Object.defineProperty方法，给cat定义了一个weight属性，这个属性在被读写的时都会打出日志。

### 计算属性
现在，橘猫已经变得可观测，任何的读写操作都会主动告诉我们，但也仅此而已，我们仍然不知道它的名字。如果我们希望在修改猫的weight之后，它能够主动告诉其他信息，这应该怎样才能办到呢？假设可以这样：

``` js
watcher(cat, 'nickname', () => {
  return car.weight > 5 ? '橘猪' : '橘猫'
})
```

我们定义了一个watcher作为“监听器”，它监听了cat的nickname属性。这个属性的值取决于cat.weight，即当cat.weight发生变化时，cat.nickname也应该发生变化，前者是后者的依赖。我们可以把cat.nickname称为“计算属性”。

那么，我们应该怎样才能正确构造这个监听器呢？可以看到，在设想当中，监听器接收三个参数，分别是被监听的对象、被监听的属性以及回调函数，回调函数返回一个该被监听属性的值。

``` js
/**
 * 当计算属性的值被更新时调用
 * @param { Any } val 计算属性的值
 */
function onComputedUpdate (val) {
  console.log(`我的昵称是：${val}`);
}
/**
 * 观测者
 * @param { Object } obj 被观测对象
 * @param { String } key 被观测对象的key
 * @param { Function } cb 回调函数，返回“计算属性”的值
 */
function watcher (obj, key, cb) {
  Object.defineProperty(obj, key, {
    get () {
      const val = cb()
      onComputedUpdate(val)
      return val
    },
    set () {
      console.error('计算属性无法被赋值！')
    }
  })
}
```
### 依赖收集

我们知道，当一个可观测对象的属性被读写时，会触发它的getter/setter方法。换个思路，如果我们可以在可观测对象的getter/setter里面，去执行监听器里面的onComputedUpdate()方法，是不是就能够实现让对象主动发出通知的功能呢？

由于监听器内的onComputedUpdate()方法需要接收回调函数的值作为参数，而可观测对象内并没有这个回调函数，所以我们需要借助一个第三方来帮助我们把监听器和可观测对象连接起来。

这个第三方就做一件事情——收集监听器内的回调函数的值以及onComputedUpdate()方法。

现在我们把这个第三方命名为“依赖收集器”：

``` js
const Dep = {
  target: null
}
```

依赖收集器的target就是用来存放监听器里面的onComputedUpdate()方法的。
定义完依赖收集器，我们回到监听器里，看看应该在什么地方把onComputedUpdate()方法赋值给Dep.target：

``` js
function watcher (obj, key, cb) {
  // 定义一个被动触发函数，当这个“被观测对象”的依赖更新时调用
  const onDepUpdated = () => {
    const val = cb()
    onComputedUpdate(val)
  }

  Object.defineProperty(obj, key, {
    get () {
      Dep.target = onDepUpdated
      // 执行cb()的过程中会用到Dep.target，
      // 当cb()执行完了就重置Dep.target为null
      const val = cb()
      Dep.target = null
      return val
    },
    set () {
      console.error('计算属性无法被赋值！')
    }
  })
}
```

我们在监听器内部定义了一个新的onDepUpdated()方法，这个方法很简单，就是把监听器回调函数的值以及onComputedUpdate()给打包到一块，然后赋值给Dep.target。这一步非常关键，通过这样的操作，依赖收集器就获得了监听器的回调值以及onComputedUpdate()方法。作为全局变量，Dep.target理所当然的能够被可观测对象的getter/setter所使用。

重新看一下我们的watcher实例：

``` js
watcher(cat, 'nickname', () => {
  return car.weight > 5 ? '橘猪' : '橘猫'
})
```

在它的回调函数中，调用了猫的health属性，也就触发了对应的getter函数。接下来我们需要回到定义可观测对象的defineReactive()方法当中，对它进行改写：

``` js
function defineReactive (obj, key, val) {
  const deps = []
  Object.defineProperty(obj, key, {
    get () {
      if (Dep.target && deps.indexOf(Dep.target) === -1) {
        deps.push(Dep.target)
      }
      return val
    },
    set (newVal) {
      val = newVal
      deps.forEach((dep) => {
        dep()
      })
    }
  })
}
```

可以看到，在这个方法里面我们定义了一个空数组deps，当getter被触发的时候，就会往里面添加一个Dep.target。回到关键知识点Dep.target等于监听器的onComputedUpdate()方法，这个时候可观测对象已经和监听器捆绑到一块。任何时候当可观测对象的setter被触发时，就会调用数组中所保存的Dep.target方法，也就是自动触发监听器内部的onComputedUpdate()方法。

至于为什么这里的deps是一个数组而不是一个变量，是因为可能同一个属性会被多个计算属性所依赖，也就是存在多个Dep.target。定义deps为数组，若当前属性的setter被触发，就可以批量调用多个计算属性的onComputedUpdate()方法了。

完成了这些步骤，基本上VUE响应式系统就已经模拟完成

### 总结
Vue 的响应式，核心机制是 观察者模式。

数据是被观察的一方，发生改变时，通知所有的观察者，这样观察者可以做出响应，比如，重新渲染然后更新视图。

响应式的核心逻辑，都在 Vue 项目的[“vue/src/core/observer”](https://github.com/vuejs/vue/tree/v2.6.10/src/core/observer)目录下面,有兴趣的可以去看看。
