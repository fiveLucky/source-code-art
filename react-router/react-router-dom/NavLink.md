# NavLink

>A <Link> wrapper that knows if it's "active" or not.  --- author

NavLink 支持样式的 Link

### 代码 & 逐行分析

```js
import { __RouterContext as RouterContext, matchPath } from "react-router";
import Link from "./Link";
import { resolveToLocation, normalizeToLocation } from "./utils/locationUtils";

function joinClassnames(...classnames) {
  return classnames.filter(i => i).join(" ");
}

function NavLink({
  // 参考规范中的定义  https://www.w3.org/TR/wai-aria-1.1/#aria-current
  "aria-current": ariaCurrent = "page",
  activeClassName = "active",
  activeStyle,
  className: classNameProp,
  // 是否匹配全部
  exact,
  isActive: isActiveProp,
  location: locationProp,
  // 是否严格匹配，例如大小写
  strict,
  style: styleProp,
  to,
  ...rest
}) {
  return (
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <NavLink> outside a <Router>");

        const currentLocation = locationProp || context.location;
        const { pathname: pathToMatch } = currentLocation;
        const toLocation = normalizeToLocation(
          resolveToLocation(to, currentLocation),
          currentLocation
        );
        const { pathname: path } = toLocation;
        // Regex taken from: https://github.com/pillarjs/path-to-regexp/blob/master/index.js#L202
        const escapedPath =
          path && path.replace(/([.+*?=^!:${}()[\]|/\\])/g, "\\$1");

        const match = escapedPath
          ? matchPath(pathToMatch, { path: escapedPath, exact, strict })
          : null;
        const isActive = !!(isActiveProp
          ? isActiveProp(match, context.location)
          : match);

        const className = isActive
          ? joinClassnames(classNameProp, activeClassName)
          : classNameProp;
        const style = isActive ? { ...styleProp, ...activeStyle } : styleProp;

        return (
          <Link
            aria-current={(isActive && ariaCurrent) || null}
            className={className}
            style={style}
            to={toLocation}
            {...rest}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

export default NavLink;

```

