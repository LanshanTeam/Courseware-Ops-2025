## python web
python 的web框架有很多 常用的大家可以考虑**Django，Flask，FastAPI**  

| 框架 | 类型 | 学习曲线 | 性能 | 灵活性 | 适用规模 |
|------|------|----------|------|--------|----------|
| Django | 全栈框架 | 较陡 | 中等 | 低 | 大型项目 |
| Flask | 微框架 | 平缓 | 中等 | 高 | 中小项目 |
| FastAPI | 现代框架 | 中等 | 高 | 中等 | API服务 |  

在这里我来介绍一下**Fastapi**  

鼓励大家去[官方教程](https://fastapi.tiangolo.com/zh/python-types/)学习
## fastapi
**FastAPI** 是一个现代、快速（高性能）的 Python Web 框架，用于构建 API，基于 Python 3.6+ 类型提示。

## 一些前置知识  

### 异步函数 (Async/Await)
- 传统的同步函数
```python
import time

def make_coffee():
    time.sleep(3)  # 等待3秒，模拟煮咖啡
    return "☕ 咖啡好了"

def make_toast():
    time.sleep(2)  # 等待2秒，模拟烤面包
    return "🍞 面包好了"

def breakfast_sync():
    print(make_coffee())  # 等3秒
    print(make_toast())   # 等2秒
    print("🍽️ 早餐完成！")

# 总耗时：3 + 2 = 5秒
```  

- 异步函数  
  
```python
import asyncio
import time

async def make_coffee():
    await asyncio.sleep(3)  # 异步等待，不阻塞其他任务
    return "☕ 咖啡好了"

async def make_toast():
    await asyncio.sleep(2)
    return "🍞 面包好了"

async def breakfast_async():
    # 同时开始两个任务
    coffee_task = make_coffee()
    toast_task = make_toast()
    
    # 等待两个任务都完成
    coffee, toast = await asyncio.gather(coffee_task, toast_task)
    
    print(coffee)
    print(toast)
    print("🍽️ 早餐完成！")

# 总耗时：最长任务的时间 ≈ 3秒~
   ```
上面是一个简单的示例，在处理**io**(input&output)密集型的代码时，异步的效率会高很多，比如现在有一个需要爬取100次的程序，就可以使用异步来爬取，效率完全不能比  

而fastapi就天生支持异步，写代码就可以用异步函数来提高程序的效率    

感兴趣的可以多看看相关的教程学习一下

---
(小话:py的异步其实我很不喜欢，以前我在这里理解就花了很大功夫，写起代码来更是麻烦，最近入门了go语言，里面的异步就很好)  

### json格式

键值对数据
```python
{
  "姓名": "张三",
  "年龄": 25,
  "学生": true
}
``` 

数据类型有
```  
字符串  
数字  
布尔值  
null  
对象  
数组  	
```
### 安装
本体下载
```bash
pip install fastapi
```
你还需要一个ASGI服务器  
因为fastapi是基于ASGI的
```bash
pip install uvicorn
```

## 第一个程序 hello world

### 先创建一个**main.py**文件
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```
### 运行
```bash
uvicorn main:app --reload 
```
可以接"--host 0.0.0.0 --port 8000"参数，这里的是默认参数  
 --reload表示服务器会自动重启，如果对代码有更改的话  

访问  http://127.0.0.1:8000  可以看到  **{"message":"Hello World"}**

交互式 API 文档(由 ReDoc 提供)  
访问 http://127.0.0.1:8000/docs。

---

在上面的代码里就是一个基本的fastapi结构,下面来讲解一下 

### 创建一个fastapi实例
```python
app = FastAPI()
```
这里的变量 app 会是 
FastAPI 类的一个「实例」
这个实例将是创建你所有 API 的主要交互对象

### 创建一个路径操作
```python
@app.get("/")
```
""@"开头为一个修饰器  一个修饰器绑定一个下面的函数  
"/" 为路径  
"get"为一个http的操作  
这样的操作有
- POST  ，用于向服务器提交数据 一般为一个表单
- GET  ，用于从服务器获取（读取）数据
- PUT ，用于完整更新资源
- DELETE  用于删除服务器上的资源  
- 
更少见的还有
- OPTIONS
- HEAD
- PATCH
- TRACE
在开 API 时通常使用特定的 HTTP 方法去执行特定的行为

### 路由操作函数
```python
async def root():
    return {"message": "Hello World"}
```
这里的async为异步函数标志 (当然函数体也可以不写异步的代码)  
当访问这个路由时函数会执行   
返回的数据需要是**json**格式 在fastapi有工具会自动对数据处理

## 注释

## 声名路径参数

下面介绍几种会用到的路径参数方法  

### 单路径参数
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```
在上面的代码里面，访问时的{item_id}" 会作为"read_item"函数的参数  
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```
可以声名参数类型，如上面 "item_id: int"

### 枚举类型的路径参数
```python
from enum import Enum    
from fastapi import FastAPI    
class opser(str, Enum):    
    lkjj = 'member1'    
    szjj = 'member2'    
    ttgg = 'member3'         
app = FastAPI()
@app.get('/get_opser/{member}')  
async def read_enum(member: opser):  
    return {member: member.name}  
```
上面我们声名一个**opser**类，它继承了**str**和**eum**  
在路径访问的带有**opser**的值，我们返回它的**name**

### 路径类型的路径参数
```python
@app.get("/files/{file_path:path}")  # :path 代表任意路径  
def read_file(file_path: str):  
    return {"file_path": file_path}
```
这里的使用场景是你的传参的参数里可能含有"/"时
一个例子
```txt
GET /files/home/user/file.txt
返回: {"file_path": "home/user/file.txt"}
 ```

### 查询参数

一些概念：  

1.查询参数就是在URL之后的key-value键值对每对键值对用&分割开来。  
示例：  
http://127.0.0.1:8000/items/?page=1&limit=10  
该请求的查询参数有两个，page和limit，值分别为1，10。  

2，当访问时不属于上面的参数查询的任意一种就会被识别为查询参数，查询参数可以是非必填，也可具有默认值  

3，查询参数也是URL的一部分，所以 “本质上” 它们都是字符串。  
但当需要使用Python类型来声明query参数时(例如用int)，他们就会被转换为相应的类型并且依据这个类型来验证传入参数。

下面举例

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```
访问
```txt
http://127.0.0.1:8000/items/?skip=0&limit=10
```
这里返回的 fake_items_db 的前 10 个元素


```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id: str, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q}
    return {"item_id": item_id}
```
这是一个**路径参数**和**查询参数**的组合使用  
**{item_id}**会被识别为一个路径参数 
访问
```txt   
http://127.0.0.1:8000/items/123?q=test
```


## 请求体
### 基础概念
当你需要将数据从客户端（例如浏览器）发送给 API 时，你将其作为「请求体」发送  
请求体是客户端发送到API的数据，响应体是API发送给客户端的数据  
你的 API 几乎总是要发送响应体。但是客户端并不总是需要发送请求体  

我们使用 Pydantic 模型来声明请求体，并能够获得它们所具有的所有能力和优点。  
和查询参数一样：数据类型的属性如果不是必须的话，可以拥有一个默认值或者是可选None，否则，该属性就是必须的。

```python
from pydantic import BaseModel  
  
class Item(BaseModel):  //自定义一个pydantic模型
    name: str   
    description: str = None  
    price: float   
    tax: float = None 
```
在这里我们访问时传入的参数是
```json
{  
    'name': 'Foo',  
    'description': '可以有可以无',  
    'price': 45.2,  
    'tax': 3.5  
}
```

### 使用请求体
```python
from fastapi import FastAPI  
from pydantic import BaseModel  
  
class Item(BaseModel):  
    name: str  
    description: str = None  
    price: float  
    tax: float = None  
  
app = FastAPI()  
 
@app.post('/body/')  
def create_item(item: Item):   ## 作为一个查询参数
    return item
```
访问(需要携带请求体)  
这里浏览器不能直接访问**post**请求

可以在**http://localhost:8000/docs**里查看  
还有一些其他的方法来测试
- 使用**request**库，定义好**data**参数
- 接口测试工具 如 apipost
- curl
  
### 请求体与其他参数结合

#### 请求体 + 路径参数

```python
from fastapi import FastAPI
from pydantic import BaseModel
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}
```

#### 请求体 + 路径参数 + 查询参数
```python
from fastapi import FastAPI
from pydantic import BaseModel
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

app = FastAPI()

@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: str | None = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```
这里的参数还有很多 如 **cookie header user-agent** 我就不过多介绍  

## 安全
你也不想你的api被人随意的get吧  
这里我们就需要实现对访问者的认证

### OAuth2与JWT集成（最常用）

```python
from datetime import datetime, timedelta
from typing import Optional
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# 配置常量
SECRET_KEY = "your-secret-key-here-change-in-production"  # 实际应从环境变量读取
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# 密码哈希
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# OAuth2方案
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

# 模拟用户数据库
fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",  # "secret"
        "disabled": False,
    }
}

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

class User(BaseModel):
    username: str
    email: Optional[str] = None
    full_name: Optional[str] = None
    disabled: Optional[bool] = None

class UserInDB(User):
    hashed_password: str

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# 用户认证依赖项
async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="无效的认证凭证",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

app = FastAPI()

# 登录端点
@app.post("/token", response_model=Token)
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="用户名或密码错误",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/users/me", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_user)):
    return current_user

@app.get("/users/me/items/")
async def read_own_items(current_user: User = Depends(get_current_user)):
    return [{"item_id": 1, "owner": current_user.username}]
```

## 连接数据库

在前面**mysql**的课程里有提到**sqlalchemy**  
这里我来讲一下怎么使用**fastapi**连接数据库做一些crud  

首先你得先安装有sqlalchemy和一个数据库服务  
在python里有一个原生得sqlite数据库，这里我使用这个做演示，当然你也可以使用其他的  

### 创建数据库模型
得益于sqlalchemy，我们可以不用写sql语句，下面是一个简单的
```python
from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

## 定义一个users的表
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True)
    email = Column(String, unique=True, index=True)
```

### 连接数据库
```python
DATABASE_URL = "sqlite:///./test.db"  # 可替换为其他数据库连接字符串

engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 创建数据库表
Base.metadata.create_all(bind=engine)
```  

### 依赖注入
```python
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session

app = FastAPI()

# 依赖项：获取数据库会话
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```
### 数据库操作
```python
from fastapi import HTTPException

@app.post("/users/")
def create_user(username: str, email: str, db: Session = Depends(get_db)):
    db_user = User(username=username, email=email)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user

@app.get("/users/{user_id}")
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = db.query(User).filter(User.id == user_id).first()
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user
```


## 中间件
在FastAPI中，中间件是非常有用的组成部分，可以处理请求和响应的生命周期中的各种操作。  
中间件是一个函数，它用于在请求到达路由处理程序之前或响应返回到客户端之前进行处理。  
通过中间件，你可以实现请求日志记录、跨域请求处理、身份验证、性能监控等功能。

一个简单的演示
### 定义和使用中间件  
```python
import time
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

# 添加CORS中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 允许的前端地址
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 自定义请求日志中间件
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    # 处理请求
    response = await call_next(request)
    
    # 记录日志
    process_time = time.time() - start_time
    print(f"{request.method} {request.url.path} - {response.status_code}")
    print(f"处理时间: {process_time:.3f}秒")
    
    # 添加自定义响应头
    response.headers["X-Process-Time"] = str(process_time)
    
    return response

@app.get("/")
async def read_root():
    return {"message": "Hello, World!"}

@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```  

添加多个中间件的执行过程
```txt
请求 → 中间件1 → 中间件2 → ... → 路由处理 → ... → 中间件2 → 中间件1 → 响应
```
  
## 作业安排
作业大家就看自己情况了，最好把上面的都敲一遍，跑一边试一试，感受一下  
那么问题来了，大家的python学的怎么样呢，这里的开发还是很需要python的基础  

web其实也算一点后端开发了，里面还有很多的东西比如鉴权，数据库，中间件...大家对开发感兴趣的可以看看蓝山java和go组(他们超厉害的)  

如果你跟的上这些，那么之后就可以参加一些比赛，写一些项目了


[推荐视频](https://www.bilibili.com/video/BV1zV2QBtE39/) 

