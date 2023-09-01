# 读react新官方文档，一些 hook 的使用


# 前言

React 的新官方文档更新了有一段时间了，最近看了下，发现有不少新东西。

# 一些默默出现的新的 hooks

## useId

在 React 中直接编写 ID 并不是一个好的习惯，一个组件可能会被渲染多次，但在 DOM 树中， ID 必须是唯一的。

不过有些时候我们必须得给组件写个 ID，比如使用 Antv 注册一个可视化图表组件的时候，我们需要通过 id 进行绑定。此时如果我们在外部多次使用这个组件的时候，为了保证 ID 的唯一性，我们需要将 id 作为一个 props 传递给这个图表组件

```
import { Chart } from '@antv/g2';

const CommonChart: FC<{ id: 'string', data: any[] }> = ({ id, data }) => {
    const lineChart = new Chart({
    container: id,
    autoFit: true,
    height: config.height ?? 400,
    padding: [20, 20, 30, 40],
      });
    //...

    return <div id={id} />
}

const App = () => {
    return (
        <>
            {/*年龄图表*/}
            <CommonChart id='age-chart' data={ageData} />
            {/*成绩图表*/}
            <CommonChart id='achievement-chart' data={achievementData} />
        </>
    )
}
```

现在有了`useId` hook，我们可以使用更简洁的办法来达到相同的目的

```
import { Chart } from '@antv/g2';

const CommonChart: FC<{ data: any[] }> = ({ data }) => {
    const chartId = React.useId();

    const lineChart = new Chart({
        container: chartId,
                    // 如果不想在DOM上看到没有语意的随机ID，也可以把它当成id前缀
                    // container: `${chartId}-line-chart`,
        autoFit: true,
        height: config.height ?? 400,
        padding: [20, 20, 30, 40],
      });
    //...

    return <div id={chartId} />
}

const App = () => {
    return (
        <>
            {/*年龄图表*/}
            <CommonChart data={ageData} />
            {/*成绩图表*/}
            <CommonChart data={achievementData} />
        </>
    )
}
```

现在，即使 `CommonChart` 多次出现在屏幕上，生成的 ID 并不会冲突。

## useSyncExternalStore

`useSyncExternalStore` 是一个让你订阅外部 store 的 React Hook。

我们的多数组件只会从它的 props、state 和 context 中获取数据，然而，有时一个组件需要一些 react 之外的 store 读取一些随时间变化的数据，比如在 React 之外持有状态的第三方状态管理库，又或者暴露出一个可变值及订阅其改变事件的浏览器 API。 前者我想不出具体的场景，不过在项目中，我们经常会订阅一些浏览器 API 的变化，如实时获取视口宽度等，并进行相应的处理。

比如监听滚动条变化，显示置底按钮：

```
// 项目节选
import { useEffect, useState } from 'react';

const ToBottomButton = () => {
    const [toBottomButtonVisible, setToBottomButtonVisible] = useState(true);

    useEffect(() => {
        const dom = document.querySelector('#root .content');
        if (!dom) return;
        const handleScrollChange = () => {
          // interval = dom的总高度 - 已滚动的高度 - 正在显示的高度
          const interval = dom.scrollHeight - dom.scrollTop - dom.clientHeight;
          setToBottomButtonVisible(interval > 80);
        };
        dom.addEventListener('scroll', handleScrollChange);
        return () => {
          dom.removeEventListener('scroll', handleScrollChange);
        };
  }, []);

    return (
        toBottomButtonVisible && (
          <Button
            shape="circle"
            icon={<VerticalAlignBottomOutlined />}
            onClick={moveToBottom}
            type="primary"
          />
    )
  );
}
```

现在有了`useSyncExternalStore` hook，我们可以通过`useSyncExternalStore` hook 来读取 dom 距离底部的距离。

`useSyncExternalStore` hook 接收两个必填参数：

- subscribe：一个订阅函数；该函数需要返回清除订阅的函数。
- getSnapShot: 一个回调函数，通过这个回调，返回我们需要的数据。

```
// useBottomInterval 自定义hook；
import { useSyncExternalStore } from 'react';

const useBottomInterval = (dom: Element | null) => {
  const getBottomInterval = () => {
    if (!dom) return 0;
    return dom.scrollHeight - dom.scrollTop - dom.clientHeight;
  };
  const subscribe = (callback: any) => {
    dom?.addEventListener('scroll', callback);
    return () => {
      dom?.removeEventListener('scroll', callback);
    };
  }
  const bottomInterval = useSyncExternalStore(subscribe, getBottomInterval);
  return bottomInterval;
};

export default useBottomInterval;
```

新的置底 button 组件

```
import useBottomInterval from '~/hooks/useBottomInterval';

const ToBottomButton = () => {
  const bottomInterval = useBottomInterval(
    document.querySelector('#root .content')
  );
  const toBottomButtonVisible = bottomInterval > 80;

  return (
    toBottomButtonVisible && (
      <Button
        shape="circle"
        icon={<VerticalAlignBottomOutlined />}
        onClick={moveToBottom}
        type="primary"
      />
    )
  );
}
```

可以把常需要订阅的浏览器 API 封装成 hook，即插即用。

