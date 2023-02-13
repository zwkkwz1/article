vue源码学习建议《vue.js设计与实现》、配合mini-vue和vue源码一起看



先看《vue.js设计与实现》，然后去源码上验证。看源码的过程中可以先在mini-vue上看哪些是重点代码（不敢保证所有重点代码都有），并且看不懂的地方可以去mini-vue找对应的代码。看上面的注释。

[mini-vue链接](https://wiki.enbrands.com/github.com/cuixiaorui/mini-vue)

[《Vue.js设计与实现》](https://m.tb.cn/h.fngUiCf)

（V大写 很重要，像我之前那些小写。出去面试看到这可能就被刷掉了）


注：《vue.js设计与实现》书里讲的是大体思路，代码结果可能和书里表现得会有些微差别，要看细节还是要用源码打着断电跑一边。


大佬们会说要想真的学懂某个源码，要自己实现一遍。


举个例子

看源码时

```
（V大写 很重要，像我之前那些小写。出去面试看到这可能就被刷掉了）



注：《vue.js设计与实现》书里讲的是大体思路，代码结果可能和书里表现得会有些微差别，要看细节还是要用源码打着断电跑一边。



大佬们会说要想真的学懂某个源码，要自己实现一遍。



举个例子

看源码时
```

一开始看的头晕。然后看mini-vue寻找解释

```
	// 返回的是一个对象的话
    // 先存到 setupState 上
    // 先使用 @vue/reactivity 里面的 proxyRefs
    // 后面我们自己构建
    // proxyRefs 的作用就是把 setupResult 对象做一层代理
    // 方便用户直接访问 ref 类型的值
    // 比如 setupResult 里面有个 count 是个 ref 类型的对象，用户使用的时候就可以直接使用 count 了，而不需要在 count.value
    // 这里也就是官网里面说到的自动结构 Ref 类型
    instance.setupState = proxyRefs(setupResult);
```

看到这段注释大概就明白了。具体的可以看 mountComponent 里面的介绍