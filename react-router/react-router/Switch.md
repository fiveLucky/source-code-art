# Switch

>The public API for rendering the first <Route> that matches.  --- author

Switch.js 主要负责 **路由匹配** 和 **渲染子组件**

### 代码 & 逐行分析

```js

class Switch extends React.Component {
  render() {
    return (
      // 作为 Router 子组件 使用
      <RouterContext.Consumer>
        {context => {
          invariant(context, "You should not use <Switch> outside a <Router>");

          const location = this.props.location || context.location;

          // 定义两个变量，缓存 匹配成功的元素，和处理过的 match 对象
          let element, match;

          // We use React.Children.forEach instead of React.Children.toArray().find()
          // here because toArray adds keys to all child elements and we do not want
          // to trigger an unmount/remount for two <Route>s that render the same
          // component at different URLs.

          // 遍历个子组件，判断 location.pathname 和 path 的匹配状态
          React.Children.forEach(this.props.children, child => {
            if (match == null && React.isValidElement(child)) {
              element = child;

              const path = child.props.path || child.props.from;

              match = path
                ? matchPath(location.pathname, { ...child.props, path })
                : context.match;
            }
          });

          return match
          // 把匹配到的元素注入额外的属性，computedMatch 传给 Route
            ? React.cloneElement(element, { location, computedMatch: match })
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}

// 开发环境的代码就省略了
if (__DEV__) { }

export default Switch;


```

### 总结

- Switch 组件必须用在 Router 里面
- Route 可以用在 Switch 里面