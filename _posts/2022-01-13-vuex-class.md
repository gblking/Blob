---
layout: post
title: Class语法创建Vue组件(三)
categories: Vue
tags: [Vue]
excerpt: 使用Class语法创建Vue组件、vuex-class
---

## 文档

[TypeScript](https://www.tslang.cn/docs/home.html)  
[Vue CLI](https://cli.vuejs.org/zh/guide/)  
[vue_class_component](https://class-component.vuejs.org/)  
[vue_property_decorator](https://github.com/kaorun343/vue-property-decorator)  
[vuex-class](https://github.com/ktsn/vuex-class/)

## 概述

`Vuex` 和 `vue-class-component` 的绑定助手  
[Vue](https://cn.vuejs.org/)  
[Vuex](https://vuex.vuejs.org/zh/)

## 安装

```sh
npm install --save vuex-class
# or
yarn add vuex-class
```

## 示例

```Vue
import Vue from 'vue'
import Component from 'vue-class-component'
import {
  State,
  Getter,
  Action,
  Mutation,
  namespace
} from 'vuex-class'

const someModule = namespace('path/to/module')

@Component
export class MyComp extends Vue {
  @State('foo') stateFoo
  @State(state => state.bar) stateBar
  @Getter('foo') getterFoo
  @Action('foo') actionFoo
  @Mutation('foo') mutationFoo
  @someModule.Getter('foo') moduleGetterFoo

  // 如果将参数省略，则使用属性名称
  // for each state/getter/action/mutation type
  @State foo
  @Getter bar
  @Action baz
  @Mutation qux

  created () {
    this.stateFoo // -> store.state.foo
    this.stateBar // -> store.state.bar
    this.getterFoo // -> store.getters.foo
    this.actionFoo({ value: true }) // -> store.dispatch('foo', { value: true })
    this.mutationFoo({ value: true }) // -> store.commit('foo', { value: true })
    this.moduleGetterFoo // -> store.getters['path/to/module/foo']
  }
}
```
