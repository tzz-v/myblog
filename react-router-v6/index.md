# react-router v6


### 前言

之前一直负责老项目的迭代，而且很少关注 router 相关的代码，直到上个月公司开了一个新项目，当我着手开始配置路由时突然发现，嗯？Switch 标签咋没了？Route 里的 component 属性咋也没了。意识到不妙的我赶紧去翻了翻 react router 的文档，发现 react router 早就在 21 年底就偷偷升级到了 v6 且变更极大（嗯 这是一个悲伤的故事，现在甚至已经更新到了 6.4.1）

之前的老项目的包版本是 5.2 的，要升级的话成本有点高，也懒得搞，就那样吧。但新项目肯定不能将就啊。那就整呗

### 正文

首先先说一下 react-router 和 react-router-dom 的区别，react-router 实现了路由的核心功能，react-router-dom 则是基于 react-router，又加了在浏览器运行环境下的一些功能，比如把 Link 组件渲染成一个 a 标签，再比如 HashRouter 会使用 window.location.hash 和 hashChange 事件构建路由。一般在项目中只需要引入 react-router-dom 包即可。

```
npm install react-router-dom
```

#### BrowserRouter

新项目使用浏览器路由即 BrowserRouter，要想在项目中使用 React Router，需要 App 组件嵌套进 BrowserRouter 组件中。

```
ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>
);
```

#### Routes 和 Route

我们使用 Route 组件来定义路由，并使用 Routes 包裹。

Route 组件接收两个 props：

- path?: string | 页面 url
- element?： React.ReactNode |当前 url 需要加载的元素

```
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/teams/:id" element={<Teams />} />
  <Route path="/teams/new" element={<NewTeams />} />
  <Route path="*" element={<PageError />} />
</Routes>
```

```
<Switch>
  <Route path="/teams/:id" component={Teams} />
  <Route path="/teams/new" component={NewTeams} />
</Switch>
```

V6 的 Route 组件不在要求我们按一定的顺序来定义路由,在旧版本中，当多个路由匹配一个模糊的 url 时，我们必须以某种方式对路由进行排序，v6 版本则会替我们选择最具体的匹配。/teams/new 将匹配这两个路由，但只会渲染 NewTeams 组件。

支持的格式：

- /groups
- /groups/admin
- /users/:id
- /users/:id/messages
- /files/\*
- /files/:id/\*

#### 嵌套路由

react-router V6 支持路由的嵌套，允许父路由控制子路由的渲染，

提供了一个渲染出口组件 Outlet，当匹配到子路由的时候会渲染在 Outlet 组件所在的位置，父路由此时仍然存在。

支持多级嵌套

```
const src = () => {
  return (
    <>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/list" element={<List />}>
          <Route path="Detail/:id" element={<Detail />} />
        </Route>
      </Routes>
    </>
  )
}

const List = () => {
  const navigate = useNavigate();
  return (
    <>
      <div className="list">
        <Button
          onClick={() => {
            navigate('/list/detail/1');
          }}
        >
          detail1
        </Button>
        <Button
          onClick={() => {
            navigate('/list/detail/2');
          }}
        >
          detail2
        </Button>
        // react-router提供了Outlet组件用来占位置
        <Outlet />
      </div>
    </>
  );
```

#### 查询参数

/mysql?mysql_group_id=mysql-test&mysql_instance_id=mysql-asfasfew

react-router V6 提供了 useSearchParams Hook，它是基于浏览器内置的 URLSearchParams 构造函数进行的封装。

useSearchParams 类似 useState，返回一个数组，第一个元素是一个 Map，可以通过其 get 方法获取查询字符串中的值，第二个元素是一个函数，用来更新 url 中的查询字符串。

```
import { useSearchParams } from 'react-router-dom';

const [searchParams, setSearchParams] = useSearchParams();

const groupId = searchParams.get('mysql_group_id')
const instanceId = searchParams.get('mysql_instance_id')

const updateOrder = (sort) => {
  setSearchParams({ key: value })
}
```

#### 编程式配置路由

react-router V6 还提供了 useRoutes Hook，使我们可以通过写一个配置对象的方式来配置路由，不需要在自行拼接 jsx.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9c3dd7664664973b0661b29800bb814~tplv-k3u1fbpfcp-zoom-1.image)

```
const routerConfig = [
  {
    path: "/",
    key: "home",
    label: "common.nav.menu.home",
    icon: <HomeOutlined />,
    element: <Home />,
  },
  {
    key: "authManageWrapper",
    label: "common.nav.menu.authManage",
    icon: <SolutionOutlined />,
    children: [
      {
        path: "/auth/list",
        key: "authList",
        label: "common.nav.menu.authList",
        element: <AuthList />,
      },
      {
        path: "/auth/add",
        key: "addAuth",
        label: "common.nav.menu.addAuth",
        element: <AddAuth />,
      },
    ],
  },
  {
    path: "*",
    hideInMenu: true,
    key: "null",
    element: <Navigate to="/" />,
  },
]
return (
  <>
    {useRoutes(routerConfig)}
  </>
)
```

在 antd 的 4.20 版本+中，他的 Menu 导航组件也支持这种写法，可以维护一份 routerConfig，对其扩充一些属性，就能同时再 Menu 组件中使用。

####

#### 其他

- 新增 useNavigate Hook 代替 useHistory Hook

```
let navigate = useNavigate();
<button
  onClick={() => {
    navigate("/invoices" + location.search);
  }}
>
  jump
</button>
```

- 可以在各个地方渲染多组路由：经常有一些需求让我们在某个 state 为 true 时展示组件 A，为 false 时展示组件 B，这个时候可以尝试在组件内写一个嵌套路由。

#### 总结：

- 定义路由时 Routes 组件和 Route 组件搭配使用，且可以在不同地方定义多组。Route 组件的属性也发生了变更。
- 新增了 Outlet 组件，支持路由的嵌套，使用起来可以更加的灵活。
- 新增了一个可以查询和更新 url 参数的 hook useSearchParams。
- 提供了 useRoutes Hook， 支持使用 js 对象的形式来定义路由。

