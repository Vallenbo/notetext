# 11：状态管理库`Pinia`

`Pinia`是`Vue`的专属状态管理库，它允许你跨组件或页面共享状态

## 11.1：安装与使用`pinia`

1.  安装语法：`npm install pinia`
2.  创建一个`pinia`（根存储）并将其传递给应用程序
main.js

```
import { createApp } from 'vue'
import App from './App.vue'

// 引入 createPinia 函数
import { createPinia } from 'pinia'

const app = createApp(App)

// 使用 createPinia() 来创建 Pinia（根存储），并应用到整个应用中
app.use(createPinia())

app.mount('#app')
```

## 11.2：`Store`

1.  `store`是一个保存状态和业务逻辑的实体，它并不与你的组件树绑定；换句话说，它承载着全局状态；它有点像一个永远存在的组件，每个组件都可以读取和写入它
2.  `store`它有三个概念，`state`、`getters`和`actions`，我们可以l理解成组件中的`data`、`computed`和`methods`
3.  在项目中的`src\store`文件夹下不同的`store.js`文件
4.  `store`是用`defineStore(_name_, _function_ | _options_)`定义的，建议其函数返回的值命名为`use...Store`方便理解

1.  参数`**name**`：名字，必填值且唯一
2.  参数`**function|options**`：可以是对象或函数形式

-   对象形式【选项模式】，其中配置`state`、`getters`和`actions`选项
-   函数形式【组合模式，类似组件组合式`API`的书写方式】，定义响应式变量和方法，并且`return`对应的变量和方法；`ref()`相当于`state`，`computed()`相当于`getters`，`function()`相当于`actions`

定义 store 【选项式】



定义 store 【组合式】




## 11.3：`State`

`state`是`store`的核心部分，主要存储的是共享的数据

### 11.3.1：定义`state`

1.  `store`采用的是选项式模式时，`state`选项为函数返回的对象，在其定义共享的数据
2.  `store`采用的是组合式模式时，在其函数内定义的`ref`变量，最终`return`出去来提供共享的数据



state【选项式】
```
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
  	// 共享的数据，为函数返回的对象形式
    state: () => ({
        age: 27,
        level: 5,
        account: 'SD77842',
        nickname: '自古风流'
    })
})
```


state【组合式】

```
import {defineStore} from "pinia";
import {ref} from "vue";

export const useUserStore = defineStore('user', () => {
    const age = ref(27)
    const level = ref(5)
    const account = ref('SD77842')
    const nickname = ref('自古风流')
    
    return { age, level, account, nickname } // 将数据暴露出去，共享给需要的组件
})
```




### 11.3.2：组件中访问`state`

1.  在选项式 API 组件中，可以使用`mapState(_storeObj_, _array_ | _object_)`帮助器将状态属性映射为只读计算属性

1.  `storeObj`引入的`store`对象
2.  `array | object`：字符串数组形式或者对象形式

-   【字符串数组形式】直接将`store`中`state`的数据映射为当前组件的计算属性，但是不能自定义名称
-   【对象形式时】`key`为自定义当前组件的计算属性名，`value`字符串形式，是`store`中`state`的共享数据

**提示：**`**mapState()**`函数映射到组件中的计算属性是只读的，如果想在组件中响应式修改`state`的数据，则应该选择`**mapWritableState()**`函数来映射计算属性

2.  在组合式 API 组件中，直接引入对应的`store`，通过`store`对象直接获取和修改`state`

**提示：**

如果想在组件中自定义变量来接收`store`中的`state`中共享的数据，我们可以这样做：

-   使用`computed(() => store._dataName_)`，具有响应式，但是只读形式
-   使用`storeToRefs(_store_)`从`store`解构想要的`state`，具有响应式，可直接修改，可自定义名称



组件中获取 state【选项式】

User.vue【选项式】

组件中获取 state【组合式】

User.vue【组合式】




## 11.4：`Getters`

`getters`是计算得到的新的共享数据，当依赖的数据发生变化时则重新计算，所以其他组件包括`store`自己不要直接对其修改




### 11.4.1：定义`Getters`

1.  `store`采用的是选项式模式时，`getters`选项中声明的函数即为计算属性

1.  在其函数内可通过`**this**`关键字来获取`store`实例，也可通过方法的第一个参数得到`store`实例
2.  如果采用的是箭头函数的话，无法使用`this`关键字，为了更方便使用`store`中实例，可为其箭头函数设置第一个参数来获取`store`实例

2.  `store`采用的是组合式模式时，可通过`**computed()**`函数通过计算得到新的数据，再将其`**return**`暴露出去即可

getters【选项式】

getters【组合式】



### 11.4.2：在组件中使用`Getters`

1.  选项式`API`的组件中，访问`store`中的`getters`和访问`state`类似，同样可使用`**mapState()**`帮助器将`**getters**`属性映射为只读计算属性

**注意：**如果采用`**mapWritableState()**`帮助器将`store`中的`getters`映射为组件内部的计算属性，依旧可以具有响应式，一旦对其进行修改则会报错

2.  在组合式`API`组件中，访问`store`中的`getters`和访问`state`类似，直接引入对应的`store`，通过`store`对象直接获取`getters`，但是如果对其进行修改则会报错

**提示：**

如果想将`store`中的`getter`中共享的数据映射为本地组件的计算属性，我们可以这样做：

-   使用`computed(() => store._getterName_)`，具有响应式，但是只读形式
-   使用`storeToRefs(_store_)`从`store`解构`getter`依旧是计算属性，所以是只读的，一旦对其进行修改则会报错，但是具有响应式，可自定义名称


组件中获取 getters【选项式】

组件中获取 getters【组合式】



## 11.5：`Actions`

`actions`一般情况下是对`state`中的数据进行修改的业务逻辑函数，`actions`也可以是异步的，您可以在其中`await`任何`API`调用甚至其他操作！


### 11.5.1：定义`Actions`

1.  `store`采用的是选项式模式时，`actions`选项中声明的函数即可共享其函数，在其函数内可通过`this`来获取整个`store`实例
2.  `store`采用的是组合式模式时，可通过声明函数，再将其`return`暴露出去即可共享其函数


actions【选项式】

actions【组合式】

### 11.5.2：组件中访问`Actions`

1.  在选项式 API 组件中，可以使用`mapActions(_storeObj_, _array_ | _object_)`帮助器将`actions`映射为当前组件的函数

1.  `storeObj`引入的`store`对象
2.  `array | object`：字符串数组形式或者对象形式

-   【字符串数组形式】直接将`store`中`actions`的函数映射为当前组件的函数，但是不能自定义名称
-   【对象形式时】`key`为自定义当前组件的函数名，`value`字符串形式，是`store`中`actions`的函数名

2.  在组合式`API`组件中，直接引入对应的`store`，通过`store`对象直接获取`actions`

**提示：**如果想将`store`中的`actions`中函数映射为本地组件的函数，可将`store`解构出对应的函数即可，也可自定应函数名，此处不可通过`storeToRefs(_store_)`函数

组件中获取 actions【选项式】

组件中获取 actions【组合式】








