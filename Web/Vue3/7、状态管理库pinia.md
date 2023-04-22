`Pinia`是`Vue`的专属状态管理库，它允许你跨组件或页面共享状态

## 11.1：安装与使用`pinia`
1.  安装语法：`npm install pinia`
2.  创建一个`pinia`（根存储）并将其传递给应用程序

main.js
```vue
import { createApp } from 'vue'
import App from './App.vue'
import { createPinia } from 'pinia' // 引入 createPinia 函数

const app = createApp(App)

app.use(createPinia()) // 使用 createPinia() 来创建 Pinia（根存储），并应用到整个应用中
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
```vue
import { defineStore } from 'pinia'

// 参数一：名字，必填值且唯一  // 参数二：选项式书写方式采用对象形式
export const useStore = defineStore('main', { // 创建 store，并暴露出去
    state: () => ({
        // ……
    }), 
    getters: {
        // ……
    },
    actions: {
        // ……
    }
})
```

定义 store 【组合式】
```vue
import { defineStore } from 'pinia'
import { computed, ref } from 'vue'


// 参数一：名字，必填值且唯一  // 参数二：组合式书写方式采用函数形式
export const useStore = defineStore('main', () => {    // 创建 store，并暴露出去
    // ref 变量  --->  state
    // computed() 计算属性  --->  getters 
    // functions 函数  --->  actions
    return { 
        // 暴露出去 变量，函数，计算属性即可
    }
})
```

## 11.3：`State`
`state`是`store`的核心部分，主要存储的是共享的数据

### 11.3.1：定义`state`
1.  `store`采用的是选项式模式时，`state`选项为函数返回的对象，在其定义共享的数据
2.  `store`采用的是组合式模式时，在其函数内定义的`ref`变量，最终`return`出去来提供共享的数据

state【选项式】
```
import { defineStore } from 'pinia'

