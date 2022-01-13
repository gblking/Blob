---
layout: post
title: Class语法创建Vue组件(二)
categories: Vue
tags: [Vue]
excerpt: 使用Class语法创建Vue组件、vue-property-decorator
---

## 文档

[TypeScript](https://www.tslang.cn/docs/home.html)  
[Vue CLI](https://cli.vuejs.org/zh/guide/)  
[vue_class_component](https://class-component.vuejs.org/)  
[vue_property_decorator](https://github.com/kaorun343/vue-property-decorator)  
[vuex-class](https://github.com/ktsn/vuex-class/)

## 概述

此库完全依赖于`vue-class-component`，可阅读其自述文档或者[Class 语法创建 Vue 组件(一)]({{ site.url }}{{site.baseurl}}/vue/2022/01/12/vue-class-component.html)

## 安装

```sh
npm install -S vue-property-decorator
```

## API

### @Prop

**`@Prop(options: (PropOptions | Constructor[] | Constructor) = {}) decorator(装饰器)`**

```Vue
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}
```

相当于

```Vue
export default {
  props: {
    propA: {
      type: Number,
    },
    propB: {
      default: "default value",
    },
    propC: {
      type: [String, Boolean],
    },
  },
};
```

如果想要获取类型定义来设置每个 prop 值的`type`属性，您可以使用[reflect-metadata](https://github.com/rbuckton/reflect-metadata)

1. 将 `emitDecoratorMetadata` 设置为 `true`.
2. 在引入`vue-property-decorator` 之前 引入 `reflect-metadata` (只需引入一次 `reflect-metadata`)

```Vue
import 'reflect-metadata'
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class MyComponent extends Vue {
  @Prop() age!: number
}
```

每个 prop 的默认值都要定义为与上面示例代码相同  
不再 支持像`@Prop() prop = 'default value'` 这样定义每个默认属性

### @PropSync

**`@PropSync(propName: string, options: (PropOptions | Constructor[] | Constructor) = {}) decorator`**

```Vue
import { Vue, Component, PropSync } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @PropSync('name', { type: String }) syncedName!: string
}
```

相当于

```Vue
export default {
  props: {
    name: {
      type: String,
    },
  },
  computed: {
    syncedName: {
      get() {
        return this.name;
      },
      set(value) {
        this.$emit("update:name", value);
      },
    },
  },
};
```

它就像除了将 prop 名称作为修饰器参数外的 @Prop, 还在后面创建了一个计算属性的 getter 和 setter. 这样你可以将它作为常规数据属性来与其进行交互, 同时也使交互和在父组件上添加`.sync`一样容易.

### @Model

**`@Model(event?: string, options: (PropOptions | Constructor[] | Constructor) = {}) decorator`**

```Vue
import { Vue, Component, Model } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @Model('change', { type: Boolean }) readonly checked!: boolean
}
```

相当于

```Vue
export default {
  model: {
    prop: "checked",
    event: "change",
  },
  props: {
    checked: {
      type: Boolean,
    },
  },
};
```

vue2.2 中新增实例 model 选项。

### @ModelSync

**`@ModelSync(propName: string, event?: string, options: (PropOptions | Constructor[] | Constructor) = {}) decorator`**

```Vue
import { Vue, Component, ModelSync } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @ModelSync('checked', 'change', { type: Boolean })
  readonly checkedValue!: boolean
}
```

相当于

```Vue
export default {
  model: {
    prop: "checked",
    event: "change",
  },
  props: {
    checked: {
      type: Boolean,
    },
  },
  computed: {
    checkedValue: {
      get() {
        return this.checked;
      },
      set(value) {
        this.$emit("change", value);
      },
    },
  },
};
```

### @Watch

**`@Watch(path: string, options: WatchOptions = {}) decorator`**

```Vue
import { Vue, Component, Watch } from "vue-property-decorator";

@Component
export default class YourComponent extends Vue {
  @Watch("child")
  onChildChanged(val: string, oldVal: string) {}

  @Watch("person", { immediate: true, deep: true })
  onPersonChanged1(val: Person, oldVal: Person) {}

  @Watch("person")
  onPersonChanged2(val: Person, oldVal: Person) {}

  @Watch("person")
  @Watch("child")
  onPersonAndChildChanged() {}
}
```

相当于

```Vue
export default {
  watch: {
    child: [
      {
        handler: "onChildChanged",
        immediate: false,
        deep: false,
      },
      {
        handler: "onPersonAndChildChanged",
        immediate: false,
        deep: false,
      },
    ],
    person: [
      {
        handler: "onPersonChanged1",
        immediate: true,
        deep: true,
      },
      {
        handler: "onPersonChanged2",
        immediate: false,
        deep: false,
      },
      {
        handler: "onPersonAndChildChanged",
        immediate: false,
        deep: false,
      },
    ],
  },
  methods: {
    onChildChanged(val, oldVal) {},
    onPersonChanged1(val, oldVal) {},
    onPersonChanged2(val, oldVal) {},
    onPersonAndChildChanged() {},
  },
};
```

### @Provide / @Inject

**`@Provide(key?: string | symbol) / @Inject(options?: { from?: InjectKey, default?: any } | InjectKey) decorator`**

```Vue
import { Component, Inject, Provide, Vue } from 'vue-property-decorator'

const symbol = Symbol('baz')

@Component
export class MyComponent extends Vue {
  @Inject() readonly foo!: string
  @Inject('bar') readonly bar!: string
  @Inject({ from: 'optional', default: 'default' }) readonly optional!: string
  @Inject(symbol) readonly baz!: string

  @Provide() foo = 'foo'
  @Provide('bar') baz = 'bar'
}
```

相当于

```Vue
const symbol = Symbol('baz')

export const MyComponent = Vue.extend({
  inject: {
    foo: 'foo',
    bar: 'bar',
    optional: { from: 'optional', default: 'default' },
    baz: symbol,
  },
  data() {
    return {
      foo: 'foo',
      baz: 'bar',
    }
  },
  provide() {
    return {
      foo: this.foo,
      bar: this.baz,
    }
  },
})
```

### @ProvideReactive / @InjectReactive

**`@ProvideReactive(key?: string | symbol) / @InjectReactive(options?: { from?: InjectKey, default?: any } | InjectKey) decorator`**  
这两个修饰器是`@Provide` 和 `@Inject` 的响应式版本. 如果一个被提供的值被父组件修改, 子组件可以监听到这个修改.

```Vue
const key = Symbol()
@Component
class ParentComponent extends Vue {
  @ProvideReactive() one = 'value'
  @ProvideReactive(key) two = 'value'
}

@Component
class ChildComponent extends Vue {
  @InjectReactive() one!: string
  @InjectReactive(key) two!: string
}
```

### @Emit

**`@Emit(event?: string) decorator`**
`@Emit` `$emit` 装饰的函数, 它们的返回值后面跟着它们的原始参数. 如果返回值是一个 Promise 对象, 则会在触发前达到完成状态.

如果事件名称不提供 `event` 参数, 函数名将会被代替使用. 在这种情况下, 驼峰命名将被转换成短横线隔开式命名.

```Vue
import { Vue, Component, Emit } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  count = 0

  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  onInputChange(e) {
    return e.target.value
  }

  @Emit()
  promise() {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```

相当于

```Vue
export default {
  data() {
    return {
      count: 0,
    }
  },
  methods: {
    addToCount(n) {
      this.count += n
      this.$emit('add-to-count', n)
    },
    resetCount() {
      this.count = 0
      this.$emit('reset')
    },
    returnValue() {
      this.$emit('return-value', 10)
    },
    onInputChange(e) {
      this.$emit('on-input-change', e.target.value, e)
    },
    promise() {
      const promise = new Promise((resolve) => {
        setTimeout(() => {
          resolve(20)
        }, 0)
      })

      promise.then((value) => {
        this.$emit('promise', value)
      })
    },
  },
}
```

### @Ref

**`@Ref(refKey?: string) decorator`**

```Vue
<template>
    <button ref="button"></button>
    <AnotherComponent />
</template>

import { Vue, Component, Ref } from 'vue-property-decorator'

import AnotherComponent from '@/path/to/another-component.vue'

@Component
export default class YourComponent extends Vue {
  @Ref() readonly anotherComponent!: AnotherComponent
  @Ref('aButton') readonly button!: HTMLButtonElement
}
```

相当于

```Vue
export default {
  computed() {
    anotherComponent: {
      cache: false,
      get() {
        return this.$refs.anotherComponent as AnotherComponent
      }
    },
    button: {
      cache: false,
      get() {
        return this.$refs.aButton as HTMLButtonElement
      }
    }
  }
}
```

### VModel

**`@VModel(propsArgs?: PropOptions) decorator`**

```Vue
import { Vue, Component, VModel } from 'vue-property-decorator'

@Component
export default class YourComponent extends Vue {
  @VModel({ type: String }) name!: string
}
```

相当于

```Vue

export default {
  props: {
    value: {
      type: String,
    },
  },
  computed: {
    name: {
      get() {
        return this.value
      },
      set(value) {
        this.$emit('input', value)
      },
    },
  },
}
```

### @Component

由 `vue-class-component` 提供

### Mixins

名为 mixins 的辅助函数, 由 `vue-class-component` 提供
