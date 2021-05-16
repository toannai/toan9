---
layout: post
title: Làm sao để load cấu hình vào Flask App cho ngầu,
date: 2020-06-07 00:32:20 +0700
description: # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Python]
---
Bữa nay rảnh tôi học code python. Do tính dễ học nên tôi chọn flask. Định viết 1 ứng dụng nhỏ, cơ mà tôi khó tính, dù nhỏ nhưng cũng phải ngăn nắp từ cái file cấu hình trở đi. Thế là tôi lại search gg đọc viết sao cho dễ coi, cho đẹp :) Và thế là sinh ra bài này. Bài viết chỉ là note le ve của cá nhân thôi chứ tôi không dám gáy vì trong list bạn tôi nhiều pro flask - kiếm cơm bằng flask :)


![Intro]( {{site.url}}/assets/img/2019/11/17/flaskconfig.jpg){:width="700px"}

Mỗi ứng dụng đều cần một số cấu hình các cấu hình nhất định, thông thường các cấu hình này được nhét một file cấu hình hoặc load từ biến môi trường vào (KLQ cơ mà tôi vẫn có những đe ve lốp pơ hardcode cấu hình vào trong code - buồn cười quá đi, cơ mà không tính ở đây ^^). Theo lẽ thường Flask không ngoại lệ, nó cũng cung cấp cho chúng ta một số cách thiết lập cấu như sau.

* File base
* Object base
* enviroment-variable-based
* instance-folders-based

### Có thể bạn đã biết :)

Chắc các bạn đã biết rồi nhưng để hệ thống tôi cứ liệt kê ra đây. Trong mỗi app flask từ khi sinh ra nó đã có khơ khớ các thông số cấu hình mặc định. Để coi ta chỉ việc in nó ra thôi.

```python
ret = app.config
```

Và kết quả in ra như sau

``` 
<Config {'ENV': 'production', 'DEBUG': True, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SECRET_KEY': None, 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': None, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0, 43200), 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'JSON_AS_ASCII': True, 'JSON_SORT_KEYS': True, 'JSONIFY_PRETTYPRINT_REGULAR': False, 'JSONIFY_MIMETYPE': 'application/json', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093Ư>
```

Không quá ngạc nhiên ta thường thấy trong code đoạn cấu hình tương tự như sau

```python  
app.config['SECRET_KEY'] = 'the random string'    
```

Bản chất là ae đang override biến SECRET_KEY trong đối tượng app.config phía trên.  Cái này cơ bản cơ mà liên quan nên tôi cứ nhắc lại ở đây (Có chút ít liên quan). Giờ quay lại chủ đề, ta lượn lờ qua một số kiểu cấu hình cơ bản hay được sử dụng.

### 1. File - based là như lào

Flask cho phép chúng ta load cấu hình từ một file vào ứng dụng. Dĩ nhiên file này phải viết theo một format nhất định, mà ở đây flask quy định là định dạng ini với cặp name/value trong một file text và ngăn cách với nhau bởi một dấu phẩy. 

Lấy ví dụ cho t/h này ta tạo file config.ini trong thư mục code với nội dung như sau:

```python
MY_SETTING='OK'
DEBUG=True
```

Để load file cấu hình trên vào chương trình ta sử dụng đoạn code sau:

```python
app.config.from_pyfile(config_file.cfg) 
```

Để gọi ra từng gía trị ta làm như sau:

```python
my_setting = app.config['MY_SETTING'] #Lấy ra gía trị cấu hình có name là MY_SETTING
```

### 2. Object - based

Xét trường hợp ta có 3 môi trường dev, test và product với một số cấu hình giống nhau ở các bên (name giống nhau, value khác nhau). Hàng ngày do release version code liên tục qua các pharse dev -> test -> product. Tính ra nếu làm theo cách trên ta cứ phải quay tay change các value liên tục khi đổi giữa các môi trường - việc này tốn t/g quá :'( và dễ nhầm lẫn nữa ( và ngày nay đặc biệt với cái gọi là CICD thì việc change giữa các môi trường này nó là việc quá thường xuyên). Để xử case này flask support ace một cách khác là sử dụng object-base configuration :). Nói thì đài dòng cơ mà mô tả cách làm nó đại loại là như sau.

**B1.** Đầu tiên tạo ra 3 class như sau (tạo bn class con thì tùy ace trong từng case) trong 1 file config.py

```python
class BaseConfig(object):
    DB = 'product'
    DEBUG = False

class DevelopmentConfig(BaseConfig):
    DB = 'dev'
    DEBUG = True

class TestingConfig(BaseConfig):
    DB = 'test'
    DEBUG = False
```

 (wtf: Dễ nhận thấy ở đây ta xài các biến class (static varible để lưu gía trị cấu hình) lý giải đơn giản là khỏi phải tạo object khi xài thôi ^^)

**B2.** Đến lúc xài ta làm như sau:

* Với môi trường Dev
   
```python
   from config import DevelopmentConfig
   app.config.from_object('DevelopmentConfig') # Load cấu hình môi trường dev
```

