# Redirect

>The public API for navigating programmatically with a component.  --- author

Redirect.js 顾名思义，是做重定向而使用的组件

### 代码 & 逐行分析

```js
function Redirect({ computedMatch, to, push = false }) {
  return (
    // 还是作为 Router 子组件使用
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <Redirect> outside a <Router>");

        const { history, staticContext } = context;
        //  操作函数 push vs replace
        const method = push ? history.push : history.replace;
        // 操作函数的参数对象
        const location = createLocation(
          computedMatch
            ? typeof to === "string"
            // 如果传入的 to 属性值是字符串还是对象使用不同的方法处理成 location 对象
              ? generatePath(to, computedMatch.params)
              : {
                  ...to,
                  pathname: generatePath(to.pathname, computedMatch.params)
                }
            : to
        );

        // When rendering in a static context,
        // set the new location immediately.
        if (staticContext) {
          method(location);
          return null;
        }

        return (
          // 这个组件主要作用是提供生命周期函数调用，在不同的时期做不同的操作
          <Lifecycle
            onMount={() => {
              // didMount 执行一次路由操作
              method(location);
            }}
            onUpdate={(self, prevProps) => {
              const prevLocation = createLocation(prevProps.to);
              // 在didUpdate时判断 to 是否改变了，如果变了，就执行路由操作
              if (
                !locationsAreEqual(prevLocation, {
                  ...location,
                  key: prevLocation.key
                })
              ) {
                method(location);
              }
            }}
            // 这里传入的 to === prevProps.to
            to={to}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

// 开发环境的代码就省略了
if (__DEV__) { }

export default Redirect;


```

### 总结

- 可以简单理解成 push/replace(props.to) 操作
- 增加了didUpdate 更新的处理