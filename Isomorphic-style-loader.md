### 项目架构
React + Mobx

使用isomorphic-style-loader做同构, 支持SSR(server side render)

### upgrade isomorphic-style-loader
isomorphic-style-loader v4 to v5

透传insertCss的方式有变化, v4依赖旧版context Api, v5依赖新版

difference: https://zh-hans.reactjs.org/docs/context.html#gatsby-focus-wrapper

```
// isomorphic-style-loader v5 接入方式
<StyleContext.Provider value={{ insertCss }} >
    <App />
</StyleContext.Provider>

...

// Class Component
import withStyles from 'isomorphic-style-loader/withStyles';

@withStyles(s)
class App {}

// Function Component
import useStyles from 'isomorphic-style-loader/useStyles';

function App { useStyles(s); }
```

详戳

https://github.com/kriasoft/isomorphic-style-loader/tree/v4.0.0

https://github.com/kriasoft/isomorphic-style-loader/tree/v5.1.0

### 引发问题

- mobx-react@5.2.3
- isomorphic-style-loader@5.1.0

```
// 问题组件声明
import { inject } from 'mobx-react';
import withStyles from 'isomorphic-style-loader/withStyles';

@inject('store')
@withStyles(s)
class App {
    render() {
        return (
            <div>{this.props.store.name}</div>
        );
    }
}
```

> Warning: declares both contextTypes and contextType static properties. The legacy contextTypes property will be igonred.
> Error: MobX injector: Store 'store' is not avaliable! Make sure it is provided by some Provider.

组件由两层HOC封装, withStyles声明了contextType, 使用hoist-non-react-statics(v3.3.1)提升了原组件的静态属性;

inject声明了contextTypes, 使用hoist-non-react-statics(v2.5.5)提升了withStyles(s)(Component)的静态属性, 此时组件里会存在contextTypes+contextType, 引发问题

### 解决方案
hoist-non-react-statics不应对contextType属性进行提升. contextType是个新api, 新版hoist-non-react-statics已经将其加入属性黑名单, 不提升。

difference:
https://github.com/mridgway/hoist-non-react-statics/blob/v2.5.5/src/index.js

https://github.com/mridgway/hoist-non-react-statics/blob/f3de655212b47f20b9c6a27e89a60d23f358925a/src/index.js#L9

- withStyles装饰器写在最外层, 此时contextTypes不会被提升到withStyles中.  规避方案, 不推荐
- 升级mobx-react, 更新hoist-non-react-statics依赖版本.  推荐















