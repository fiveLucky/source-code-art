# Link

>The public API for rendering a history-aware <a>.  --- author

Link 基于 a 标签开发的一个组件

### 代码 & 逐行分析

```js
import { __RouterContext as RouterContext } from "react-router";
import { resolveToLocation, normalizeToLocation } from "./utils/locationUtils";

function isModifiedEvent(event) {
  return !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey);
}

// 封装基础组件
function LinkAnchor({ innerRef, navigate, onClick, ...rest }) {
  const { target } = rest;

  return (
    <a
      {...rest}
      ref={innerRef} // TODO: Use forwardRef instead
      onClick={event => {
        // 拦截click事件
        try {
          if (onClick) onClick(event);
        } catch (ex) {
          event.preventDefault();
          throw ex;
        }

        if (
          !event.defaultPrevented && // onClick prevented default

          // event.button === 0 是监听鼠标左键按下，但是IE里鼠标左键按下返回 1，IE 下岂不是有问题？？？
          // 参考文档 https://developer.mozilla.org/zh-CN/docs/Web/API/event.button

          event.button === 0 && // ignore everything but left clicks
          (!target || target === "_self") && // let browser handle "target=_blank" etc.
          !isModifiedEvent(event) // ignore clicks with modifier keys
        ) {
          // 阻止 href 默认行为
          event.preventDefault();
          // 执行传入的回调
          navigate();
        }
      }}
    />
  );
}

// 默认component 是上面的 LinkAnchor
function Link({ component = LinkAnchor, replace, to, ...rest }) {
  return (
    // 作为 Router 子组件使用
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <Link> outside a <Router>");

        const { history } = context;

        // 如果 to 是个字符串，location 就是一个标准化的 location对象  { key, pathname, search, hash, state: {} }
        // 如果 to 是个函数，location 就是 函数的返回值
        // 通常情况下是一个 location 对象
        const location = normalizeToLocation(
          resolveToLocation(to, context.location),
          context.location
        );

        // 输出一个 pathname
        const href = location ? history.createHref(location) : "";

        return React.createElement(component, {
          ...rest,
          // 当 href 属性有效时，a 标签会先执行 click 事件，然后触发 href 行为（会刷新页面）
          // event.preventDefault(); 阻止了 href 行为
          href,
          // click 事件回调
          navigate() {
            const location = resolveToLocation(to, context.location);
            const method = replace ? history.replace : history.push;
            // 路由操作
            method(location);
          }
        });
      }}
    </RouterContext.Consumer>
  );
}

export default Link;

```

