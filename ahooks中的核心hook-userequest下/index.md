# ahooks中的核心hook-useRequest（下）


开启掘金成长之旅！这是我参与「掘金日新计划 · 12 月更文挑战」的第 2 天，[点击查看活动详情](https://juejin.cn/post/7167294154827890702 'https://juejin.cn/post/7167294154827890702')

## 前言

之前大致说了下 useRequest hook 的基本功能的实现。但 useRequest 的强大不止于此。它还支持如 loading 状态延时、请求防抖、节流、依赖刷新等功能。不过其实现方式都是通过内置的插件 hook 来进行实现的。这样做既可以保证核心代码的简洁，还能更方便的扩展出更高级的功能。并且还支持用户进行自定义插件。

## 正文

### 如何定义一个插件

可以通过插件 hook 的 ts 定义，看一下怎么定义一个插件：

插件 hook 接收两个参数，

- fetchInstance: fetch 实例；
- fetchOptions: useRequest 接收的 options；

支持返回一些回调函数：

- onBefore： 在请求接口前调用；
- onRequest： 在请求接口时调用；
- onSuccess： 在请求接口成功后调用；
- onError： 在请求接口失败后调用；
- onFinally： 在请求接口完成后调用；
- onCancel： 在请求接口取消后调用；
- onMutate： 在出发 mutate 函数时调用；

插件 hook 可以返回一系列的生命周期函数，使我们可以在接口请求的任意一个时机对 fetchInstance 进行处理。

插件 hook 还支持挂载一个静态方法 onInit()，在 Fetch 创建之前，可以通过 Init 方法进行一些前提处理。得到初始的 state。

```js
const use***plugin = (
  fetchInstance,
  fetchOptions,
) => {
  const onBefore = () => {
    // 对fetchInstance的状态的处理...
  }
  ...
  return {
    onBefore,
    ...
  }
}
use***plugin.onInit = (fetchOptions) => {
  ...
}
export default use***plugin
```

### 插件的使用

插件 hook 一般用来处理 Fetch 实例的状态，在 Fetch 实例被创建之前，会先执行所有插件可能存在的 onInit 方法，该方法会返回 initData，用来初始化 Fetch 实例。

在 Fetch 实例被初始化后，执行所有的插件 hooks，此时，会得到一个包含（onBrfore、onSuccess...）的数组。并将这些方法挂载到 fetchInstance 上。方便在各个时机使用。

```js
const useRequestImplement = (
  service: Promise<any>,
  options,
  plugins: Array<Plugin>
) => {
  // const { manual = false, ...rest } = options;
  // const fetchOptions = {
  //   manual,
  //   ...rest,
  // };

  // const serviceRef = useLatest(service);
  // const update = useUpdate();

  const fetchInstance = useCreation(() => {
    // 在Filter方法中的Boolean可以当成一个转换函数，用来筛选掉空的initData。
    const initState = plugins
      .map((p) => p?.onInit?.(fetchOptions))
      .filter(Boolean);
    // 将初始化的initState传给构造函数Fetch。
    return new Fetch(
      serviceRef,
      fetchOptions,
      update,
      Object.assign({}, ...initState)
    );
  }, []);

  // fetchInstance.options = fetchOptions;

  // 执行所有插件hook，并将其挂载到fetchInstance上。
  fetchInstance.pluginImpls = plugins.map((p) =>
    p(fetchInstance, fetchOptions)
  );

  // useMount(() => {
  //   // useCachePlugin can set fetchInstance.state.params from cache when init
  //   const params = fetchInstance.state.params ?? [];
  //   // @ts-ignore
  //   fetchInstance.runAsync(...params);
  // });
  // useUnmount(() => {
  //   fetchInstance.cancel();
  // });
  // return {
  //   loading: fetchInstance.state.loading,
  //   data: fetchInstance.state.data,
  //   error: fetchInstance.state.error,
  //   params: fetchInstance.state.params || [],
  //   cancel: useMemoizedFn(fetchInstance.cancel.bind(fetchInstance)),
  // };
};
```

```js
export default class Fetch<TData, TParams extends any[]> {
//   public count: number = 0;
//
//   public state = {
//     loading: false,
//     params: undefined,
//     data: undefined,
//     error: undefined,
//   };

  public pluginImpls: Array<PluginReturn>
  constructor(
    public serviceRef,
    public options,
    public subscribe,
    public initState
  ) {
    this.state = {
      ...this.state,
      loading: !options.manual,
      ...initState, //在constructor时会讲插件初始化生成的状态更新到state中。
    };
  }
  // setState(s) {
  //   this.state = {
  //     ...this.state,
  //     ...s,
  //   };
  //   this.subscribe();
  // }

  // 插件处理函数，会批量执行某个生命周期的函数。并返回处理后的状态。
  runPluginHandler(event: keyof PluginReturn, ...rest: any[]) {
    // @ts-ignore
    const r = this.pluginImpls.map((i) => i[event]?.(...rest)).filter(Boolean);
    return Object.assign({}, ...r);
  }
//   async runAsync(...params: TParams): Promise<any> {
//     this.count += 1;
//     const currentCount = this.count;

    	// 执行所有插件hooks返回的‘onBefore’方法，并根据返回的state进行相应的处理
      const {
        stopNow = false,
        returnNow = false,
        ...state
      } = this.runPluginHandler('onBefore', params);
      if (stopNow) {
        return new Promise(() => {});
      }
//
//     this.setState({
//       loading: true,
//       params,
        ...state,
//     });
//
    // this.options.onBefore?.(params);
//     try {
//       const servicePromise = this.serviceRef.current(...params);
//       const res = await servicePromise;
//       if (currentCount !== this.count) {
//         return new Promise(() => {});
//       }
//       this.setState({
//         data: res,
//         error: undefined,
//         loading: false,
//       });
//
      // this.options.onSuccess?.(res, params);
      // this.options.onFinally?.(params, res, undefined);
        this.runPluginHandler('onSuccess', res, params);
  			if (currentCount === this.count) {
          this.runPluginHandler('onFinally', params, res, undefined);
        }
//
//       return res;
//     } catch (error) {
//       if (currentCount !== this.count) {
//         // prevent run.then when request is canceled
//         return new Promise(() => {});
//       }
//       this.setState({
//         error,
//         loading: false,
//       });
//
      // this.options.onError?.(error, params);
      // this.options.onFinally?.(params, undefined, error);
        this.runPluginHandler('onError', error, params);
				if (currentCount === this.count) {
          this.runPluginHandler('onFinally', params, undefined, error);
        }
//       throw error;
//     }
//   }

//   cancel() {
//     this.count += 1;
//     this.setState({
//       loading: false,
//     });
  		this.runPluginHandler('onCancel');
//   }
//
//   refreshAsync() {
//     // @ts-ignore
//     return this.runAsync(...(this.state.params || []));
//   }

  // run(...params: TParams) {
  //   this.runAsync(...params).catch((error) => {
  //     if (!this.options.onError) {
  //       console.error(error);
  //     }
  //   });
  // }
  // refresh() {
  //   // @ts-ignore
  //   this.run(...(this.state.params || []));
  // }

  mutate(data?: TData | ((oldData?: TData) => TData | undefined)) {
    // const targetData = isFunction(data) ? data(this.state.data) : data;
    this.runPluginHandler('onMutate', targetData);
    // this.setState({
    //   data: targetData,
    // });
  }
}
```

> 大概了解了插件 hook 的运行机制。其抛出一系列的回调函数，在接口请求的各个时机执行，对 fetchInstance 进行处理

### Loading Delay

useRequest 有个 loading delay 的功能，通过设置 options.loadingDelay ，可以延迟 loading 变成 true 的时间，有效防止闪烁。

这个内置 hook 插件用来控制 loading 状态的延迟。

```
const { loading, data } = useRequest(getUsername, {
  loadingDelay: 300
});

return <div>{ loading ? 'Loading...' : data }</div>
```

例如上面的场景，假如 getUsername 在 300ms 内返回，则 loading 不会变成 true，避免了页面展示 Loading... 的情况。

我们可以尝试着通过插件 hook 的方式来实现一下。默认情况下，loading 状态在接口 onBefore 的时候会被更新成 true。那我们就需要在接口 onBefore 的时候，将其改为 false，并加一个定时器，让其在指定时间之后再变为 true：

```
import { useRef } from 'react';
import type { Plugin, Timeout } from '../types';


const useLoadingDelayPlugin: Plugin<any, any[]> = (fetchInstance, { loadingDelay }) => {
  const timerRef = useRef<Timeout>();
  // 用户没有设置loadingDelay时直接返回
  if (!loadingDelay) {
    return {};
  }
  // 取消定时器。
  const cancelTimeout = () => {
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }
  };
  return {
    // 更改loading的状态为false，并加个定时器，让其在指定的delay时间后在更正为true。
    onBefore: () => {
      cancelTimeout();
      timerRef.current = setTimeout(() => {
        fetchInstance.setState({
          loading: true,
        });
      }, loadingDelay);
      return {
        loading: false,
      };
    },
    // 在请求完成或取消时，记得销毁定时器。
    onFinally: () => {
      cancelTimeout();
    },
    onCancel: () => {
      cancelTimeout();
    },
  };
};
export default useLoadingDelayPlugin;
```

...

### ready 和 refreshDeps

useRequest 支持 ready 配置，当 options 中的 ready 值为  `false`  时，请求永远都不会发出。其具体行为如下：

1.  当  `manual=false`  自动请求模式时，每次  `ready`  从  `false`  变为  `true`  时，都会自动发起请求，会带上参数  `options.defaultParams`。
2.  当  `manual=true`  手动请求模式时，只要  `ready=false`，则通过  `run/runAsync`  触发的请求都不会执行。

支持依赖刷新，当 options 中的 refreshDeps 值变化后，会重新触发请求。

由于这两个需要类似，可以放在一个 hook 里实现

```js
import { useRef } from 'react';
import useUpdateEffect from '../../../useUpdateEffect';
import type { Plugin } from '../types';

// support refreshDeps & ready
const useAutoRunPlugin: Plugin<any, any[]> = (
  fetchInstance,
  {
    manual,
    ready = true,
    defaultParams = [],
    refreshDeps = [],
    refreshDepsAction,
  }
) => {
  // 避免重复请求的ref。
  const hasAutoRun = useRef(false);
  hasAutoRun.current = false;

  // 在自动模式（manual === false）且ready === true时发送请求。
  useUpdateEffect(() => {
    if (!manual && ready) {
      hasAutoRun.current = true;
      fetchInstance.run(...defaultParams);
    }
  }, [ready]);

  // 在自动模式下，当refreshDeps发生变化，会出发refresh事件。
  useUpdateEffect(() => {
    if (hasAutoRun.current) {
      return;
    }
    if (!manual) {
      hasAutoRun.current = true;
      fetchInstance.refresh();
    }
  }, [...refreshDeps]);

  return {
    // 在请求前判断ready状态，等于true才能发送请求。
    onBefore: () => {
      if (!ready) {
        return {
          stopNow: true,
        };
      }
    },
  };
};

// 还有一个onInit事件，初始化loading的状态，在自动模式且ready为true时状态才为true。
useAutoRunPlugin.onInit = ({ ready = true, manual }) => {
  return {
    loading: !manual && ready,
  };
};

export default useAutoRunPlugin;
```

...

其他插件大同小异，可以前往 ahooks 仓库查看<https://github.com/alibaba/hooks/tree/master/packages/hooks/src/useRequest/src/plugins>

end.

