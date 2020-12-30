### flask-sqlalchemy

官网上说从`flask.ext.sqlalchemy`里面 import, 我觉得应该是文档太老旧了或者是 python 版本不匹配... 但总之是需要从`flask_sqlalchemy`里面来 import, import 的东西都是一样的

#### 一个比较合理的拆包方案

官方的文档示例给的都是在同一个文件下面的 model 声明和 db 的初始化. 但是在其他文件中引用 app 会导致 import 的问题报错. (绝对不排除是因为我对 python 的 import 机制不熟悉导致... 主要这玩意儿有点违背我从 JavaScript 上面积攒下来的经验和直觉)

一个比较合理的方案是在单独的一个文件中声明 db, 并在最后的 app 文件中将 db 初始化绑定到 app 中

类似

```py
# model/binding_models.py

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy

class User(db.Model):
  ...

class Article(db.Model):
  ...

```

```py
# app.py
from models.binding_models import db

...
db.init_app(app)
...

app.app_context().push()
# 参考文档: http://www.pythondoc.com/flask-sqlalchemy/contexts.html
```

#### sqlalchemy 模型的 JSON 序列化

默认的 sqlalchemy 模型不能进行 JSON 的序列化, 会报错它并非可序列化的.

stackoverflow 上面有诸多的[答案](https://stackoverflow.com/questions/5022066/how-to-serialize-sqlalchemy-result-to-json)可以参考使用

但其中对于 python3.7+, flask1.1+的应用来说 通过添加`@dataclass`是最方便的方案

```py
from dataclasses import dataclass

@dataclass
class User(db.Model):
  id: int = db.Column(db.Integer)
  ...
```

```py
from flask import jsonify
from models import User
...
user = User(...)
...
return jsonify(user)
# { "id": 1, ...... }
```

#### 多线程

非当前线程的情况下, flask 的 app 上下文并不会共享, 因此在子线程中执行 sqlalchemy 会出错, 提示
