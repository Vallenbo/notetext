`Axios`是一个基于`promise`网络请求库，作用于`node.js`和浏览器中
`Axios`在服务端它使用原生`node.js``http`模块，而在客户端（浏览端）则使用 `XMLHttpRequests`
`Axios`可以拦截请求和响应、转换请求和响应数据、取消请求、自动转换`JSON`数据
`Axios`安装方式：==npm install axios==

## 12.1：`Axios`配置项
这些是创建请求时最常用的配置选项；详细的配置项请前往[Axios](https://www.axios-http.cn/docs/req_config)官网
**提示**：只有**url**是必需的；如果没有指定`method`，则请求将默认使用**GET**方法

```json
{
    url: '/user', // 请求的服务器地址 URL        
    method: 'GET', // 请求方式，默认值 GET
    baseURL: 'https://some-domain.com/api/', // 如果 url 不是绝对地址，则会发送请求时在 url 前方加上 baseURL
    headers: {'X-Requested-With': 'XMLHttpRequest'}, // 自定义请求头
    params: { ID: 12345 }, // 与请求一起发送的 URL 参数
    data: { firstName: 'Fred' },  // 作为请求体被发送的数据，仅适用 'PUT', 'POST', 'DELETE 和 'PATCH' 请求方法
    timeout: 1000, // 请求超时的毫秒数，如果请求时间超过 `timeout` 的值，则请求会被中断，默认值是 `0` (永不超时)，
    responseType: 'json', // 期望服务器返回的数据类型，选项包括: 'arraybuffer', 'document', 'json', 'text', 'stream'， 浏览器专属：'blob'，默认值 json
    transformRequest: [function (data, headers) { // 允许在向服务器发送前，修改请求数据，它只能用于 'PUT', 'POST' 和 'PATCH' 这几个请求方法
        return data; // 对发送的 data 进行任意转换处理
    }],
    transformResponse: [function (data) { // 在传递给 then/catch 前，允许修改响应数据
    	return data; // 对接收的 data 进行任意转换处理
    }]
}
```

## 12.2：发送请求
可以向 `**axios**`传递相关配置来创建请求

### 12.2.1：`axios(_config_)`

发送一个 POST 请求
```
axios({
    method: 'POST', // 请求方式
    url: '/example-url/……', // 请求地址
    // …… 其他配置 ……
})
```

发送一个 GET 请求
```
axios({
    method: 'GET', // 请求方式，可省略不写
    url: '/example-url/……', // 请求地址
    // …… 其他配置 ……
})
```

### 12.2.2：`axios(_url_[, config])`

发送一个 POST 请求
```
axios(
    '/example-url/……', // 请求地址
    {
        method: 'POST', // 请求方式
        // …… 其他配置 ……
    }
)
```

发送一个 GET 请求
```
axios(
     '/example-url/……', // 请求地址
    {
        method: 'GET', // 请求方式，可省略不写
        // …… 其他配置 ……
    }
)
```

### 12.2.3：请求方式别名
为了方便起见，已经为所有支持的请求方法提供了别名
1.  `axios.**request**(_config_)`
2.  `axios.**get**(_url_[, _config_])`
3.  `axios.**delete**(_url_[, _config_])`
4.  `axios.**head**(_url_[, config])`
5.  `axios.**options**(_url_[, config])`
6.  `axios.**post**(url[, _data_[, _config_]])`
7.  `axios.**put**(url[, _data_[, _config_]])`
8.  `axios.**patch**(url[, _data_[, _config_]])`

**注意：**在使用别名方法时，`**url**`、`**method**`、`**data**` 这些属性都不必在`**config**`中指定

发送一个 POST 请求
```
axios.post(
    '/example-url/……', // 请求地址
    { /* 请求体中的参数 */ },
    { /* 其他配置 */ },
)
```

发送一个 GET 请求
```
axios.get(
    '/example-url/……', // 请求地址
    { /* 其他配置 */ },
)
```

## 12.3：自定义创建实例
`axios.create([_config_])`：调用`create`函数传入自定义配置，来创建自定义`axios`实例

src/request/axiosInstance .js
```js
import axios from 'axios'

const request = axios.create({
    baseURL: 'https://some-domain.com/api/',
    timeout: 1000,
    headers: {'X-Custom-Header': 'foobar'}
})

export default request
```
**使用自定义实例发送请求：**

```
import request from '@/request/axiosInstance.js'
request({
    method: 'POST', // 请求方式
    url: '/example-url/……', // 请求地址
    // …… 其他配置 ……
})
```

```
import request from '@/request/axiosInstance.js'
request('/example-url/……', // 请求地址
    {
        method: 'POST', // 请求方式
        // …… 其他配置 ……
    }
)
```

```
import request from '@/request/axiosInstance.js'
request.post(
    '/example-url/……', // 请求地址
    { /* 请求体中的参数 */ },
    {/* …… 其他配置 …… */}
)
```

## 12.4：响应数据
发送请求后通过`.then(_response_ => {})`来获取服务器响应的数据
`response`响应式结构：
1.  `**data**`：服务器提供的响应【最重要】
2.  `status`：来自服务器响应的 HTTP 状态码，成功为`200`，请求地址不存在为`404`，服务器异常为`500`，请求方式错误为`405`……
3.  `statusText`：来自服务器响应的 HTTP 状态信息
4.  `headers`：服务器响应头
5.  `config`： 请求的配置信息
6.  `request`：生成此响应的请求，在`node.js`中它是最后一个`ClientRequest`实例，在浏览器中则是`XMLHttpRequest`实例

```
axios({
    method: 'GET',
    url: '/example-url/……'
    // …… 其他配置 ……
}).then(response => {
    console.log(response.data) // 获取服务器传递来的数据
})
```

```
axios.get(
    url: '/example-url/……'
    { /* …… 其他配置 …… */ }
)
.then(response => {
    console.log(response.data) // 获取服务器传递来的数据
})
```

## 12.5：请求错误处理
发送请求后，使用`.**catch(**_**error**_ **=> {})**`来处理此次请求异常，请求成功发出且服务器也响应了状态码，但状态代码超出了`2xx`的范围
``
```
axios({
    method: 'GET', // 请求方式
    url: '/example-url/……', // 请求地址
}).catch(error => {
    console.log('请求失败！')
})
```

## 12.6：解决跨域问题
1.  跨域：指的是浏览器不能执行其他网站的脚本；它是由浏览器的同源策略造成的，是浏览器对`javascript`施加的安全限制
2.  同源策略：是指协议，域名，端口都要相同，其中有一个不同都会产生跨域
3.  浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源，如果未处理跨域访问则会在请求时控制台出现`**Access-Control-Allow-Origin**……`的报错信息
4.  如何处理跨域问题，可在`vite`项目的`vite.config.js`文件中添加`proxy`代理

```
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [vue()],
    resolve: {
        alias: {
            '@': fileURLToPath(new URL('./src', import.meta.url))
        }
    },
    server: { // 服务
        proxy: { // 代理
            '/ok': {
                target: 'https://v.api.aa1.cn/api', // 代理后台服务器地址
                changeOrigin: true, //允许跨域               
                rewrite: path => path.replace(/^\/ok/,'') // 将请求地址中的 /ok 替换成空
            }
        }
    }
})
```

## 12.7：例子
本次请求测试采用的是APISpace提供的测试API，当然如果你有自己的测试的API也可测试自己的API  
提示：测试APISpace提供的接口需要登录其账号获取鉴权私钥，领取测试案例的使用次数方可测试  

代理服务来解决跨域问题
```vue
import { fileURLToPath, URL } from 'node:url'
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
    plugins: [vue()],
    resolve: {
        alias: {
            '@': fileURLToPath(new URL('./src', import.meta.url))
        }
    },
    server: { // 服务
        proxy: { // 代理
            '/apispace': {
                target: 'https://eolink.o.apispace.com/', // 代理后台服务器地址
                changeOrigin: true, //允许跨域               
                rewrite: path => path.replace(/^\/apispace/,'') // 将请求地址中的 /ok 替换成空
            }
        }
    }
})
```

### 12.7.1：智能天气实况
支持全国以及全球多个城市的天气查询，包含国内`3400+`个城市以及国际`4`万个城市的实况数据，更新频率分钟级别
[**智能天气实况**](https://www.apispace.com/eolink/api/456456/apiDocument)

**请求方式** `**GET**`
**请求地址**`**https://eolink.o.apispace.com/456456/weather/v001/now**`
**请求头**   `X-APISpace-Token`  鉴权私钥
			`Authorization-Type` 鉴权方式，值为：apikey

**请求**`**URL**`**参数**   `areacode`城市[ID](https://easy-open-link.feishu.cn/wiki/wikcnXfeZ1lmUCbCSac2hYAEFb2)，必填，类型：`String`

**返回数据格式** `**JSON**`
**返回案例**
{
    "status": 0, // 状态码
    "result": {
        "location": {
            "areacode": "JPN10041001001",	// 城市 ID
            "name": "足立区",		    // 城市中文名
            "country": "日本",		    // 所属国家中文名
            "path": "足立区,足立区,东京都,日本"    // 行政区划路径
        },
        "realtime": {
            "text": "多云",		// 天气现象，string 类型
            "code": "01",			// 天气现象编码，string 类型
            "temp": 6.5,			// 气温，单位℃，double 类型
            "feels_like": 6,		// 体感温度，单位℃，int 类型
            "rh": 38,			// 相对湿度，单位%，int 类型
            "wind_class": "2级",		// 蒲福氏风级，string 类型
            "wind_speed": 2.5,	// 风速，单位m/s，double 类型
            "wind_dir": "南风",		// 风向，string 类型
            "wind_angle": 187,	// 风向角度，0表示正北，180表示正南，int 类型
            "prec": 0.0,			// 过去1小时降水量，单位毫米(mm)，double 类型
            "clouds": 99,		// 云量，单位%，int 类型
            "vis": 12085,		// 能见度，单位米(m)，int类型
            "pressure": 1020,		// 气压，单位百帕(hPa)，int类型
            "dew": -6,			// 露点温度，单位℃，int 类型
            "uv": 2,			// 紫外线指数，int 类型
            "snow": 0.0,		// 降雪量，单位厘米(cm)，double 类型 #国内城市不支持#
            "weight": 0,		// 文案权重，int类型
            "brief": "今日惊蛰",		// 天气短文案，string类型
            "detail": "今日惊蛰，春雷惊百虫",		// 天气长文案 ，string 类型
        },
        "last_update": "2021-03-05 19:07:44"	// 数据更新时间(北京时间)
    }
}


请求测试案例：  

获取智能天气【选项式】

获取智能天气【组合式】



### 12.7.2：随机笑话大全
具有最新、最及时的笑话段子；笑话具有篇幅短小，故事情节简单而巧妙，往往出人意料，给人突然之间笑神来了的奇妙感觉的特点