* Với môi trường Test

```python
   from config import TestingConfig
   app.config.from_object('TestingConfig') # Load cấu hình môi trường tét
```

* Trong cả 2 môi trường để lấy ra một gía trị ví dụ DB ta  có cùng cách làm như sau

```python
 db = app.config['DB'] #Lấy gía trị DB trong môi trường dev
```

Như vậy nhìn lại ở trên ta chỉ cần thay phần tên class thôi là có thể đổi qua lại các gía trị của các môi trường rồi. Nhìn có vẻ trong sáng phết :)

### 3. EnvVars-based

Với cách làm này python cho phép ta đọc file config vào từ biến môi trường. :) Ta mô tả bằng các bước sau.

**B1.** Tạo file .env với nội dung

```python
DEBUG = False
DB = 'mydb'
```
**B2.** Trong chương trình thay bằng load vào 1 file cấu hình có tên cố định, ta load vào file cấu hình tùy thuộc vào gía trị của biến môi trường CONF

```
app.config.from_envvar('CONF')
```

**B3.** Khi run chương trình trước tiên ta phải truyền vào gía trị của biến môi trường CONF là tên file cấu hình (ở bước 1 ta đặt là .env)
 
```python
	export CONF=.env
```

 OK như vậy trên cùng shell này ta run chương trình thôi. 

 Và đương nhiên khi cần gọi ra thông số cấu hình ta vẫn dùng cú pháp quen thuộc.

```python
 	db = app.config['DB'] #Lấy gía trị DB trong file .env
```

Nhìn chung cách làm này cũng khá là hay ho. Tùy chỉnh dễ ràng tên file cấu hình trong các môi trường dựa vào biến môi trường :)

### 4. Instance-based
Với các hình thức trên ta nhận thấy rằng toàn bộ các file cấu hình hoàn toàn nằm trong thư mục code, là một phần của project. Từ flask 0.8 có sinh ra một cái gọi là  instance folder. Để lý giải cho cái sự ra đời này người ta đưa ra vấn đề là một số cấu hình kiểu "nhạy cảm" ví như password, token api đặc biệt là trên môi trường product người ta không muốn comit lên source control (git/sv), muốn đặt hẳn ra một chỗ riêng biệt. Bài toán này được flask giải bằng cách cho phép cấu hình tách hẳn thư mục chứa file configuration ra khỏi code. Khi cần thì load vô.

Mô tả như sau.

**B1.** Khi tạo Flask object ta thêm đoạn cấu hình sau

* Nếu instance folder là đường dẫn tuyệt đối

```python
  app = Flask(__name__, instance_path='/path/to/instance/folder') # (1)
```

* Nếu  instance folder là đường dẫn tương đối thì ta phải tạo 1 thư mục **instance** (tên này là quy ước) trong thư mục app để lưu các cấu hình sau đó thêm option instance_relative_config khi tạo đối tượng Flask

```python
app = Flask(__name__, instance_relative_config=True) # (2)
```

**B2** Lúc này ta có thể yên tâm đặt các file cấu hình như sử dụng các phương thức 1, 2, 3 trên ở bất cứ đâu không nhất thiết là trong source code.

```python
app.config.from_pyfile('flask.cfg') #Load cấu hình từ file
app.config.from_object('DevelopmentConfig') # Load cấu hình từ object
```

###  Anw, Còn tôi, tôi đã chọn cái nào cho ngầu?

Mỗi cách ở trên đều có cái hay cái dở (tùy vào t/h sử dụng và oánh gía chủ quan của người dùng). Riêng cá nhân tôi, sau hồi tham khảo gg, tôi chọn kiểu mix - Tức là kết hợp các kiểu trên.

```python
 from config import DevelopmentConfig
 from config import TestingConfig
 from config import DevelopmentConfig

config = {
    "development": "DevelopmentConfig",
    "testing": "TestingConfig",
    "default": "DevelopmentConfig"
}


def configure_app(app):
    config_name = os.getenv('FLASK_CONFIGURATION', 'default')
    app.config.from_object(config[config_name]) # object-based default configuration
    app.config.from_pyfile('config.cfg', silent=True) # instance-folders configuration
```

Mô tả qua  ^^: Cơ bản cách tôi lựa chọn là tạo ra 1 map. Các Key là tên môi trường value là tên các class (object-based) tương ứng với môi trường - mục đích là để tạo alias cho ngắn. Alias này sẽ được truyền vào từ biến môi trường FLASK_CONFIGURATION lúc run app để quyết định môi trường chạy là gì (dev/test/product). Dĩ nhiên ở object-based này tô chỉ lưu các cấu hình có thể public (ko quá nhậy cảm). Kết hợp với đó tôi cũng lựa chọn kết hợp instance folder như là nơi lưu các cấu hình nhạy cảm của môi trường product không thể đặt trong source code. Cấu hình này được tách hoàn toàn ra khỏi source code.

Bài viết có vẻ khá dài, 

Anw, hy mọi người hiểu tôi viết cái quái gì :) Happy  weekend,