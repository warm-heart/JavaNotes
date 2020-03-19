简介
静态资源服务器，反向代理，负载均衡等，这些场景下，nginx无处不在。但是本人是在解决单个tomcat承受不了并发量的前提下，才走进的nginx。以下都是个人的总结，如果有不对的话，可以指出来，毕竟是自学的。。。
下载nginx
http://nginx.org/en/download.html  这个就不多说了，直接下载就行了。
一、静态资源服务器
使用nginx实现静态资源服务器，我们可以通过nginx来访问静态资源。修改nginx配置（conf/nginx.conf）文件为：
  server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
123456789101112
listen： 这个参数大家应该都不陌生，就是监听的端口号。
server_name： 浏览器上输入的域名。
location： 表示url匹配，/表示全部匹。
root： 表示匹配成功之后进入的目录。
index： 表示默认的页面。
效果显示：
我们在html这个目录下放一个mzd.html，然后开发浏览器访问。


扩展:
默认的文件夹（html文件夹）是可以改变的，nginx默认访问的是nginx.exe同一级别目录的html文件夹，我们可以修改location和root来修改这个访问的文件夹。
 location /nginx_static{
            root   F:/;
            index  index.html index.htm;    
  }
1234
只要浏览器访问的是www.tuesdayma.com/nginx_static/XXXX将会访问F盘中nginx_static文件夹下面的静态资源。如果只想改变默认的访问文件夹，那么只要修改root即可。
二、根据域名访问不同路径
本地测试的话，可以修改host文件，我准备了两个域名：www.tuesdayma.com和static.tuesdayma.com。修改nginx配置文件为：
   server {
        listen       80;
        server_name  static.Tuesdayma.com;
        location / {
            root   extend/static;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
   server {
        listen       80;
        server_name  www.Tuesdayma.com;
        location / {
            root   extend/www;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
123456789101112131415161718192021222324
效果显示：


三、反向代理
实质： 个人理解反向代理就是nginx拦截动态请求之后转发给某个tomcat。这个在集群和分布式都可以使用这个来进行配置转发。
作用： 隐藏真实的访问ip地址。我们可以看到流程图中我们访问的最多也就是公网的ip，但是具体tomcat在那个ip是不知道的，这样就能减少tomcat被攻击，提高了服务器的安全性。
流程图（个人版本）：

配置文件修改：
     server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            proxy_pass   http://127.0.0.1:8090;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
123456789101112
proxy_pass: 127.0.0.1可以换成任何一个通的内网地址，这个ip表示你要真实访问的tomcat所在的位置，proxy_pass的值就表示你真正访问的域名是什么（站在公网服务器角度来说）。
四、location匹配模式及顺序




表示
描述




location = /uri地址
=开头表示精确匹配，只有完全匹配上才能生效。


location ^~ /uri地址
^~ 开头对URL路径进行前缀匹配，并且在正则之前。


location ~ 正则表达式
~开头表示区分大小写的正则匹配。


location ~* 正则表达式
~*开头表示不区分大小写的正则匹配。


location /uri地址
不带任何修饰符，也表示前缀匹配，但是在正则匹配之后。


location /
通用匹配，拦截所有，但是优先级最低，只有前面都没有被拦截的情况下，才会被拦截到这里。


五、动静分离
静态资源： html、js、css、图片、音乐、视频等。
动态资源： 我的理解就是我们所说的接口。这里需要注意的是：themleaf、freemark这些模板引擎的html不是静态资源而应该属于动态资源（个人认为）。
实现动静分离的两种方法： 通过不同域名来拦截和location来拦截。
不同域名来拦截： 用大白话来说就是，动态请求和静态请求使用不同的域名。比如所有的静态资源都使用static.tuesdayma.com域名来访问，所有的接口都使用www.tuesdayma.com来请求。
  #动态请求拦截
  server {
        listen       80;
        server_name  www.Tuesdayma.com;
        location / {
           # proxy_redirect off;  
           # proxy_set_header Host $host;  
           # proxy_set_header X-Real-IP $remote_addr;  
           # proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
            proxy_pass   http://127.0.0.1:8080;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
   #静态请求拦截
   server {
        listen       80;
        server_name  static.Tuesdayma.com;
        location /nginx_static{
            root   F:/;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }  
    }

123456789101112131415161718192021222324252627282930
优点：扩展性比较强，静态资源是什么都可以。
缺点：存在跨域问题，所有的html全靠ajax请求来请求接口，后端域名和前端不一致导致跨域问题。
解决：1、在nginx中把注释掉的放开；2、将tomcat的所有接口配置成域名跨域访问（这个东西个人建议可以在tomcat中写个拦截器进行拦截，然后统一处理）。
 response.setHeader("Access-Control-Allow-Origin", "*");
 response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
 response.setHeader("Access-Control-Max-Age", "3600");
 response.addHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
1234
location来拦截： 这个只配置一个server，然后配置两个location，一个通过正则表达式拦截静态资源，还有一个拦截.do结尾的接口请求。
优点：不存在跨域问题。
缺点：扩展性太差，静态资源的类型不可确定，每增加一种类型都需要重新修改配置文件并且重启nginx。
六、负载均衡
定义： 为了解决高并发问题，负载均衡服务器拦截所有的请求，采用负载均衡算法，分配到不同的tomcat上。
作用： 减少单台tomcat的压力
upstram  XXX： 表示负载均衡服务器，也是通常再说的上游服务器。
三种基本的负载均衡算法： 轮询、权重、ip绑定。
轮询： 这是nginx默认的负载均衡算法，简单来说就是从上到下按顺序轮流（127.0.0.1:8082轮完就轮到127.0.0.1:8081，127.0.0.1:8081轮完就轮到127.0.0.1:8082）。注意：mzd的地方需要保持一致。。。
 upstream  mzd {
       server 127.0.0.1:8082;
       server 127.0.0.1:8081;
    }
    server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            proxy_pass  http://mzd;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

1234567891011121314151617
权重： 我的理解就是哪台服务器配置好就多轮几次，或者你就想某个服务器多轮几次。简单来说就是轮到次数的比例，数字越大表示轮到的概率越大。（个人认为weight都设置为1的时候和轮询没什么区别）
 upstream  mzd {
       server 127.0.0.1:8082 weight=2;
       server 127.0.0.1:8081 weight=3;
    }
    server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            proxy_pass  http://mzd;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

1234567891011121314151617
ip绑定： 我的理解就是你第一次访问的时候，nginx会根据你的ip通过哈希算法，算出某个值，然后取分配哪个tomcat，当你第二次访问，第三次访问。。。之后的任何一次访问都是去请求那个第一次访问的tomcat。
 upstream  mzd {
	
       server 127.0.0.1:8082 ;
       server 127.0.0.1:8081 ;
       ip_hash;
    }
    server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            proxy_pass  http://mzd;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

12345678910111213141516171819
问题1： 可能在测试负载均衡的时候，你可能会发现浏览器一直在访问同一个tomcat，这时候你第一反应就是我的负载均衡配置的有问题。。。其实这是正常的反应，不过按上面的方法配的应该是没问题的，其实有两种可能：一就是确认一下所有的服务器都能不能正常访问；二不要点浏览器上面的刷新按钮或者什么重新加载之类的，还有Ctrl+F5，这都可能会导致这个问题（本人被坑了一下午，还以为是负载均衡哪里配错了。。。）。
问题2：故障转移问题，我想应该都会有一个疑问，就是负载均衡的时候，其中一个请求被分配到的那个tomcat挂了之后，会怎么样？？？上面说的三种模式结果都是这样。。。。

其实nginx有默认故障转移机制的，但是很慢（本人测了一下，默认好像是要60秒左右，一分钟后就有自动发送到其他tomcat上去了，其实就是proxy_connect_timeout的默认值）。。。个人觉得生产环境根本不可能用nginx默认的故障转移机制的，是在等待太久了。。。
解决： 设置这些参数之后就能快点。。。
    server {
        listen       80;
        server_name  www.tuesdayma.com;
        location / {
            proxy_pass  http://mzd;
            proxy_connect_timeout 3s;
            proxy_read_timeout 5s;
            proxy_send_timeout 3s;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
12345678910111213141516
proxy_connect_timeout： nginx向tomcat发起连接，即第一次握手等待tomcat回应的超时时间，tomcat的这次回应只是说明能正常连接，并没有响应具体请求的内容。
proxy_send_timeout： nginx将请求发送给tomcat的超时时间，应该是确认能正常连接之后向tomcat发送真正的业务请求。
proxy_read_timeout： tomcat接受到真正业务请求之后，nginx等待tomcat响应具体请求的内容的超时时间。差不多可以理解tomcat处理具体请求时间的最大值，也就是tomcat必须在这个时间内做完业务逻辑处理。
注意： 以上三个参数的都是个人的理解，因为网上对着三个参数什么都有。。。个人觉得proxy_read_timeout应该设置为20秒左右，因为针对于不同的业务处理，如果比较复杂，则耗时会比较长。