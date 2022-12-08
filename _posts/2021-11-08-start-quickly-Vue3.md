---
layout: post
title: 一文快速上手Vue3
subtitle:
categories: Vue3
tags: [Vue3]
excerpt: 对Vue3的概括、笔记
---

### Vue3 的改变

**1. 性能提升**

- 打包大小减少 41%
- 初次渲染快 55%，更新渲染快 133%
- 内存减少 54%  
  ......

**2. 源码升级**

- 使用 Proxy 代替 defineProperty 实现响应式
- 重写虚拟 DOM 的实现和 Tree-Shaking  
  ......

**3. 拥抱 TypeScript**

- Vue3 可以更好的支持 TypeScript

**4. 新特性**

1. Composition API (组合 API)

   - setup 配置
   - ref 与 reactive
   - watch 与 inject  
     ......

2. 新的内置组件

   - Fragment
   - Teleport
   - Suspense

3. 其他改变
   - 新的生命周期钩子
   - data 选项应始终被声明为一个函数
   - 移除 keyCode 支持作为 v-on 的修饰符  
     ......

### 创建 Vue3.0 工程

1.  使用 vue-cli 创建

    ```js
    // 查看@vue/cli版本，确保@vue/cli版本在4.5.0以上
    vue --version
    // 安装或者升级你的@vue/cli
    npm install -g @vue/cli
    // 创建
    vue create vue_test
    // 启动
    cd vue_test
    npm run serve
    ```

