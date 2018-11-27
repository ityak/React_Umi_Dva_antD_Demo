# React_Umi_Dva_antD_Demo

- [dva 入门：手把手教你用写应用](https://juejin.im/entry/5879a63c1b69e600582da314)
- [React + Dva + Antd + Umi 快速入门](https://blog.csdn.net/SCU_Cindy/article/details/82432971)
- [dva 框架介绍](https://www.jianshu.com/p/8b7def32740f)
- [10分钟 让你dva从入门到精通](https://www.jianshu.com/p/69f13e9123d9)
- [UmiJS官方文档](https://umijs.org/zh/guide/#%E7%89%B9%E6%80%A7)
- [TypeScript 入门教程](https://ts.xcatliu.com/)

## 基础框架概念

- **React** 
    前端三大框架之一。
    
- **Dva** 
    由阿里架构师 sorrycc 带领 team 完成的一套前端框架，在作者的 github 里是这么描述它的：”dva 是 react 和 redux 的最佳实践”。
    
- **Antd** 
    是阿里的一套开箱即用的中台前端/设计解决方案，UI框架。
    
- **Umi** 
    一套可插拔的企业级 react 应用框架，同样由dva作者 sorrycc 完成。他在Umi中引入了 UI 工具 antd，打包工具 roadhog，路由 react-router和状态管理器 dva，做到了可插拔机制。

---

## Dva 初实践

一般来说，可以分为主要的三个部分`models`、`services`和`views`。其中`views`负责页面上的展示，这个不做赘述；`services`里面主要写一些请求后台接口的方法；`models`是其中最重要的概念，这里存放了各种数据，并对数据进行相应的交互。

### view层

```bash
import React, { Component } from 'react';
import { Form, Input } from 'antd';
import { connect } from 'dva/index';

@Form.create()
class View extends Component {
  render() {
    return(
      <div>
        <Form>
          <FormItem label="commitMessage" {...formItemLayout}>
            {getFieldDecorator('commitMessage', {
              rules: [{ type: 'string' }]
            })(<Input />)}
          </FormItem>
        </Form>
      </div>
        );
    }
}

const mapStateToProps = state => {
  const { 
    checkBranches
  } = state.projects;
  return {
    checkBranches
  };
};
export default connect(mapStateToProps)(View);
```
`View`层负责页面的展示问题，如`React`写法一致，最后通过`connect`方法应用`model`层的数据。

### Service层

```bash
import request from '@src/utils/request';

export function checkBranches({ id }) {
  return request(`/projects/${id}/branches`, {
    method: 'GET',
    headers: {
      'Content-type': 'application/json'
    }
  });
}
```

`Service`层主要负责存放请求后台接口的方法。这里的`request`封装了`fetch`函数，返回的是一个`promise`对象。`request`中传入两个参数，第一个是`url`是请求地址，第二个`options`是请求的参数，看情况传入，比如说这里传入了`method`和`headers`。

### Model层

```bash
import * as services from '@services/index';

export default {
  namespace: '',
  state: {},
  reducers: {},
  effects: {}.
  subscriptions: {}
}
```
`model`里面包括以下五部分：`namespace`、`state`、`reducers`、`effects`、`subscriptions`，缺一不可。注意，这里也需要从`service`层导入相应的方法。

- **namespace 命名空间**
```
namespace: 'projects'
```

- **state 相当于原生React中的state状态，用于存放数据的初始值。**
```
state: {
    projectsData: []
}
```

- **reducers 用于存放能够改变`view`的`action`，这里按照官方说明，不应该做数据的处理，只是用来`return state`，从而改变view层的展示。**
```
reducers: {
    getProjectAllData(state, action) {
        return { ...state, ...action.payload };
    },
}
```
- **effects 用于和后台交互，是处理异步数据逻辑的地方。**
```
effects: {
    *getAllProjects({ payload = {} }, { call, put }) {
        try {
            const res = yield call(projectsService.checkBranches, payload);
            yield put({
                type: 'getProjectData',
                payload: {
                    projectsData: res.data
                }
            });
        } catch (e) {
            message.warning(e.message);
        }
    },
}
```
- subscriptions 订阅监听，比如监听路由，进入页面如何如何，就可以在这里处理。相当于原生`React`中的`componentWillMount`方法。就比如上述代码，监听/project路由，进入该路由页面后，将发起`getAllProjects aciton`，获取页面数据。
```
subscriptions: {
    setup({ dispatch, history }) {
        return history.listen(({ pathname }) => {
            if (pathname === '/projects') {
                dispatch({
                    type: 'getAllProjects'
                });
            }
        });
    }
}
```
---
### Dva 数据流向

总的来说如下：View层操作 –> 触发models层effect中方法 –> 触发service层请求，获取后台数据 –> 触发model层处理相应数据的方法，存储至reducer中 –> 更新model层中state –> 触发view层的render方法进行重新渲染 –> 页面更新

 如图所示
 ![数据流](https://img-blog.csdn.net/2018090600192160?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NDVV9DaW5keQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
 ---
 ## 参考资源
 
- [umi + dva，完成用户管理的 CURD 应用](https://github.com/sorrycc/blog/issues/62)
- [Dva入门课](https://dvajs.com/guide/introduce-class.html)
- [Umi官方指南](https://umijs.org/)
- [React官方指南](https://reactjs.org/)
- [Dva官方指南](https://dvajs.com/)
- [Ant Design](https://ant.design/)
- [Dva + Ant Design 前后端分离之 React 应用实践](https://www.jianshu.com/p/1329a324101d)
- [Dva Github](https://github.com/dvajs/dva/blob/master/README_zh-CN.md)
