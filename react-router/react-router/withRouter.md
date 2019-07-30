# withRouter

>A public higher-order component to access the imperative API.  --- author

withRouter 就是一个高阶组件，为组件注入 context 里的 props `{ history, location, match, staticContext }`。



### 代码 & 逐行分析

```js
import React from "react";
import PropTypes from "prop-types";
import RouterContext from "./RouterContext";
// 这个库是用来 merge 组件 static 属性方法，并返回第一个参数组件
import hoistStatics from "hoist-non-react-statics";
import invariant from "tiny-invariant";

function withRouter(Component) {
  const displayName = `withRouter(${Component.displayName || Component.name})`;
  const C = props => {
    // 暴露被包装组件的 ref
    const { wrappedComponentRef, ...remainingProps } = props;

    return (
      <RouterContext.Consumer>
        {context => {
          invariant(
            context,
            `You should not use <${displayName} /> outside a <Router>`
          );
          return (
            <Component
              {...remainingProps}
              {...context}
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };

  C.displayName = displayName;
  C.WrappedComponent = Component;

  if (__DEV__) {
    C.propTypes = {
      wrappedComponentRef: PropTypes.oneOfType([
        PropTypes.string,
        PropTypes.func,
        PropTypes.object
      ])
    };
  }

  return hoistStatics(C, Component);
}


export default withRouter;


```

### 总结

- 可以简单理解成 push/replace(props.to) 操作
- 增加了didUpdate 更新的处理