2.  使用 vite 创建  
     [官方文档](https://v3.cn.vuejs.org/guide/installation.html#vite)  
     [vite 官网](https://vitejs.cn)

    vite - 新一代前端构建工具

    优势如下：

    - 开发环境中，无需打包操作，可快速的冷启动
    - 轻量快速的热重载(HMR)
    - 真正的按需编译，不再等待整个应用编译完成

    传统构建与 vite 构建对比图：  
     传统构建  
     ![传统构建]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/01.webp)  
     vite 构建  
     ![vite构建]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/02.webp)

    ```js
    // 创建工程
    npm init vite-app <project-name>
    // 进入工程目录
    cd <project-name>
    // 安装依赖
    npm install
    // 运行
    npm run dev
    ```

    ### 常用 Composition API

    #### setup

    1. Vue3.0 中一个新的配置项，值为一个函数
    2. setup 是所有 Composition API（组合 API） “ 表演的舞台 ”
    3. 组件中所用到的：数据、方法等等，均要配置在 setup 中
    4. setup 函数的两种返回值：

       - 若返回一个对象，则对象中的属性、方法, 在模板中均可以直接使用。（重点关注！）
       - 若返回一个渲染函数：则可以自定义渲染内容。（了解）

    5. 注意点：
       - 尽量不要与 Vue2.x 配置混用
         - Vue2.x 配置（data、methos、computed...）中可以访问到 setup 中的属性、方法。
         - 但在 setup 中不能访问到 Vue2.x 配置（data、methos、computed...）。
         - 如果有重名, setup 优先。
       - setup 不能是一个 async 函数，因为返回值不再是 return 的对象, 而是 promise, 模板看不到 return 对象中的属性。（后期也可以返回一个 Promise 实例，但需要 Suspense 和异步组件的配合）

    #### ref 函数

    1.  作用: 定义一个响应式的数据
    2.  语法: `const xxx = ref(initValue)`
        - 创建一个包含响应式数据的**引用对象（reference 对象，简称 ref 对象）**
        - JS 中操作数据： `xxx.value`
        - 模板中读取数据: 不需要.value，直接：`<div>{{xxx}}</div>`
    3.  备注：
        - 接收的数据可以是：基本类型、也可以是对象类型。
        - 基本类型的数据：响应式依然是靠 `Object.defineProperty()`的 `get` 与 `set` 完成的。
        - 对象类型的数据：内部“ 求助 ” 了 Vue3.0 中的一个新函数—— `reactive` 函数。

    #### reactive 函数

    - 作用: 定义一个**对象类型**的响应式数据（基本类型不要用它，要用 `ref` 函数）
    - 语法：`const 代理对象= reactive(源对象)`接收一个对象（或数组），返回一个**代理对象**（Proxy 的实例对象，简称 proxy 对象）
    - reactive 定义的响应式数据是“深层次的”。
    - 内部基于 ES6 的 Proxy 实现，通过代理对象操作源对象内部数据进行操作。

    #### Vue3.0 中的响应式原理

    - 实现原理:

      - 通过 Proxy（代理）: 拦截对象中任意属性的变化, 包括：属性值的读写、属性的添加、属性的删除等。
      - 通过 Reflect（反射）: 对源对象的属性进行操作。
      - MDN 文档中描述的 Proxy 与 Reflect：

        - Proxy：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy

        - Reflect：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect

      ```js
      new Proxy(data, {
        // 拦截读取属性值
        get(target, prop) {
          return Reflect.get(target, prop);
        },
        // 拦截设置属性值或添加新属性
        set(target, prop, value) {
          return Reflect.set(target, prop, value);
        },
        // 拦截删除属性
        deleteProperty(target, prop) {
          return Reflect.deleteProperty(target, prop);
        },
      });

      proxy.name = "tom";
      ```

      #### reactive 对比 ref

      1.  从定义数据角度对比：
          - ref 用来定义：**基本类型数据**
          - reactive 用来定义：**对象（或数组）类型数据**
          - ref 也可以用来定义**对象（或数组）类型数据**, 它内部会自动通过 `reactive` 转为**代理对象**
      2.  从原理角度对比：
          - ref 通过 `Object.defineProperty()`的 `get` 与 `set` 来实现响应式（数据劫持）
          - reactive 通过使用 **Proxy** 来实现响应式（数据劫持）, 并通过 **Reflect** 操作**源对象**内部的数据
      3.  从使用角度对比：
          - ref 定义的数据：操作数据需要`.value`，读取数据时模板中直接读取**不需要**`.value`
          - reactive 定义的数据：操作数据与读取数据：**均不需要**`.value`

      #### setup 注意点

      1. 执行时机

         - 在 beforeCreate 之前执行一次，this 是 undefined

      2. setup 参数
         - props：值为对象，包含：组件外部传递过来，且组件内部声明接收了的属性。
         - context：上下文对象
           - attrs: 值为对象，包含：组件外部传递过来，但没有在 props 配置中声明的属性, 相当于 `this.$attrs`
           - slots: 收到的插槽内容, 相当于 `this.$slots`
           - emit: 分发自定义事件的函数, 相当于 `this.$emit`

      #### computed 函数

      - 与 vue2.x 中的 computed 配置功能一致
      - 写法

      ```js
      import {computed} from 'vue'

      setup(){
         ...
      //计算属性——简写
         let fullName = computed(()=>{
            return person.firstName + '-' + person.lastName
         })
         //计算属性——完整
         let fullName = computed({
            get(){
                  return person.firstName + '-' + person.lastName
            },
            set(value){
                  const nameArr = value.split('-')
                  person.firstName = nameArr[0]
                  person.lastName = nameArr[1]
            }
         })
      }
      ```

      #### watch 函数

      - 与 Vue2.x 中 watch 配置功能一致
      - 两个"坑":
        - 监视 reactive 定义的响应式数据时：oldValue 无法正确获取、强制开启了深度监视（deep 配置失效）
        - 监视 reactive 定义的响应式数据中某个属性时：deep 配置有效

      ```js
      //情况一：监视ref定义的响应式数据
      watch(
        sum,
        (newValue, oldValue) => {
          console.log("sum变化了", newValue, oldValue);
        },
        { immediate: true }
      );

      //情况二：监视多个ref定义的响应式数据
      watch([sum, msg], (newValue, oldValue) => {
        console.log("sum或msg变化了", newValue, oldValue);
      });

      /* 情况三：监视reactive定义的响应式数据
               若watch监视的是reactive定义的响应式数据，则无法正确获得oldValue！！
               若watch监视的是reactive定义的响应式数据，则强制开启了深度监视 
      */
      watch(
        person,
        (newValue, oldValue) => {
          console.log("person变化了", newValue, oldValue);
        },
        { immediate: true, deep: false }
      ); //此处的deep配置不再奏效

      //情况四：监视reactive定义的响应式数据中的某个属性
      watch(
        () => person.job,
        (newValue, oldValue) => {
          console.log("person的job变化了", newValue, oldValue);
        },
        { immediate: true, deep: true }
      );

      //情况五：监视reactive定义的响应式数据中的某些属性
      watch(
        [() => person.job, () => person.name],
        (newValue, oldValue) => {
          console.log("person的job变化了", newValue, oldValue);
        },
        { immediate: true, deep: true }
      );

      //特殊情况
      watch(
        () => person.job,
        (newValue, oldValue) => {
          console.log("person的job变化了", newValue, oldValue);
        },
        { deep: true }
      ); //此处由于监视的是reactive素定义的对象中的某个属性，所以deep配置有效
      ```

      #### watchEffect 函数

      - watch 的套路是：既要指明监视的属性，也要指明监视的回调

      - watchEffect 的套路是：不用指明监视哪个属性，监视的回调中用到哪个属性，那就监视哪个属性

      - watchEffect 有点像 computed：

        - 但 computed 注重的计算出来的值（回调函数的返回值），所以必须要写返回值
        - 而 watchEffect 更注重的是过程（回调函数的函数体），所以不用写返回值

      ```js
      //watchEffect所指定的回调中用到的数据只要发生变化，则直接重新执行回调。
      watchEffect(() => {
        const x1 = sum.value;
        const x2 = person.age;
        console.log("watchEffect配置的回调执行了");
      });
      ```

      #### toRef

      - 作用：创建一个 ref 对象，其 value 值指向另一个对象中的某个属性
      - 语法：const name = toRef(person,'name')
      - 应用: 要将响应式对象中的某个属性单独提供给外部使用时
      - 扩展：toRefs 与 toRef 功能一致，但可以批量创建多个 ref 对象，语法：toRefs(person)

      #### toRaw 与 markRaw

      1. toRaw

         - 作用： 将一个由`reactive`生成的响应式对象转为普通对象
         - 使用场景： 用于读取响应式对象对应的普通对象，对这个普通对象的所以操作，不会引起页面更新

      2. narkRaw
         - 作用： 标记一个对象，使其永远不会再成为响应式对象
         - 使用场景：
           - 有些值不应被设置为响应式的，例如复杂的第三方类库等
           - 当渲染具有不可变数据源的大列表时，跳过响应式转换可提高性能

      #### provide 与 inject

      - 作用： 实现祖与后代组件间的通信
      - 使用： 父组件使用`provide`提供数据，后代组件使用`inject`来接收使用这些数据

      ```js
      // 祖组件中
      setup(){
         ......
         let car = reactive({name:'奔驰',price:'40万'})
         provide('car',car)
         ......
      }

      // 后代组件中
      setup(props,context){
         ......
         const car = inject('car')
         return {car}
         ......
      }
      ```

      #### 响应式数据的判断

      - isRef: 检查一个值是否为一个 ref 对象
      - isReactive: 检查一个对象是否是由 reactive 创建的响应式代理
      - isReadonly: 检查一个对象是否是由 readonly 创建的只读代理
      - isProxy: 检查一个对象是否是由 reactive 或者 readonly 方法创建的代理

      ### Composition API 的优势

      1. Options API 存在的问题  
         使用传统 OptionsAPI 中，新增或者修改一个需求，就需要分别在 data，methods，computed 里修改。
         ![03]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/03.webp)  
         ![04]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/04.webp)

      2. Composition API 的优势  
         我们可以更加优雅的组织我们的代码，函数。让相关功能的代码更加有序的组织在一起。  
         ![05]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/05.webp)  
         ![06]({{site.url}}{{site.baseurl}}/assets/images/post/20211108/06.webp)

      ### 新组件

      #### Fragment

      - 在 Vue2 中: 组件必须有一个根标签
      - 在 Vue3 中: 组件可以没有根标签, 内部会将多个标签包含在一个 Fragment 虚拟元素中
      - 好处: 减少标签层级, 减小内存占用

      #### Teleport

      Teleport 是一种能够将我们的模板移动到 DOM 中 Vue app 之外的其他位置的技术, 类似于 React 的 Portal

      ```html
      <div id="teleport-target"></div>

      .....

      <!-- to 属性就是目标位置 -->
      <teleport to="#teleport-target">
        <div v-if="visible" class="toast-wrap">
          <div class="toast-msg">我是一个 Toast 文案</div>
        </div>
      </teleport>
      ```

      #### Suspense

      - 等待异步组件时渲染一些额外的内容，让应用拥有更好的用户体验
      - 使用步骤：

        - 异步引入组件

        ```js
        import { defineAsyncComponent } from "vue";
        const Child = defineAsyncComponent(() =>
          import("./components/Child.vue")
        );
        ```

        - 使用`Suspense`包裹组件，并配置好`default` 与 `fallback`

        ```html
        <template>
          <div class="app">
            <h3>我是App组件</h3>
            <Suspense>
              <template v-slot:default>
                <Child />
              </template>
              <template v-slot:fallback>
                <h3>加载中.....</h3>
              </template>
            </Suspense>
          </div>
        </template>
        ```

      ### 全局 API 的转移

      将全局的 API，Vue.xxx 调整到应用实例 app 上

      | 2.x 全局 API (Vue)       | 3.x 实例 API (app)          |
      | ------------------------ | --------------------------- |
      | Vue.config.xxx           | app.config.xxx              |
      | Vue.config.productionTip | 移除                        |
      | Vue.component            | app.component               |
      | Vue.directive            | app.directive               |
      | Vue.mixin                | app.mixin                   |
      | Vue.use                  | app.use                     |
      | Vue.prototype            | app.config.globalProperties |

      ### 其他改变

      1.  移除 keyCode 作为 v-on 的修饰符，同时也不再支持 config.keyCodes
      2.  移除 v-on.natice 修饰符
      3.  移除过滤器 filter
      4.  v-model 的本质变化
          - prop: vlue -> modelValue
          - event: input -> update:modelValue
      5.  .sync 修饰符已移除，由 v-model 代替
      6.  v-if 优先 v-for 解析
