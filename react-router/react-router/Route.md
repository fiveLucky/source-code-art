# Route

>The public API for matching a single path and rendering.  --- author

Route.js 主要负责 **路由匹配** 和 **渲染子组件**

### 代码 & 逐行分析

```js

function isEmptyChildren(children) {
  return React.Children.count(children) === 0;
}

/**
 * The public API for matching a single path and rendering.
 */
class Route extends React.Component {
  render() {
    return (
      // 使用react context最新的api，
      <RouterContext.Consumer>
      // context 是 Router 传下来的 value 属性
        {context => {
          invariant(context, "You should not use <Route> outside a <Router>");

          // 缓存 location 对象，默认会使用 context.location
          const location = this.props.location || context.location;
          // computedMatch 这个属性只有 Switch 才会传递
          const match = this.props.computedMatch
            ? this.props.computedMatch // <Switch> already computed the match for us
            : this.props.path
            // 通常情况下会走这个分支，这个方法主要是使用 pathname 和 传入的 path 做匹配，
            // 会使用props里的其他属性做多层判断
              ? matchPath(location.pathname, this.props)
              : context.match;

          const props = { ...context, location, match };

          let { children, component, render } = this.props;

          // Preact uses an empty array as children by
          // default, so use null if that's the case.
          if (Array.isArray(children) && children.length === 0) {
            children = null;
          }
          // children 属性可以是 function 
          if (typeof children === "function") {
            children = children(props);

            if (children === undefined) {
              if (__DEV__) {
                const { path } = this.props;

                warning(
                  false,
                  "You returned `undefined` from the `children` function of " +
                    `<Route${path ? ` path="${path}"` : ""}>, but you ` +
                    "should have returned a React element or `null`"
                );
              }

              children = null;
            }
          }

          return (
            // 支持嵌套 Route
            <RouterContext.Provider value={props}>
              {children && !isEmptyChildren(children)
                ? children
                // 如果 匹配正确 就渲染子元素
                : props.match
                  ? component
                  // 如果传入了 component 属性，就注入一些 props
                    ? React.createElement(component, props)
                    : render
                    // 如果传入了 render 属性，就使用 返回值
                      ? render(props)
                      : null
                  : null}
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}
// 开发环境的代码就省略了
if (__DEV__) { }

export default Route;


```

### 总结

- 配合 Router 结合 context api 实现了 组件间层级通信，Route 支持嵌套
- 支持 children component render 三种传递子元素的方式