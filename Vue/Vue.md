### 新建项目

```
vue create note-fron
```

### Vue使用elementUI和axios

```
npm i element-ui -S
cnpm install --save axios vue-axios 


//main.js
import ElementUI from 'element-ui'
import axios from 'axios'
import VueAxios from 'vue-axios'
import 'element-ui/lib/theme-chalk/index.css'


Vue.use(VueAxios, axios)
Vue.use(ElementUI)
```



### 定义变量

```
     data() {
            return {
                users: {},
                use: 0,
                decription: 'today'
            }
        },
```

vue路由传值

```
  <button @click="routerTo">click here to news page</button>
      routerTo: function () {
             this.$router.push({name: 'Home', params: {userName:'cooper',password:'1122'}});
            }
            
     接收端
        data() {
            return {
                userName:'',
                password:'',
                description: 'today'
            }
        },
          mounted() {
            this.userName =this.$route.params.userName,
            this.password =this.$route.params.password
        }
```

### axios

#### get

默认是表单编码，后端用@RequestParam注解接收，

```

// 为给定 ID 的 user 创建请求
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });

// 上面的请求也可以这样做
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
       
```

#### post

默认情况下，axios将JavaScript对象序列化为JSON。此时后端用@RequestBody注解接收实体类参数，如果 要以application / x-www-form-urlencoded格式发送数据（后端用@RequestParam注解接受），可以使用以下选项之一。

```
方式一
const params = new URLSearchParams();
params.append('param1', 'value1');
params.append('param2', 'value2');
axios.post('/foo', params);

方式二
const qs = require('qs');
axios.post('/foo', qs.stringify({ 'bar': 123 }));
或者以另一种方式（ES6），
import qs from 'qs';
const data = { 'bar': 123 };
const options = {
  method: 'POST',
  headers: { 'content-type': 'application/x-www-form-urlencoded' },
  data: qs.stringify(data),
  url,
};
axios(options);
```