```
// 实时获取窗口宽度的hook：useWindowWidth
export const useWindowWidth = () => {
  const bottomInterval = useSyncExternalStore(subscribe, getWindowWidth);
  return bottomInterval;
};

// 重新渲染时传入一个不同的 subscribe 函数，React 会用新传入的 subscribe 函数
// 重新订阅该 store。我们可以通过在组件外声明 subscribe 来避免。
const getWindowWidth = () => window.innerWidth;

const subscribe = (callback: any) => {
  window.addEventListener('resize', callback);
  return () => {
    window.removeEventListener('resize', callback);
  };
}
```

# 不必要的 state 和 Effect

如果一个值可以基于现有的 props 或 state 计算得出，就没必要把它也作为一个 state，多余的 effect 会导致一些不必要的 re-render。可以在渲染期间直接计算出这个值，这将使你的代码更快、更简洁，以及更少出错。当这个计算比较昂贵的时候可以通过`useMemo` hook 缓存这个昂贵的计算。

```
import { useState } from 'react';
const Page = () => {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // const [fullName, setFullName] = useState('');
  // useEffect(() => {
  //   setFullName(`${firstName}-${lastName}`);
  // }, [firstName, lastName]);

  const name = `${firstName}-${lastName}`

  // ...
}
```

```
import { useState } from 'react';
const Page = () => {
  const [newTodo, setNewTodo] = useState('');

  // const [visibleTodos, setVisibleTodos] = useState([]);
  // useEffect(() => {
  //   setVisibleTodos(getFilteredTodos(todos, filter)); //一个复杂的筛选计算
  // }, [todos, filter]);

  const visibleTodos = getFilteredTodos(todos, filter);
    // or
  const visibleTodos = useMemo(() => {
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);

        // ...
}
```

# 关于在 useEffect 中获取数据

在日常开发中，常常有这样一个场景，当某个请求参数变化时，我们需要根据参数的变化重新发送请求，

```
const [userId, setUserId] = React.useState();
const [data, setData] = React.useState();

React.useEffect(() => {
    getUserInfo({ userId }).then((res) => { // 某个ajax请求
            setData(res);
    });
}, [user]);

// userId:
// undefined -> 'user-1';
// 'user-1' -> 'user-2';
```

上述这种 在 effect 中的请求调用是一种获取数据的常用方法，但这种方法存在一个不容易触发的 bug，当状态`user` 发生两次 change 的时候，请求`getUserInfo` 也会发送两次，我们虽然可以控制请求发送的先后顺序，但我们控制不了请求响应的顺序，尤其是在网速比较差劲而这个请求又比较耗时的时候。在这种情况下，后发送的请求可能会先进行响应，先发送请求反而后进行响应，从而导致脏数据。可以想象，当`setData`被执行了两次之后，我的`user` 状态是‘user-2’，`data` 却是通过‘user-1’响应的数据。

### 解决

```
useEffect(() => {
  let ignore = false;
  getUserInfo({ userId }).then((res) => { // 某个ajax请求
    if (!ignore) {
            setData(res);
    }
  });
  return () => {
    ignore = true;
  };
}, [userId]);
```

我们无法‘撤销’已经发送的请求，但我们可以忽略请求所响应的结果，在 effect 内部维护一个 ignore 变量，通过闭包，组件被卸载之后所响应的结果，会被我们忽略，以保证不相关的响应不会继续影响我们的程序，userId 如果从‘user-1’变成了‘user-2’，ignore 属性可以确保我们忽略‘user-1’的响应，及时这个响应在‘user-2’响应之后到达。

# 关于 antd ProTable 的 feature/bug 引发的问题

看了上面关于 effect 容易引发的 bug，我想我们项目里的 table 好像没怎么处理相关的请求，带筛选功能的 Table 组件都能看到这样一段没有经过忽略处理的代码：

```
useEffect(() => {
  refreshTable(); // 刷新请求
}, [tableState.refresh, tableState.filteredInfo]);
```

从刚才得到的结论来看，如果我降低页面的网速，并快速的设置或移除多个筛选项的话，多试几次，总能偶尔触发刚刚说的 bug，不过我试试了几次，并没有出现这种 bug，反而出现了别的 bug：

我两次触发了 refreshTable 刷新请求，第二次请求会被忽略。

出现这种情况，应该 ProTable 内部做了某种处理。然后我看了下 proTable 的源码发现了问题出现的原因：

<img src="demo.png" alt="image.png" width="400" />

在 table 请求的 loading 状态为 true 时，他不会处理后续相同的请求，直至当前请求结束。这个效果看起来有点类似节流，只是节流的时间不固定，取决于请求所耗的时间。

不过我觉得这样的处理方式，相对于 bug，更像是个 feature，页面初始化时他可以避免我们频繁发送相同的请求，虽然有些过于一刀切了。如果这样的行为可以让用户进行控制或许会更好。

## 解决办法

显然，即便我通过闭包进行忽略处理，也无法解决，毕竟他的第二次请求根本就没有发出去。不过我可以让筛选、排序、刷新等会发送请求的动作和 table 的 loading 进行绑定，让 table loading 为 true 的时候，禁止用户触发 refresh 相关动作，直至当前的请求完成。
又或者比较糙的办法，我添加一个请求等待，在第一次请求完毕之后在立刻触发最新的请求。（抖机灵）。

end.

