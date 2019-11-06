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

### 发送请求

```
   数据定义
     data() {
            return {
                user: {}
            }
        },
     //函数   
   created() {
            this.$http.post("http://localhost:8082/vue", {userId: 123, password: this.message, userName: 1234})
                .then((result) => {
                    var res = JSON.parse(JSON.stringify(result))
                    console.log(res)
                    if (res.data.code == 200) {
                        this.user = res.data.data
                    }
                    else alert(res.data.msg)
                })
                .catch((err) => {
                    return err
                })

        },
        
        
```

