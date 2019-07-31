# HashRouter

>The public API for a <Router> that uses window.location.hash.  --- author

HashRouter 使用 createHashHistory api 创建路由

### 代码 & 逐行分析

```js
import { Router } from "react-router";
import { createHashHistory as createHistory } from "history";

class HashRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}

export default HashRouter;

```

