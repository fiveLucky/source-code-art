# Prompt

>The public API for prompting the user before navigating away from a screen.  -- author

Prompt 组件主要目的是提供了路与变动后，离开当前页面前的**全局提示信息**，类似于 v3 的hook

### 代码 && 逐行分析

```js

function Prompt({ message, when = true }) {
  return (
    // 作为 Router 子组件使用
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <Prompt> outside a <Router>");
        // 通过 when 来控制是否生效
        if (!when || context.staticContext) return null;
        // 这个history.block 会处理 在创建 history 实例时传入的 getUserConfirmation 参数，
        // 把这个组件的props.message作为 getUserConfirmation 第一个参数 传给用户
        const method = context.history.block;

        return (
          <Lifecycle
            onMount={self => {
              self.release = method(message);
            }}
            onUpdate={(self, prevProps) => {
              if (prevProps.message !== message) {
                self.release();
                self.release = method(message);
              }
            }}
            // 在 unmount 阶段执行回调
            onUnmount={self => {
              self.release();
            }}
            message={message}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

if (__DEV__) {}

export default Prompt;

```