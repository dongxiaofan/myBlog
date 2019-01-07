---
title: 关于Axios
date: 2018-12-21 15:45:59
---
#### Axios是什么
Axios是一个基于promise的http库，可以用在浏览器和node.js中。

#### Axios的特点
* 从浏览器中创建 XMLHttpRequests
* 从 node.js 创建 http 请求
* 支持 Promise API
* 拦截请求和响应
* 转换请求数据和响应数据
* 取消请求
* 自动转换 JSON 数据
* 客户端支持防御 CSRF
PS：
防止CSRF:就是让你的每个请求都带一个从cookie中拿到的key, 根据浏览器同源策略，假冒的网站是拿不到你cookie中得key的，这样，后台就可以轻松辨别出这个请求是否是用户在假冒网站上的误导输入，从而采取正确的策略。

### 引入方式
``` bash
$ npm install axios // 使用 npm
$ bower install axios // 使用 bower
<script src="https://unpkg.com/axios/dist/axios.min.js"></script> // 使用 cdn
```

### 请求配置
``` bash
{
  url: 'url', // 用于请求的服务器 URL
  method: 'get', // 创建请求时使用的方法,默认是 get,可设置为post
  baseURL: '/api', // 将自动加在 'url' 前面，除非 'url' 是一个绝对 URL
  transformRequest: [function (data) { // 'transformRequest' 允许在向服务器发送前，修改请求数据
    return data;
  }],
  transformResponse: [function (data) { // 'transformResponse' 在传递给 then/catch 前，允许修改响应数据
    return data;
  }],
  headers: {'X-Requested-With': 'XMLHttpRequest'}, // 即将被发送的自定义请求头
  params: { // 与请求一起发送的 URL 参数
    ID: 12345
  },
  paramsSerializer: function(params) { // 负责 'params' 序列化的函数
    return Qs.stringify(params, {arrayFormat: 'brackets'})
  },
  data: { // 作为请求主体被发送的数据
    firstName: 'Fred'
  },
  timeout: 1000, // 指定请求超时的毫秒数(0 表示无超时时间)
  withCredentials: false, // 跨域请求时是否需要使用凭证
  auth: { // 'auth' 表示应该使用 HTTP 基础验证，并提供凭据
    username: 'janedoe',
    password: 's00pers3cret'
  },
  responseType: 'json', // 表示服务器响应的数据类型
  proxy: { // 定义代理服务器的主机名称和端口
    host: '127.0.0.1',
    port: 9000,
    auth: : { // 表示 HTTP 基础验证应当用于连接代理，并提供凭据
      username: 'mikeymike',
      password: 'rapunz3l'
    }
  },
}
```

### 拦截器
在请求或响应被 then 或 catch 处理前拦截它们
``` bash
  // 添加请求拦截器
  axios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
  }, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
  });

  // 添加响应拦截器
  axios.interceptors.response.use(function (response) {
    // 对响应数据做点什么
    return response;
  }, function (error) {
    // 对响应错误做点什么
    return Promise.reject(error);
  });
```

如果你想在稍后移除拦截器，可以这样
``` bash
  var myInterceptor = axios.interceptors.request.use(function () {/*...*/});
  axios.interceptors.request.eject(myInterceptor);
```

### ajax和axios、fetch的区别
JQuery ajax 是对原生XHR的封装，除此以外还增添了对JSONP的支持。经过多年的更新维护，真的已经是非常的方便了，优点无需多言；如果是硬要举出几个缺点，那可能只有：
1.本身是针对MVC的编程,不符合现在前端MVVM的浪潮
2.基于原生的XHR开发，XHR本身的架构不清晰。
3.JQuery整个项目太大，单纯使用ajax却要引入整个JQuery非常的不合理（采取个性化打包的方案又不能享受CDN服务）
4.不符合关注分离（Separation of Concerns）的原则
5.配置和调用方式非常混乱，而且基于事件的异步模型不友好。

fetch号称是AJAX的替代品，是在ES6出现的，使用了ES6中的promise对象。Fetch是基于promise设计的。Fetch的代码结构比起ajax简单多了，参数有点像jQuery ajax。但是，一定记住fetch不是ajax的进一步封装，而是原生js，没有使用XMLHttpRequest对象。
fetch的优点：
1.语法简洁，更加语义化
2.基于标准 Promise 实现，支持 async/await
3.同构方便，使用 [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)
4.更加底层，提供的API丰富（request, response）
5.脱离了XHR，是ES规范里新的实现方式

fetch的缺点：
1.fetch只对网络请求报错，对400，500都当做成功的请求，服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject。
2.fetch默认不会带cookie，需要添加配置项： fetch(url, {credentials: 'include'})
3.fetch不支持abort，不支持超时控制，使用setTimeout及Promise.reject的实现的超时控制并不能阻止请求过程继续在后台运行，造成了流量的浪费
4.fetch没有办法原生监测请求的进度，而XHR可以
