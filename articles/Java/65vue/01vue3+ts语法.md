# 搭建环境

## 环境

1. 安装npm

2. 安装vite和项目模板

    ```
    npm init vite@latest
    选择vue
    选择typescript
    ```

3. 用IDE打开项目，打开终端，初始化包

    ```
    npm install
    npm install less less-loader
    ```

4. 启动。打开package.json，里面有各环境下的启动命令

    ```
    # 启动（相当于npm run dev）
    vite
    ```

## 项目结构

```
public 放静态资源，但不会被vite所编译
src
	assets		放静态资源
	components 	自定义公共组件
	App.vue 	vue全局入口文件
	main.ts 	全局配置文件
	style.css 	可以删除
	vite-env.d.ts 声明文件扩充：选了ts,ts不认识vue后缀，此文件定义了文件扩充
index.html 项目首页，vue的App.vue就挂在这里
package.json 依赖包版本，以及打包、运行命令

```

## 单文件结构

```
<script setup lang="ts">
// 可以有多个，但是setup属性的只能有一个
// setup lang="ts" 是vue的语法糖，方便开发者直接将代码写在这里面，而不是先写
// export default {}
</script>

<template>
// 只能有一个
</template>

<style scoped>
// 可以有多个
</style>
```



# vue

## 回顾vue语法

```
<template>
  <div>
    <button @click="btnClick"  :style="myStyle">button</button>
  </div>
  <input v-model="a" type="text">
  <div>{{ a }}</div>
  <div :key="index" v-for="(item,index) in arr">{{ item }}</div>
</template>

<script setup lang="ts">
import {ref} from "vue";
// v-on:click 或者 @click 绑定事件
const btnClick = () => {
  console.log("点击事件");
}
// v-bind:style 绑定属性
const myStyle = {
  color: 'red'
}
// v-model用于双向绑定表单内容
// 用ref reactive才能双向绑定数据
const a = ref('请输入内容');
const arr: string[] = ['haha', 'hehe', 'lala']
</script>
```

## 虚拟DOM

问题：用JS直接操作DOM比较耗时，因为DOM属性很多。

解决：用JS的计算性能换取操作DOM的性能。通过JS来生成一个AST节点树，这个节点树就是虚拟DOM。

## diff算法

v-for列表时，如果新增/删除了其中一个，加key可以更快的操作。此时vue作者用了一个diff算法。感兴趣自己去搜。

## ref,reactive

* ref支持所有类型；reactive只支持object类型，不支持string等类型
* ref和reactive都支持泛型定义，也支持自动类型推断
* ref取值赋值都需要加.value， reactive不需要
* reactive是Proxy对象，类似于指针，直接赋值可能会破坏响应式，解决：数组用push方法；把数组当作对象的一个属性。

ref代码

```
import {ref} from "vue";
type People = {
  name: string
}
const one = ref<People>({name: 'cat'})
const two = ref('hello')
const changeB = () => {
  one.value.name = 'xxxx'
}
```

## toRef,toRefs,toRaw

* toRef只能修改响应式对象，非响应式不能。常用于提取响应式对象中的一部分。
* toRef只能解构一个属性，toRefs能解构多个属性。
* toRaw将响应式变成非响应式

```
import {ref,toRef} from "vue";
const one = ref({name: 'cat', age: 18})
const age = toRef(one.value, 'age');
const changeB = () => {
  age.value = 20
}
```

## watch侦听器

侦听ref或reactive中的值的变化

* 可以侦听多个（用中括号括住）
* 如果侦听ref对象，需要打开深度侦听
* 如果侦听reactive对象，默认已开启深度侦听

```
let message = ref({name:'tom',age:18})
// 打开深度侦听
watch(message, (newVal, oldVal) => {
	console.log(newVal)
},{deep: true})
// 侦听对象中的单个属性，需要用个函数返回属性
watch(()=>message.value.name, (newVal, oldVal)=>{
	console.log(newVal)
})
```

## watchEffect高级侦听器

watchEffect函数中用到的变量都会被侦听，没有用到的不会

```
watchEffect(() => {
  console.log("message1", one)
  console.log("message2", age)
})
```

## 生命周期

这里声明周期是指每个组件的声明周期，包括一个DIV元素，所以每一个DIV元素改变时都会触发。

```
// beforeCreate created setup语法糖模式是没有这两个生命周期的 setup去替代了
console.log('setup')
// 组件创建
onBeforeMount(() => {
  console.log("1")
});
onMounted(() => {
  console.log("2")
});
// 组件更新
onBeforeUpdate(() => {
  console.log("3")
});
onUpdated(() => {
  console.log("4")
});
// 组件销毁
onBeforeUnmount(() => {
  console.log("5")
});
onUnmounted(() => {
  console.log("6")
});
```

## 父子组件传参

父传子

```
子组件定义
    const props = defineProps({
      title: {
        type: String,
        default: "默认值"
      }
    })
 
父组件
	// 使用子组件
	<Menu :title="title"></Menu>
	// 值
	const title = '菜单1'
```

子传父

```
子组件定义
    const emit = defineEmits(['on-click']);
    const send = () => {
      emit('on-click', '消息xxxx')
    }
父组件
	// 使用子组件
	<Menu @on-click="getName"></Menu>
	// 定义方法获取消息
	const getName = (name:string) => {
  		console.log(name)
	}
```

