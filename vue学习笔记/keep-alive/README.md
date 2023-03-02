# vue3内置组件KeepAlive组件学习笔记

组件的渲染是个递归过程，比较消耗性能，Vue提供了内置组件KeepAlive，用于缓存它包囊的组件。



... 表示这个位置省略了一部分代码

```
const KeepAliveImpl = {
    name: `KeepAlive`,
    // Marker for special handling inside the renderer. We are not using a ===
    // check directly on KeepAlive in the renderer, because importing it directly
    // would prevent it from being tree-shaken.
    __isKeepAlive: true,
    props: {
        include: [String, RegExp, Array],
        exclude: [String, RegExp, Array],
        max: [String, Number]
    },
    setup(props, { slots }) {
		...
		const sharedContext = instance.ctx;
		const cache = new Map();
        const keys = new Set();
		...
		sharedContext.activate = (vnode, container, anchor, isSVG, optimized) => {
            const instance = vnode.component;
            move(vnode, container, anchor, 0 /* ENTER */, parentSuspense);
            // in case props have changed
            patch(instance.vnode, vnode, container, anchor, instance, parentSuspense, isSVG, vnode.slotScopeIds, optimized);
            queuePostRenderEffect(() => { // ⑦
                instance.isDeactivated = false;
                if (instance.a) {
                    invokeArrayFns(instance.a); // 调用 onActivated 钩子
                }
                const vnodeHook = vnode.props && vnode.props.onVnodeMounted;
                if (vnodeHook) {
                    invokeVNodeHook(vnodeHook, instance.parent, vnode);
                }
            }, parentSuspense);
            ...
        };
		...
		let pendingCacheKey = null; // ③
        const cacheSubtree = () => {
            // fix #1621, the pendingCacheKey could be 0
            if (pendingCacheKey != null) {
                cache.set(pendingCacheKey, getInnerChild(instance.subTree));
            }
        };
		...
        return () => {
            ...
			const children = slots.default();
        	const rawVNode = children[0]; // ①
			...
			const comp = vnode.type;
			const key = vnode.key == null ? comp : vnode.key; // ②
        	const cachedVNode = cache.get(key);
			...
			return isSuspense(rawVNode.type) ? rawVNode : vnode;
        };
    }
}
```


组件运行 render 函数生成vnode（这个vnode不是dom，vnode.el才是dom），后面说的vnode都是这个vnode
```
function handleSetupResult(instance, setupResult, isSSR) {
    if (isFunction(setupResult)) {
        // setup returned an inline render function
        if (instance.type.__ssrInlineRender) {
            // when the function's name is `ssrRender` (compiled by SFC inline mode),
            // set it as ssrRender instead.
            instance.ssrRender = setupResult;
        }
        else {
            instance.render = setupResult;
        }
    }
	...
}
```

setupResult是setup函数执行的返回值。可以看出 如果setupResult是函数。它将赋值给render。

即KeepAlive组件的render方法为其setup方法的返回值。即每次更新或者挂载组件。就是执行：
```
		return () => {
            ...
			const children = slots.default();
        	const rawVNode = children[0]; // ①
			...
			let vnode = getInnerChild(rawVNode);
			const comp = vnode.type;
			const key = vnode.key == null ? comp : vnode.key; // ②
        	const cachedVNode = cache.get(key);
			pendingCacheKey = key;
            if (cachedVNode) { // ③
                // copy over mounted state
                vnode.el = cachedVNode.el; // 获取缓存的dom
                vnode.component = cachedVNode.component; // 获取缓存的组件实例
                if (vnode.transition) {
                    // recursively update transition hooks on subTree
                    setTransitionHooks(vnode, vnode.transition);
                }
                // avoid vnode being mounted as fresh
                vnode.shapeFlag |= 512 /* ShapeFlags.COMPONENT_KEPT_ALIVE */;
                // make this key the freshest
                keys.delete(key);
                keys.add(key);
            }
			...
			return isSuspense(rawVNode.type) ? rawVNode : vnode;
        };
```
生成vnode。

KeepAlive组件本身并不渲染，它要渲染和缓存的是它的子节点即slots.default()[0]

#### 缓存设计
**方法：**看 **cacheSubtree** 方法，instance.subTree被缓存在cache中。组件每次挂载时会更新这个缓存。

instance.subTree即slots.default()[0] ：
```
function mountComponent() {
    const componentOptions = vnode.type // vnode的type保存组件选项。
    const { render， data } = componentOptions
    const state = reactive(data())
    const instance = {
        isMounted: false,
        subTree: null
    }
    //  因为更新时是对比vnode。所以要在vnode上保存instance
    vnode.comoponent = instance
     
    effect(() => {
        const subTree = render.call(state)
        if(!instance.isMounted) {
            patch(null, subTree)
            instance.isMounted = true
        } else {
            // 实现补丁更新。
            patch(instance.subTree, subTree)
        }
        instance.subTree = subTree
    })
}
```

##### 缓存使用：
看代码③，render函数的执行结果可以从catch缓存中获取

然后在patch过程中，KeepAlive自带的activate方法将被调用。

执行move(vnode, container, anchor, 0 /* ENTER */, parentSuspense);方法，将正确的dom（即vnode.el）插入正确的位置。

