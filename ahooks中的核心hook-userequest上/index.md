# ahooks中的核心hook-useRequest（上）


开启掘金成长之旅！这是我参与「掘金日新计划 · 12 月更文挑战」的第 1 天，[点击查看活动详情](https://juejin.cn/post/7167294154827890702 'https://juejin.cn/post/7167294154827890702')

## 前言

useRequest 是一个异步数据管理的 hooks，是 ahooks Hooks 库的核心 hook，因为其通过插件式组织代码，大部分功能都通过插件的形式来实现，所以其核心代码行数较少，简单易懂，还可以支持我们自定义扩展功能。可以说，useRequest 能处理 React 项目绝大多数的网络请求场景。

让咱自己写可能写不出来，那就先从模仿开始，通过阅读 useRequest 的代码，从中学习大佬们的代码逻辑和思维处理。

> ahooks： <https://ahooks.js.org/>

文中代码基于 3.7.2 版本

## 前置 hook 了解

在 useRequest 的源码实现中使用到了一些其他的 hooks

- useCreation：useMemo 或 useRef 的替代品。
- useLatest：返回当前最新值的 Hook， 可以避免闭包问题。
- useMemoizedFn：useCallback 的替代品 。
- useMount：只在组件初始化时执行的 Hook。
- useUnmount：在组件卸载（unmount）时执行的 Hook。
- useUpdate：强制组件重新渲染的 hook。

## 实现最基础的 useRequest Hook

```js
const { data, error, loading, cancel } = useRequest(service);
```

useRequest 通过定义一个 class 类 Fetch 来维护相关的数据（data，loading 等）和方法（run， refresh 等）然后在 useRequestImplement 中创建 Fetch 实例，并返回实例属性和方法。

我对代码进行了拆分，保留了 useRequest 中核心的功能，该 hook 接收一个 promise，需要返回 data、error、loading、cancel 状态。

```js
const useRequestImplement = (service: Promise<any>) => {
  const serviceRef = useLatest(service);
  const update = useUpdate();

  const fetchInstance = useCreation(() => {
    return new Fetch(serviceRef, update);
  }, []);
  useMount(() => {
    // useCachePlugin can set fetchInstance.state.params from cache when init
    const params = fetchInstance.state.params ?? [];
    // @ts-ignore
    fetchInstance.runAsync(...params);
  });
  useUnmount(() => {
    fetchInstance.cancel();
  });
  return {
    loading: fetchInstance.state.loading,
    data: fetchInstance.state.data,
    error: fetchInstance.state.error,
    params: fetchInstance.state.params || [],
    cancel: useMemoizedFn(fetchInstance.cancel.bind(fetchInstance)),
  };
};
```

```js
export default class Fetch<TData, TParams extends any[]> {
  public count: number = 0;

  public state = {
    loading: false,
    params: undefined,
    data: undefined,
    error: undefined,
  };
  constructor(public serviceRef, public subscribe) {}
  setState(s) {
    this.state = {
      ...this.state,
      ...s,
    };
    this.subscribe();
  }
  async runAsync(...params: TParams): Promise<any> {
    this.count += 1;
    const currentCount = this.count;

    this.setState({
      loading: true,
      params,
    });
    try {
      const servicePromise = this.serviceRef.current(...params);
      const res = await servicePromise;
      if (currentCount !== this.count) {
        return new Promise(() => {});
      }
      this.setState({
        data: res,
        error: undefined,
        loading: false,
      });
      return res;
    } catch (error) {
      if (currentCount !== this.count) {
        // prevent run.then when request is canceled
        return new Promise(() => {});
      }
      this.setState({
        error,
        loading: false,
      });

      throw error;
    }
  }

  cancel() {
    this.count += 1;
    this.setState({
      loading: false,
    });
  }

  refreshAsync() {
    // @ts-ignore
    return this.runAsync(...(this.state.params || []));
  }
}
```

## 实现剩余核心功能

接收用户的自定义配置，包括 manual(手动模式)和一些回调函数（onBefore,onSuccess, onError,onFinally）

```js
const useRequestImplement = (service: Promise<any>, options) => {
  const { manual = false, ...rest } = options;
  const fetchOptions = {
    manual,
    ...rest,
  };
  // const serviceRef = useLatest(service);
  // const update = useUpdate();

  // const fetchInstance = useCreation(() => {
  return new Fetch(serviceRef, fetchOptions, update);
  // }, []);
  fetchInstance.options = fetchOptions;
  useMount(() => {
    if (!manual) {
      const params = fetchInstance.state.params || options.defaultParams || [];
      // @ts-ignore
      fetchInstance.run(...params);
    }
  });
  // useUnmount(() => {
  //   fetchInstance.cancel();
  // });
  return {
    //   loading: fetchInstance.state.loading,
    //   data: fetchInstance.state.data,
    //   error: fetchInstance.state.error,
    //   params: fetchInstance.state.params || [],
    //   cancel: useMemoizedFn(fetchInstance.cancel.bind(fetchInstance)),
    refresh: useMemoizedFn(fetchInstance.refresh.bind(fetchInstance)),
    refreshAsync: useMemoizedFn(fetchInstance.refreshAsync.bind(fetchInstance)),
    run: useMemoizedFn(fetchInstance.run.bind(fetchInstance)),
    runAsync: useMemoizedFn(fetchInstance.runAsync.bind(fetchInstance)),
    mutate: useMemoizedFn(fetchInstance.mutate.bind(fetchInstance)),
  };
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
  constructor(public serviceRef, public options, public subscribe) {
    this.state = {
      ...this.state,
      loading: !options.manual,
    };
  }
  // setState(s) {
  //   this.state = {
  //     ...this.state,
  //     ...s,
  //   };
  //   this.subscribe();
  // }
//   async runAsync(...params: TParams): Promise<any> {
//     this.count += 1;
//     const currentCount = this.count;
//
//     this.setState({
//       loading: true,
//       params,
//     });
//
    this.options.onBefore?.(params);
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
      this.options.onSuccess?.(res, params);
      this.options.onFinally?.(params, res, undefined);
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
      this.options.onError?.(error, params);
      this.options.onFinally?.(params, undefined, error);
//       throw error;
//     }
//   }


//   cancel() {
//     this.count += 1;
//     this.setState({
//       loading: false,
//     });
//   }
//
//   refreshAsync() {
//     // @ts-ignore
//     return this.runAsync(...(this.state.params || []));
//   }


  run(...params: TParams) {
    this.runAsync(...params).catch((error) => {
      if (!this.options.onError) {
        console.error(error);
      }
    });
  }
  refresh() {
    // @ts-ignore
    this.run(...(this.state.params || []));
  }


  mutate(data?: TData | ((oldData?: TData) => TData | undefined)) {
    const targetData = isFunction(data) ? data(this.state.data) : data;
    this.setState({
      data: targetData,
    });
  }
}
```

#### runAsync 和 run

runAsync 方法返回一个 promise，使用 runAsync 时，当请求报错会中断后续操作，需要手动捕获异常。

run 方法则对 runAsync 进行了封装，帮助我们了捕获异常，或可以通过 options.onError 来处理异常行为。

#### refresh 和 refreshAsync

useRequest 维护了一份 params，调用 run()和 runAsync()的时候会同时更新 params。以便给 refresh 和 refreshAsync 方法使用

#### cancel

useRequest 维护了一个 count。

而 runAsync 方法本身也维护一个 currentCount。

每次调用 runAsync 时，count 进行一次++操作，然后将其赋值给 currentCount。

每次 cancel 方法 count 会再进行一次++操作。通过比较 count 和 currentCount 的值来判断用户是否进行了取消操作，进行相应的处理

#### mutate

支持立即修改 useRequest 返回的 data 参数。

mutate 的用法与 React.setState 一致，支持 mutate(newData) 和 mutate((oldData) => newData) 两种写法。

## 小结

以上是 useRequest hook 的基本功能，剩余功能如 loading 状态延时、请求防抖、节流、数据缓存等功能都是通过插件的形式进行实现的，具体实现可以看 ahooks 中的核心 hook-useRequest（下）