export const useUserStore = defineStore('user', {
    state: () => ({ // 共享的数据，为函数返回的对象形式
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
	a.  `storeObj`引入的`store`对象
	b.  `array | object`：字符串数组形式或者对象形式
	- 【字符串数组形式】直接将`store`中`state`的数据映射为当前组件的计算属性，但是不能自定义名称
	- 【对象形式时】`key`为自定义当前组件的计算属性名，`value`字符串形式，是`store`中`state`的共享数据
**提示：**`**mapState()**`函数映射到组件中的计算属性是只读的，如果想在组件中响应式修改`state`的数据，则应该选择`**mapWritableState()**`函数来映射计算属性

2.  在组合式 API 组件中，直接引入对应的`store`，通过`store`对象直接获取和修改`state`
**提示：**
如果想在组件中自定义变量来接收`store`中的`state`中共享的数据，我们可以这样做：
-   使用`computed(() => store._dataName_)`，具有响应式，但是只读形式
-   使用`storeToRefs(_store_)`从`store`解构想要的`state`，具有响应式，可直接修改，可自定义名称

```vue 
<template> //组件中获取 state【选项式】
    <UserVue></UserVue>
    <h2>mapState 映射的计算属性</h2>
    <ul>
        <li>年龄：{{ age }}</li>
        <li>等级：{{ level }}</li>
        <li>账号：{{ user_account }}</li>
        <li>昵称：{{ user_nickname }}</li>
    </ul>

    <button @click="age += 10">更改年龄</button>
  |
    <button @click="user_nickname += '='">更改昵称</button>

    <hr>
    <h2>mapWritableState 映射的计算属性</h2>

    <ul>
        <li>年龄：{{ user_age }}</li>
        <li>等级：{{ user_level }}</li>
        <li>账号：{{ account }}</li>
        <li>昵称：{{ nickname }}</li>
    </ul>

    <button @click="user_age += 10">更改年龄</button>
  |
    <button @click="nickname += '='">更改昵称</button>
</template>

<script>
import {mapState, mapWritableState} from 'pinia'
import {useUserStore} from '@/store/useUserStore'
import UserVue from '@/components/User.vue'

export default {
    components: {UserVue},
    computed: {
        // mapState：将 store 的 state 映射成当前组件的计算属性
        // 具有响应式，但是是只读
        // 字符串数组形式：不能自定义计算属性名
        // 对象形式：可以自定义计算属性名
        ...mapState(useUserStore, ['age', 'level']),
        ...mapState(useUserStore, {
            user_account: 'account',
            user_nickname: 'nickname'
        }),
        // mapWritableState 与 mapState 用法类似，区别：它可以响应式的读写映射的计算属性
        ...mapWritableState(useUserStore, ['account', 'nickname']),
        ...mapWritableState(useUserStore, {
            user_age: 'age',
            user_level: 'level'
        }),
    }
}
</script>

<script setup>
import {useUserStore} from '@/store/useUserStore'
import {storeToRefs} from 'pinia'
import {computed} from 'vue'
import UserVue from '@/components/User.vue'


const user_store = useUserStore() // 获取 UserStore 实例
// 通过 computed() 将 store 中 state 映射成当前组件中的计算属性，具有响应性，但是是只读的
const user_age = computed(() => user_store.age)
const user_level = computed(() => user_store.level)
const user_account = computed(() => user_store.account)
const user_nickname = computed(() => user_store.nickname)
const { 
    age,
    level,
    account: userAccount,
    nickname: userNickname
} = storeToRefs(user_store)// storeToRefs 将 store 中 state 解构为组件的数据，具有响应性，还可以响应式修改
</script>
```

```vue
<template> //User.vue
    <ul>
        <li>年龄：{{ age }}</li>
        <li>等级：{{ level }}</li>
        <li>账号：{{ account }}</li>
        <li>昵称：{{ nickname }}</li>
    </ul>
</template>

<script>
import {mapState} from 'pinia'
import {useUserStore} from '@/store/useUserStore'

export default {
    computed: {
        ...mapState(useUserStore, ['age', 'level', 'account', 'nickname'])
    }
}
</script>
```

## 11.4：`Getters`
`getters`是计算得到的新的共享数据，当依赖的数据发生变化时则重新计算，所以其他组件包括`store`自己不要直接对其修改

### 11.4.1：定义`Getters`
1.  `store`采用的是选项式模式时，`getters`选项中声明的函数即为计算属性
	a.  在其函数内可通过`**this**`关键字来获取`store`实例，也可通过方法的第一个参数得到`store`实例
	b.  如果采用的是箭头函数的话，无法使用`this`关键字，为了更方便使用`store`中实例，可为其箭头函数设置第一个参数来获取`store`实例
2.  `store`采用的是组合式模式时，可通过`**computed()**`函数通过计算得到新的数据，再将其`**return**`暴露出去即可

getters【选项式】
```vue
import { defineStore } from "pinia"

export const useUserStore = defineStore('user', {
    state: () => ({
        birthday: '1992-12-27',
        age: 30
    }),
    getters: { // 通过计算得到的新的共享的数据，只读 // 如果依赖的数据发生变化，则会重新计算
        month() {
            return this.birthday.split('-')[1]  // this 为 store 实例，当然其函数的第一个参数也为 store 实例
        },
        ageStage: store => { // 因箭头函数无法使用 `this`，函数的第一个参数为 store 实例
            if(store.age < 18) return '未成年'
            if(store.age < 35) return '青年'
            if(store.age < 50) return '中年'
            if(store.age >= 50) return '老年'
        }
    }
})
```

getters【组合式】
```vue
import { defineStore } from "pinia"  
import { computed, ref } from "vue"

export const useUserStore = defineStore('user', () => {
    const birthday = ref('1992-12-27')
    const age = ref(30)
    
    const month = computed(() => { // 声明通过计算得到的共享数据，是只读的，如果依赖的数据发生变化则会重新计算
        return birthday.value.split('-')[1]
    })
    const ageStage = computed(() => {
        if (age.value < 18) return '未成年'
        if (age.value < 35) return '青年'
        if (age.value < 50) return '中年'
        if (age.value >= 50) return '老年'
    })
    return { birthday, age, month, ageStage }
})
```

### 11.4.2：在组件中使用`Getters`
1.  选项式`API`的组件中，访问`store`中的`getters`和访问`state`类似，同样可使用`**mapState()**`帮助器将`**getters**`属性映射为只读计算属性
**注意：**如果采用`**mapWritableState()**`帮助器将`store`中的`getters`映射为组件内部的计算属性，依旧可以具有响应式，一旦对其进行修改则会报错

2.  在组合式`API`组件中，访问`store`中的`getters`和访问`state`类似，直接引入对应的`store`，通过`store`对象直接获取`getters`，但是如果对其进行修改则会报错
**提示：**
如果想将`store`中的`getter`中共享的数据映射为本地组件的计算属性，我们可以这样做：
-   使用`computed(() => store._getterName_)`，具有响应式，但是只读形式
-   使用`storeToRefs(_store_)`从`store`解构`getter`依旧是计算属性，所以是只读的，一旦对其进行修改则会报错，但是具有响应式，可自定义名称

```vue
<template>
    <h3>mapState 字符串数组形式将 getters 映射成计算属性</h3>
    <ul>
        <li>月份：{{ month }}</li>
    </ul>

    <button @click="month = '5'">更改月份</button>

    <hr>
    <h3>mapState 对象形式将 getters 映射成计算属性</h3>
    <ul>
        <li>年龄阶段：{{ user_age_stage }}</li>
    </ul>

    <button @click="user_age_stage = '未知'">更改年龄阶段</button>

    <hr>
    <h3>mapWritableState 字符串数组形式将 getters 映射成计算属性</h3>
    <ul>
        <li>年龄阶段：{{ ageStage }}</li>
    </ul>

    <button @click="ageStage = '未知'">更改年龄阶段</button>

    <hr>
    <h3>mapWritableState 对象形式将 getters 映射成计算属性</h3>
    <ul>
        <li>月份：{{ birthday_month }}</li>
    </ul>
    <button @click="birthday_month = '5'">更改年龄阶段</button>

    <hr>
    生日：<input type="date" v-model="birthday">
    |
    年龄：<input type="number" min="1" max="100" v-model="age">
</template>

<script>
import { mapState, mapWritableState } from 'pinia'
import { useUserStore } from './store/useUserStore'

export default {
    computed: { // 从 store 中映射 getters 和映射 state 用法相同，都可以用 mapState
        ...mapState(useUserStore, ['month']), // 具有响应式，但是是只读的，如果修改了则会警告
        ...mapState(useUserStore, {
            user_age_stage: 'ageStage'
        }),
        // 从 store 中 映射 getters 和映射 state 用法相同，都可以用 mapWritableState
        ...mapWritableState(useUserStore, ['ageStage']), // 具有响应式，但是是只读的，如果修改了则会报错
        ...mapWritableState(useUserStore, {
            birthday_month: 'month'
        }),
        ...mapWritableState(useUserStore, ['birthday', 'age']) // 把 store 中 stage 解构为自己的计算属性
    }
}
</script>

<script setup>
import { storeToRefs } from 'pinia'
import { computed } from 'vue'
import { useUserStore } from './store/useUserStore'

// store 实例，可直接通过 store 获取 getters, 但是是只读的，如果一旦修改则会报错
const user_store = useUserStore()
  
// 通过 computed 将 getters 映射为自己的计算属性， 但是是只读的，如果一旦修改则会警告
const birthday_month = computed(() => user_store.month)
const user_age_stage = computed(() => user_store.ageStage)
  
// 通过 storeToRefs 将 getters 解构为自己的计算属性， 但是是只读的，如果一旦修改则会警告
const { month, ageStage: userAgeStage } = storeToRefs(user_store)

// 将 state 解构为自己的数据
const { birthday, age } = storeToRefs(user_store)
</script>
```

## 11.5：`Actions`
`actions`一般情况下是对`state`中的数据进行修改的业务逻辑函数，`actions`也可以是异步的，您可以在其中`await`任何`API`调用甚至其他操作！

### 11.5.1：定义`Actions`
1.  `store`采用的是选项式模式时，`actions`选项中声明的函数即可共享其函数，在其函数内可通过`this`来获取整个`store`实例
2.  `store`采用的是组合式模式时，可通过声明函数，再将其`return`暴露出去即可共享其函数

actions【选项式】
```vue
import {defineStore} from "pinia"

export const useUserStore = defineStore('user', {
    state: () => ({
        nickname: '自古风流',
        age: 20
    }),
    actions: { // 定义共享的函数，其主要是修改 state 中的数据的逻辑代码 // 其函数可以是异步的
        setUserInfo(nickname, age) {
            this.nickname = nickname // 可通过 `this` 来获取当前 store 实例
            this.age = age
        },
        setUserInfoByObject(user) {
            this.nickname = user.nickname
            this.age = user.age
        }
    }
})
```

```vue
import {defineStore} from "pinia"
import {ref} from "vue";

export const useUserStore = defineStore('user', () => {
    const nickname = ref('自古风流')
    const age = ref(20)

    function setUserInfo(user_nickname, user_age) { // 定义函数（注意：形参不要和 ref 名冲突）
        nickname.value = user_nickname
        age.value = user_age
    }

    function setUserInfoByObject(user) {
        nickname.value = user.nickname// 可通过 `this` 来获取当前 store 实例
        age.value = user.age
    }
    return {nickname, age, setUserInfo, setUserInfoByObject} // 暴露函数即可共享函数
})
```

### 11.5.2：组件中访问`Actions`
1.  在选项式 API 组件中，可以使用`mapActions(_storeObj_, _array_ | _object_)`帮助器将`actions`映射为当前组件的函数
	a.  `storeObj`引入的`store`对象
	b.  `array | object`：字符串数组形式或者对象形式
	-   【字符串数组形式】直接将`store`中`actions`的函数映射为当前组件的函数，但是不能自定义名称
	-   【对象形式时】`key`为自定义当前组件的函数名，`value`字符串形式，是`store`中`actions`的函数名
2.  在组合式`API`组件中，直接引入对应的`store`，通过`store`对象直接获取`actions`

**提示：**如果想将`store`中的`actions`中函数映射为本地组件的函数，可将`store`解构出对应的函数即可，也可自定应函数名，此处不可通过`storeToRefs(_store_)`函数


```vue
<template> //组件中获取 actions【选项式】
    <ul>
        <li>昵称：{{ nickname }}</li>
        <li>昵称：{{ age }}</li>
    </ul>
    
    <button @click="setUserInfo('Tom', 15)">修改信息</button>
    <button @click="set_info_by_object({ nickname: 'Jack', age: 40})">
        修改信息
    </button>
</template>

<script> 
import {mapActions, mapState} from "pinia"
import {useUserStore} from "@/store/useUserStore"

export default {
    computed: {
        ...mapState(useUserStore, ['nickname', 'age'])
    },
    methods: { // 使用 mapActions 将 store 中的 actions 映射为自己的函数
        ...mapActions(useUserStore, ['setUserInfo']), // 采用函数形式，无法自定义映射的函数名
        ...mapActions(useUserStore, { // 采用对象形式，可自定义映射的函数名
            set_info_by_object: 'setUserInfoByObject'
        })
    }
}
</script>
```

```vue
<template>  //组件中获取 actions【组合式】
    <ul>
        <li>昵称：{{ nickname }}</li>
        <li>昵称：{{ age }}</li>
    </ul>
  
    <span>直接使用 store 执行 actions：</span>
    <button @click="user_store.setUserInfo('Tom', 15)">修改信息</button>
    <button @click="user_store.setUserInfoByObject({ nickname: 'Jack', age: 40})">
      	修改信息
    </button>

    <hr>
    <span>将 store 中的 actions 映射成自己的函数：</span>
    <button @click="setUserInfo('Annie', 45)">修改信息</button>
    <button @click="set_user_info_object({ nickname: 'Drew', age: 50})">
      	修改信息
    </button>
</template>

<script setup> 
import {useUserStore} from "@/store/useUserStore"
import { storeToRefs } from "pinia"

const user_store = useUserStore() // 可直接使用 store 执行 actions
const {nickname, age} = storeToRefs(user_store)

// 可将 store 中的 actions 映射为自己的函数，可自定映射的函数名（不可使用 storeToRes 函数）
const {setUserInfo, setUserInfoByObject: set_user_info_object} = user_store
</script>
```







