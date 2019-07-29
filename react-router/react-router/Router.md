# Router

>The public API for putting history on context.  --- author

Router.js 主要封装了一个基础的Router组件，为整个路由提供了一个框架能力

### 代码 & 逐行分析

```js

class Router extends React.Component {
  static computeRootMatch(pathname) {
    return { path: "/", url: "/", params: {}, isExact: pathname === "/" };
  }

  constructor(props) {
    super(props);

    this.state = {
      // 这个组件接收一个 history 参数，
      // 这个history是 history库（https://github.com/ReactTraining/history）的一个实例
      // 可以是 hashHistory、browserHistory、memoryHistory
      location: props.history.location
    };

    // This is a bit of a hack. We have to start listening for location
    // changes here in the constructor in case there are any <Redirect>s
    // on the initial render. If there are, they will replace/push when
    // they mount and since cDM fires in children before parents, we may
    // get a new location before the <Router> is mounted.

    // 定义两个变量，判断初始化时是否存在 Redirect 路由组件
    this._isMounted = false;
    this._pendingLocation = null;

    if (!props.staticContext) {
      // 如果不是静态路由，注册回调函数，这里的回调函数会在路由变动的时候执行，由 history 代码操作
      this.unlisten = props.history.listen(location => {
        if (this._isMounted) {
          this.setState({ location });
        } else {
          // 走到这个分支的情况是： 子组件在RouterdidMount前操作了路由，比如 Redirect 在didMount时对路由进行 push/replace，
          // 子组件的didMount是先于父组件 didMount 执行的，这里缓存 新的 location 对象

          // 为什么不直接 setState ，而是缓存下来，在didMount阶段执行？
          // 这是为了避免子组件多次操作路由（如 存在多个 Redirect 组件）时，导致这里多次调用 setState
          this._pendingLocation = location;
        }
      });
    }
  }

  componentDidMount() {
    this._isMounted = true;

    if (this._pendingLocation) {
      // 只在这里进行一次 setState 即可
      this.setState({ location: this._pendingLocation });
    }
  }

  componentWillUnmount() {
    // 关闭事件监听
    if (this.unlisten) this.unlisten();
  }

  render() {
    return (
      // 使用新的 context api
      <RouterContext.Provider
        children={this.props.children || null}
        value={{
          history: this.props.history,
          location: this.state.location,
          match: Router.computeRootMatch(this.state.location.pathname),
          staticContext: this.props.staticContext
        }}
      />
    );
  }
}

// 开发环境
if (__DEV__) {
  // 属性类型校验
  Router.propTypes = {
    children: PropTypes.node,
    history: PropTypes.object.isRequired,
    staticContext: PropTypes.object
  };

  // 重写 componentDidUpdate
  Router.prototype.componentDidUpdate = function(prevProps) {
    // 校验History属性
    warning(
      prevProps.history === this.props.history,
      "You cannot change <Router history>"
    );
  };
}

export default Router;

```

### 总结

- 在constructor里做路由监听，实现了初始化时的 Redirect
- 使用实例属性缓存最新的 location 对象，避免重复渲染
- 使用 react 最新的 context api，提供 provider，这样 Router 就是最外层的框架组件，其他路由组件必须在 Router 里面使用

代码简单易懂，可读性强