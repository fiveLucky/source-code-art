# MemoryRouter

>The public API for a <Router> that stores location in memory.  --- author

MemoryRouter 使用 memoryHistory 创建的路由


### 代码 & 逐行分析

```js

import { createMemoryHistory as createHistory } from "history";

class MemoryRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}


export default MemoryRouter;

```
