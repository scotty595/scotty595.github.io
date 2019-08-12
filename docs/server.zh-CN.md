---
order: 7
title: 和服务端进行交互
type: 开发
---

Ant Design Pro 是一套基于 React 技术栈的单页面应用，我们提供的是前端代码和本地模拟数据的开发模式，通过 API 的形式和任何技术栈的服务端应用一起工作。下面将简单介绍和服务端交互的基本写法。

## 前端请求流程

在 Ant Design Pro 中，一个完整的前端 UI 交互到服务端处理流程是这样的：

1. UI 组件交互操作；
2. 调用 model 的 effect；
3. 调用统一管理的 service 请求函数；
4. 使用封装的 request.ts 发送请求；
5. 获取服务端返回；
6. 然后调用 reducer 改变 state；
7. 更新 model。

从上面的流程可以看出，为了方便管理维护，统一的请求处理都放在 `services` 文件夹中，并且一般按照 model 维度进行拆分文件，如：

```
services/
  user.ts
  api.ts
  ...
```

其中，`utils/request.ts` 是基于 [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) 的封装，便于统一处理 POST，GET 等请求参数，请求头，以及错误提示信息等。具体可以参看 [request.ts](https://github.com/ant-design/ant-design-pro/blob/master/src/utils/request.ts)。

例如在 services 中的一个请求用户信息的例子：

```ts
// services/user.ts
import request from '../utils/request';

export async function query() {
  return request('/api/users');
}

export async function queryCurrent() {
  return request('/api/currentUser');
}

// models/user.ts
import { queryCurrent } from '../services/user';
...
effects: {
  *fetch({ payload }, { call, put }) {
    ...
    const response = yield call(queryCurrent);
    ...
  },
}
```

### 处理异步请求

在处理复杂的异步请求的时候，很容易让逻辑混乱，陷入嵌套陷阱，所以 Ant Design Pro 的底层基础框架 [dva](https://github.com/dvajs/dva) 使用 `effect` 的方式来管理同步化异步请求：

```js
effects: {
  *fetch({ payload }, { call, put }) {
    yield put({
      type: 'changeLoading',
      payload: true,
    });
    // 异步请求 1
    const response = yield call(queryFakeList, payload);
    yield put({
      type: 'save',
      payload: response,
    });
    // 异步请求 2
    const response2 = yield call(queryFakeList2, payload);
    yield put({
      type: 'save2',
      payload: response2,
    });
    yield put({
      type: 'changeLoading',
      payload: false,
    });
  },
},
```

通过 [generator](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Statements/function*) 和 [yield](https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/Operators/yield) 使得异步调用的逻辑处理跟同步一样，更多可参看 [dva async logic](https://github.com/dvajs/dva/blob/master/docs/GettingStarted.md#async-logic)。
