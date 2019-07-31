# BrowserRouter

>The public API for a <Router> that uses HTML5 history.  --- author

BrowserRouter 使用 createBrowserHistory api 创建路由

### 代码 & 逐行分析

```js
import { Router } from "react-router";
import { createBrowserHistory as createHistory } from "history";

class BrowserRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}


export default BrowserRouter;

```

