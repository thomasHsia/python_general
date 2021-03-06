复习: 
flask只包含框架核心: 路由和对应的视图函数
服务器和服务器对应可以执行的代码--> 框架

```
mkvirtualenv flask_py2
pip install flask==0.10.1

pip freeze > requirements.txt
pip install -r requirements.txt

from flask import Flask
app = Flask(__name__)  # extra初始化参数: static_url_path, static_folder, template_folder

@app.route('/')
def index():
  return "hello world"
  
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

配置参数:
```
class MyConfig(object):
    DEBUG = True
app.config.from_object(MyConfig)
或者
app.config.from_pyfile("yourconfig.cfg")
```

在视图读取配置参数       from flask import current_app
app.config.get() 或者  current_app.config.get()

1. 同一个路由装饰多个视图函数
2. 同一个视图多个路由装饰器
3. 利用methods限制路由中的访问方式
4. 使用url_for进行反解析

动态路由: 路由传递的参数默认是string类型处理, django是在路由中写regex提取，flask中用到了转换器，用<>提取，
<>中参数默认是string类型, 如果定制flask其中有int,float,path三种类型; :之前是类型，之后是参数名；
如果是要用自定义转换器，写一个类; from werkzeug.routing import BaseConverter; 这个类在初始化方法
中规定需要有的属性regex: 
```
class re_url(BaseConverter):
    def __init__(self, url_map, *args):
        super(re_url self).__init__(url_map)
        self.regex = args[0]

    def to_python(self, value):
        return value   # /index/<
        
    def to_url(self, value):
       return value   # 对应某一个试图函数中有的url_for(,): 
                      # url_for()反解析试图函数对应的路由，第一个参数是某一个试图函数的函数名，后面是这个试图函数对应的路由需要传递的参数
       
app = Flask(__name__)
app.url_map.converters["re"] = re_url

@app.route('/user/< re("[a-z]{3}") :id>')  # re("[a-z]{3}")是转换器的名字
def hello_itcast(id):
    return 'hello %s' % id
```

获取请求参数: from flask import request
request: 常用属性 data, form, args, cookies, headers, method, url, files

e.g. 
```
from flask import request

@app.route('/upload', methods = ["GET", "POST"]
def upload_file():
    if request.method == "POST":
        f = request.files['the_file']
        f.save('/var/www/uploads/uploaded_file.txt')
       ...
```

### day 2 

2. abort 函数与 errorhandler;
3. 构建试图函数返回值的两种方式: 
    a. 元组: 可以返回一个元组,元组必须是 (response, status, headers)的形式，并且至少包含一个元素。
    status值会覆盖状态代码，headers可以是一个列表或字典，作为额外的消息表头值
    b. 
    ```
    make_response: 
        resp = make_response()
        resp.headers["sample"] = "value"
        resp.status = "404 NOT FOUND"
    ```
4. 使用 jsonify 返回JSON数据
```
    import json; 
    json.dumps(): python普通类型转换为json类型
    json.loads(): json类型转化为python类型
    jsonify帮我们将数据直接转换为json字符串，并且设置Content-Type响应头为application/json
```

5. redirect:
    因为redirect重定向到某一个URL/route, 但是知道的是执行某一个功能的视图函数,所以redirect多与url_for()反解析试图函数的URL一起用
6. 设置和读取cookie: 
```
    resp = make_response()
    resp.set_cookie(key, value = "", max_age = None)
    resp.delete_cookie(key)
  e.g.
    resp.set_cookie("itcast", "python", max_age = 3600)
    resp.headers["itcast2"] = "python2"
    resp.headers["Set-Cookie"] = "itcast=python; Expires=Fri, 01-Feb-2019 21:50:35 GMT; Max-Age=3600; Path=/"
    
    get_cookie: cookie = request.cookies.get("itcast")
                return cookie
    resp.delete_cookie("itcast")
```

7. 设置和读取session: 

    - 为了使用session,  需要设置 secret_key
    - 视图函数保存session数据, session用法与dict一样:
    - 获取session数据:session.get(), 取法是dict的取法
    - 本质上，session是存储在服务器中的，具体来讲是存在服务区后段的数据库中，每一个session相当于一个locker，浏览器要拿出对应的session:
    - 要用session_id－－－这个浏览器中cookie唯一的标识, 去找存在数据库中的session的locker
    - (浏览器中cookie的session_id对应后端数据库中的session)
    - session可以存在MySQL, redis, 文件，或者程序运行内存中 
    - session: 现实中后台跨服务器/多台服务器: nginx配置负载均衡使得浏览器访问前后两次去不同的服务器: 第一次访问第一台服务器，
    第一台服务器保存了session；第二次访问的时候因为第二台没有被访问过所以没有session记录，但是实际上是访问过的: 
    解决问题: 配置nginx: 使得同一个地方来的(e.g. IP)访问总是导向同一台服务器，这样就不会有session跨平台的问题了;
    现在没有这个********session跨服务器*********的问题了; 
    因为session保存的地方是单独的一台服务器，单独的一个服务器保存下来的
