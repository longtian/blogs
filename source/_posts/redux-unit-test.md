---
title: Redux 如何单元测试
date: 2016-05-04 12:00:00
tags:
---

[源码](https://github.com/longtian/redux-unit-test-example)

redux 里有一些好玩的东西，如果与 React 一起学习各种环境搭起来速度比较慢，也比较麻烦。如果使用单元测试来学习，
并且阅读源代码，可以更快地搞懂。

## TOP Level
> Redux 顶层的 API 允许你创建 Store, 合并 Reducer，并通过 storeEnhancer 来引入中间件进行扩展。

- createStore(reducer, [initialState], [enhancer])
- combineReducers(reducers)
- applyMiddleware(...middlewares)
- bindActionCreators(actionCreators, dispatch)
- compose(...functions)

## Store API
> Store 创建后有这些方法可供调用

- dispatch
- subscribe
- getState

## Redux 常用插件

### redux-actions
> 简化 Action 的创建，使其符合 FSA 标准

- createAction(type, payloadCreator, metaCreator)
- handleAction(type, reducer | reducerMap)
- handleActions(reducerMap, defaultState)

### redux-promise
> 使用 Promise 作为 Action 的 payload，以此减少不必要的代码

## redux-thunk
> 使用 Funtion 作为 Action， 从而延迟执行