子传父2，子组件暴露属性或方法

```
子组件定义
	const msg = "hello father";
    defineExpose({
      msg
    })
父组件
	// 使用子组件
	<Menu ref="I"></Menu>
	// 使用子组件暴露的方法
	const I = ref()
    onMounted(()=>{
      console.log(I.value?.msg)
    })
```

## 局部组件、全局组件、动态组件

局部组件需要这样引入和使用

```
import Content from './views/layout/Content.vue'
```

全局组件

```
// 1. 在main.ts里定义
import App from './App.vue'
import ContentVue from './views/layout/Content.vue'
export const app = createApp(App)
app.component('Content', ContentVue)
app.mount('#app')
// 2. 其他组件中就可以直接使用
```

动态组件：几个组件公用一个挂载点（Tab页切换）

```
// 通过component
<button @click="dynamic">Vue Tab Button</button>
<component :is="comId"></component>

// shallowRef浏览器console不会有警告
const comId = shallowRef(AVue)
const dynamic = () => {
  if (comId.value == BVue) {
    comId.value = markRaw(AVue)
  } else {
    comId.value = markRaw(BVue)
  }
}
```

## 插槽

匿名插槽

```
// 子组建
<template>
  <div>
    Test
    <slot></slot>
  </div>
</template>

// 父组件 
<TestVue>
  <template v-slot>
    <div>匿名插槽插入内容</div>
  </template>
</TestVue>
```

具名插槽

```
// 子组建
<template>
  <div>
    <slot name="top"></slot>
    Test
    <slot name="bottom"></slot>
  </div>
</template>

// 父组件， v-solt:可以简写成#
<TestVue>
    <template v-slot:top>
        <div>匿名插槽插入内容</div>
    </template>
    <template v-slot:bottom>
        <div>哈哈哈</div>
    </template>
</TestVue>
```

## Teleport传递组件

Teleport是Vue3内置方法，可以直接用。

传递组件：如果父级用了position:relative，那么子组件位置可能改变，为了使子组件位置在主窗口位置不变，使用Teleport。

```
// to表示传送到的元素（id class 元素等都可以）
<Teleport to="body">
	<TestVue><TestVue>
</Teleport>
```

## keep-alive缓存组件

如果有两个组件，只展示其中一个。另一个可以缓存起来。

```
<keep-alive>
	<AVue v-if="flag"></AVue>
	<BVue v-else></BVue>
</keep-alive>

// keep-alive还有两个声明周期函数，keep-alive每次切换组件时用。
onActiveed()
onDeactiveed()
```

## transition动画组件

transition：vue提供的实现动画效果的组件。

animate.css: 封装好的动画效果的css文件。

先安装animate.css

```
npm install animate.css
```

使用

```
<template>
  <div>
    <button @click="flag = !flag">showContent</button>
  </div>
  <transition :duration="500" enter-active-class="animate__animated animate__fadeIn" leave-active-class="animate__animated animate__fadeOut">
    <div v-if="flag" class="content"></div>
  </transition>
</template>

<script setup lang="ts">
import 'animate.css'
import {ref} from "vue";
const flag = ref(true)
</script>
```

注意：

```
* animate.css如果是4以上版本，class都需要加animate__animated
* transition也有生命周期，这里不讲
* transition还可以设置初始动画效果，即apper apper-from-class apper-active-class apper-to-class
```

## 依赖注入Provide / Inject

父向子传递值可以用props，但是如果是深层嵌套子组件，一直用props比较麻烦。

Provide可以在父组件定义好数据或方法，任何后代都可以使用Inject来接受。

```
// 父组件
const test = ref('haha')
// provide('msg', test)
provide('msg', readonly(test))

// 子组件（子组件修改，父组件也修改）
const msg = inject<Ref<string>>('msg')
```

## Mitt

vue2兄弟组件传参，使用的是Bus，相当于写一个发布订阅模式。

vue3中的Bus就是Mitt库。

安装依赖包

```
npm install mitt
```

创建Bus.ts

```
import mitt from 'mitt';
export default mitt();
```

使用

```
// 组件A
import mitt from '../../Bus'
const emitter = () => {
  mitt.emit('helloEvent', {msg: 'hello'})
};

// 组件B
import mitt from '../../Bus'
mitt.on('helloEvent', (p) => {
  console.log('接受参数：', p)
})
```

## 自定义Hook

hook主要用来处理复用代码逻辑的封装，在vue2中是Mixins。（Mixins有一些问题）

hook官方封装了很多方法，可以直接使用。本节主要讲解自定义hook

新建hooks目录，新建ts文件

```
const addFun = (num1:number, num2:number) => {
    console.log(num1 + num2)
}
export default addFun
```

在组件中用

```
import addFun from '../../hooks/index'
addFun(1, 2)
```



## 全局函数、全局变量

```
// 在main.ts中直接定义 （方法和变量一样）
app.config.globalProperties.$hello = 'Hello World'

// 在组件中使用
{{ $hello }}
```



## UI库

web端：

ElementPlus：setup语法糖模式

Ant Design：setup函数模式

IView:  options api模式

移动端：

Vant: setup函数模式

## 样式穿透

```
<el-input class="test"></el-input>
.test {
 background: red;
}

这样写样式不生效，因为scoped把样式放在了代码最后。应该用deep
.test{
    :deep(input){
     background:red;
    }
}
```