#### 细节
#### 由代码①可以看到，KeepAlive组件内只能包含单个插槽。

由代码②可知，当KeepAlive的插槽的组件不存在key属性时，用KeepAlive的插槽的组件实例的type对象作为key。
```
<keep-alive>
    <router-view></router-view>
</keep-alive>
```
**所以如果路由缓存写法如上。路由切换时，key指向的一直是RouterView组件。 cachedVNode 获取会出错。最终也不会执行KeepAlive相关的缓存功能。**
```
<router-view #default="{Component}">
    <keep-alive>
      <component :is="Component"/>
    </keep-alive>
  </router-view>
```
所以要写成这种形式

**onActivated的调用**
```
function onActivated(hook, target) {
    registerKeepAliveHook(hook, "a" /* LifecycleHooks.ACTIVATED */, target);
}
function onDeactivated(hook, target) {
    registerKeepAliveHook(hook, "da" /* LifecycleHooks.DEACTIVATED */, target);
}
function registerKeepAliveHook(hook, type, target = currentInstance) {
    // cache the deactivate branch check wrapper for injected hooks so the same
    // hook can be properly deduped by the scheduler. "__wdc" stands for "with
    // deactivation check".
    const wrappedHook = hook.__wdc ||
        (hook.__wdc = () => {
            // only fire the hook if the target instance is NOT in a deactivated branch.
            let current = target;
            while (current) {
                if (current.isDeactivated) {
                    return;
                }
                current = current.parent;
            }
            return hook();
        });
    injectHook(type, wrappedHook, target);
    // In addition to registering it on the target instance, we walk up the parent
    // chain and register it on all ancestor instances that are keep-alive roots.
    // This avoids the need to walk the entire component tree when invoking these
    // hooks, and more importantly, avoids the need to track child components in
    // arrays.
    if (target) { // ⑥
        let current = target.parent;
        while (current && current.parent) {
            if (isKeepAlive(current.parent.vnode)) {
                injectToKeepAliveRoot(wrappedHook, type, target, current);
            }
            current = current.parent;
        }
    }
}
function injectHook(type, hook, target = currentInstance, prepend = false) {
    if (target) {
        const hooks = target[type] || (target[type] = []);
        // cache the error handling wrapper for injected hooks so the same hook
        // can be properly deduped by the scheduler. "__weh" stands for "with error
        // handling".
        const wrappedHook = hook.__weh ||
            (hook.__weh = (...args) => {
                if (target.isUnmounted) {
                    return;
                }
                // disable tracking inside all lifecycle hooks
                // since they can potentially be called inside effects.
                pauseTracking();
                // Set currentInstance during hook invocation.
                // This assumes the hook does not synchronously trigger other hooks, which
                // can only be false when the user does something really funky.
                setCurrentInstance(target);
                const res = callWithAsyncErrorHandling(hook, target, type, args);
                unsetCurrentInstance();
                resetTracking();
                return res;
            });
        if (prepend) {
            hooks.unshift(wrappedHook);
        }
        else {
            hooks.push(wrappedHook);
        }
        return wrappedHook;
    }
    else if ((process.env.NODE_ENV !== 'production')) {
        const apiName = toHandlerKey(ErrorTypeStrings[type].replace(/ hook$/, ''));
        warn(`${apiName} is called when there is no active component instance to be ` +
            `associated with. ` +
            `Lifecycle injection APIs can only be used during execution of setup().` +
            (` If you are using async setup(), make sure to register lifecycle ` +
                    `hooks before the first await statement.`
                ));
    }
}
function injectToKeepAliveRoot(hook, type, target, keepAliveRoot) {
    // injectHook wraps the original for error handling, so make sure to remove
    // the wrapped version.
    const injected = injectHook(type, hook, keepAliveRoot, true /* prepend */);
    ...
}
```

onActivated的第一个参数（即onActivated内写的函数：hook）将保存在当前组件实例（currentInstance） currentInstance[type]内。onActivated的type是a。 即将函数保存在组件实例的a属性内。

看代码⑦，KeepAlive组件自带的activate方法运行时。 组件实例的a属性内的方法将被执行。即onActivated内的方法被执行。

看代码⑥，如果父组件是KeepAlive的，父组件的组件实例的a属性内也保存了这个hook,这个过程是while的所以只要祖先组件内有一个是KeepAlive的，这个hook就会存进这个祖先组件实力的a属性内。**所以KeepAlive内的组件，不论是第几层的子节点。它的onActivated钩子函数都会被执行**
```
function mountComponent() {
    const componentOptions = vnode.type // vnode的type保存组件选项。
    const { render， data } = componentOptions
    const state = reactive(data())
    const instance = {
        isMounted: false,
        subTree: null
    }
    //  因为更新时是对比vnode。所以要在vnode上保存instance
    vnode.comoponent = instance
     
    effect(() => {
        const subTree = render.call(state)
        if(!instance.isMounted) {
            patch(null, subTree)
            instance.isMounted = true
        } else {
            // 实现补丁更新。
            patch(instance.subTree, subTree)
        }
        instance.subTree = subTree
    })
}
```