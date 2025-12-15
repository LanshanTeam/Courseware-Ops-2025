

# Python进阶课

## pip包管理工具

### 什么是pip

pip是Python的包安装器，就像是Python的应用商店，我们可以用pip命令下载别人写的模块。

由于我们使用的都是python3，因此我们最好使用pip3来管理我们的包，因为：

- **pip**：可能指向Python 2或Python 3，取决于系统配置

- **pip3**：明确指向Python 3的包管理器

  ```shell
  #这是pip的一些基本命令
  pip3 --version                    # 查看版本
  pip3 install 包名                  # 安装包
  pip3 install 包名==1.2.3           # 安装特定版本
  pip3 install --upgrade 包名        # 升级包
  pip3 uninstall 包名               # 卸载包
  pip3 list                         # 列出已安装包
  pip3 show 包名                    # 显示包详情
  ```

###  requirements.txt文件

- 记录项目依赖

- 便于团队协作和部署

  ```bash
  # 生成requirements.txt
  pip3 freeze > requirements.txt
  
  # 从requirements.txt安装
  pip3 install -r requirements.txt
  ```

## 深入了解模块

### 什么是模块？

#### 定义

模块就是一个.py文件，里面包含了你想要重复使用的代码：函数、类、变量等等

如果打个比方的话：

- 模块 ——> 工具箱
- 函数/类 ——> 工具（锤子、螺丝刀）
- 导入模块 ——> 打开工具箱

#### 为什么需要模块化

1. **代码重用**：一次编写，多次使用
2. **组织代码**：分门别类，结构清晰
3. **避免命名冲突**：不同模块可以有相同名字的函数
4. **共享代码**：可以分享给他人使用

### 创建并导入一个简单的模块

这只是一个简单的示例，模块的文件内容如下：

```python
# calculator.py
def add(x, y):
    return x + y

def subtract(x, y):
    return x - y

def multiply(x, y):
    return x * y

def divide(x, y):
    if y != 0:
        return x / y
    else:
        return "除数不能为零"
```

现在我们来看看导入它有几种方式：

```python
# 方法1：导入整个模块
import calculator
result = calculator.add(5, 3)

# 方法2：导入特定函数
from calculator import add, multiply
result = add(5, 3)

# 方法3：导入所有函数（不推荐）
from calculator import *
result = add(5, 3)

# 方法4：给模块起别名
import calculator as calc
result = calc.add(5, 3)
```

注意，这种导入方式是在两个文件都是在同一文件夹下的情况才能进行

### 常用内置模块

#### math模块 - 数学运算

```python
import math

# 常用函数
print(math.sqrt(16))      # 平方根: 4.0
print(math.pow(2, 3))     # 幂运算: 8.0
print(math.pi)           # π: 3.141592653589793
print(math.ceil(4.2))    # 向上取整: 5
print(math.floor(4.8))   # 向下取整: 4
print(math.factorial(5)) # 阶乘: 120
```

#### datetime模块 - 日期和时间

```python
import datetime

now = datetime.datetime.now()
print("当前时间：", now)
print("年份：", now.year)
print("今天是星期：", now.strftime("%A"))

# 计算7天后的日期
future = now + datetime.timedelta(days=7)
print("一周后的日期：", future.date())
```

#### random模块 - 随机数生成

```python
import random

print(random.randint(1, 10))      # 1到10之间的随机整数

print(random.random())            # 0到1之间的随机浮点数

colors = ['红', '绿', '蓝', '黄']
print(random.choice(colors))      # 随机选择一个元素

# 随机打乱
cards = ['A', 'K', 'Q', 'J', '10']
random.shuffle(cards)
print(cards)                      # 打乱后的列表

# 随机抽样
print(random.sample(range(100), 5))  # 从0-99中随机选5个不重复的数
```

#### os模块 - 操作系统交互

```python
import os

# 获取当前工作目录
print(os.getcwd())

# 列出目录内容
print(os.listdir('.'))

# 创建目录
os.makedirs('new_folder/sub_folder', exist_ok=True)

# 路径拼接（跨平台）
file_path = os.path.join('folder', 'subfolder', 'file.txt')

# 检查文件/目录是否存在
print(os.path.exists('file.txt'))

# 获取环境变量
print(os.environ.get('PATH'))
```

### 常用第三方模块

第三方模块需要自行用pip3 install [模块名]安装才能使用

#### requests - HTTP请求库

```python
import requests

# 发送GET请求
response = requests.get('https://api.github.com')
print(f"状态码: {response.status_code}")
print(f"响应内容: {response.text[:100]}...")

# 带参数的请求
params = {'q': 'python', 'page': 1}
response = requests.get('https://api.github.com/search/repositories', params=params)

# 发送POST请求（模拟登录）
data = {'username': 'user', 'password': 'pass'}
response = requests.post('https://httpbin.org/post', data=data)

# 下载文件
url = 'https://example.com/image.jpg'
response = requests.get(url)
with open('image.jpg', 'wb') as f:
    f.write(response.content)
```

#### pandas - 数据分析库

```python
import pandas as pd

# 创建DataFrame
data = {
    '姓名': ['张三', '李四', '王五'],
    '年龄': [25, 30, 35],
    '城市': ['北京', '上海', '广州']
}
df = pd.DataFrame(data)
print(df)

# 基本操作
print(df.head())        # 查看前几行
print(df.describe())    # 统计描述
print(df['年龄'].mean()) # 计算平均值

# 数据筛选
young_people = df[df['年龄'] < 30]
print(young_people)

# 读取CSV文件
# df = pd.read_csv('data.csv')
# 保存为CSV
# df.to_csv('output.csv', index=False)
```

### 示例

#### 用datatime模块实现生日倒计时提醒

```python
import datetime

def birthday_countdown(name, birthday):
    today = datetime.date.today()
    next_birthday = datetime.date(today.year, birthday.month, birthday.day)
    
    if next_birthday < today:
        next_birthday = datetime.date(today.year + 1, birthday.month, birthday.day)
    
    days_left = (next_birthday - today).days
    return f"{name}的生日还有{days_left}天"

# 测试
birthday = datetime.date(2006, 4, 28)
print(birthday_countdown("XX", birthday))
```

### 模块的组织：包（Package）

#### 什么是包

包是包含多个模块的文件夹，大体长这样：

```text
my_package/
├── __init__.py
├── module1.py
├── module2.py
└── sub_package/
    ├── __init__.py
    └── module3.py
```

这个结构如果形象化类比就是：

```text
图书馆（项目）
├── 自然科学区（包）
│   ├── 数学类（子包）
│   │   ├── 代数.py（模块）
│   │   └── 几何.py（模块）
│   └── 物理类（子包）
│       ├── 力学.py（模块）
│       └── 电磁学.py（模块）
└── 人文科学区（另一个包）
```

为什么我们要使用包呢？因为当你有多个模块时，这里有两种做法

这是反面教材：

```
project/
├── user_management.py
├── order_processing.py
├── payment_gateway.py
├── inventory_control.py
├── shipping_logistics.py
├── report_generator.py
├── data_analyzer.py
├── email_sender.py
├── sms_notifier.py
└── ... 
```

这是推荐做法

```
project/
├── auth/          # 认证相关
├── orders/        # 订单处理
├── payments/      # 支付
├── inventory/     # 库存
├── shipping/      # 物流
├── reports/       # 报表
├── analytics/     # 分析
└── notifications/ # 通知
```

这样显然更加一目了然，易于管理。

#### 创建我们的第一个包

##### 结构

```
my_first_package/
├── __init__.py    # 这个东西必须要有，是包的身份证
├── math_tools.py
├── string_utils.py
└── data_formatters.py
```

##### __init__.py文件

这个文件的作用在于告诉python：这是一个包，不是一个普通的文件夹

此文件可以留空，也可以进行一些初始化导入，比如：

```python
print("my_first_package 已被导入")

# 定义包的版本
__version__ = "1.0.0"

# 可以在这里导入包内的模块，方便用户使用
from .math_tools import add, multiply
from .string_utils import reverse_string
```

##### 创建模块文件

math_tools.py

```python
def add(a, b):
    """加法"""
    return a + b

def multiply(a, b):
    """乘法"""
    return a * b

def is_prime(n):
    """判断质数"""
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True
```

string_utils.py

```python
def reverse_string(s):
    """反转字符串"""
    return s[::-1]

def count_vowels(s):
    """统计元音字母"""
    vowels = "aeiouAEIOU"
    count = 0
    for char in s:
        if char in vowels:
            count += 1
    return count
```

#### 使用自己创建的包

##### 相对路径导入（包在同一目录）

main.py 与 my_first_package 在同一目录

```python
import my_first_package

print(my_first_package.__version__)
print(my_first_package.add(5, 3))  # 因为我们在__init__.py中导入了add
```

##### **导入包中的特定模块**

```python
from my_first_package import math_tools

print(math_tools.is_prime(17))  # True
print(math_tools.is_prime(15))  # False
```

##### **导入模块中的特定函数**

```python
from my_first_package.string_utils import reverse_string, count_vowels

print(reverse_string("Hello"))  # olleH
print(count_vowels("Python"))   # 1
```

#### 讲解多层包结构

这里我们以电商系统的包组织为例：

```
ecommerce_system/          # 项目根目录
├── main.py               # 主程序
└── ecommerce/            # 主包
    ├── __init__.py
    ├── constants.py      # 常量定义
    ├── models/           # 数据模型子包
    │   ├── __init__.py
    │   ├── user.py
    │   ├── product.py
    │   └── order.py
    ├── services/         # 服务层子包
    │   ├── __init__.py
    │   ├── payment.py
    │   ├── shipping.py
    │   └── notification.py
    └── utils/            # 工具子包
        ├── __init__.py
        ├── validators.py
        ├── formatters.py
        └── helpers.py
```

##### 多层包的模块导入

###### **情况1：同一包内的模块相互导入**

```python
# ecommerce/models/user.py
from .product import Product  # 从同级模块导入
from ..constants import MAX_USERS  # 从上级包导入
from ..utils.validators import validate_email  # 从兄弟包导入

class User:
    def __init__(self, name, email):
        if validate_email(email):
            self.name = name
            self.email = email
        else:
            raise ValueError("无效的邮箱地址")
```

###### **情况2：不同包间的导入**

```python
# ecommerce/services/payment.py
from ..models.order import Order  # 相对导入
from ..utils.formatters import format_currency  # 相对导入

def process_payment(order_id, amount):
    order = Order.get_by_id(order_id)
    formatted_amount = format_currency(amount)
    print(f"处理订单 {order_id} 的支付: {formatted_amount}")
```

##### 绝对导入与相对导入

###### 绝对导入（推荐）

```python
# 从项目根目录开始的完整路径
from ecommerce.models.user import User
from ecommerce.services.payment import process_payment
```

###### 相对导入（包内部使用）

```python
# 在当前包内的相对位置
from .user import User  # 同一目录
from ..services import payment  # 上级目录
from ...other_package import module  # 上上级目录
```

#### __init__.py的妙用

__init__.py可以被精心“设计”

```python
# ecommerce/__init__.py

# 1. 定义包的基本信息
__version__ = "2.1.0"
__author__ = "Lanshan运维Python教学"
__description__ = "一个电商系统包"

# 2. 导入常用功能，简化用户导入
from .models.user import User
from .models.product import Product
from .models.order import Order
from .services.payment import process_payment
from .services.shipping import calculate_shipping

# 3. 定义包的快捷函数
def create_user(name, email):
    """创建用户的快捷方式"""
    from .models.user import User
    return User(name, email)

def get_all_products():
    """获取所有产品的快捷方式"""
    from .models.product import Product
    return Product.get_all()

# 4. 包级别的配置
API_KEY = None

def configure(api_key):
    """配置包的全局设置"""
    global API_KEY
    API_KEY = api_key
    print("电商包配置完成")
```

使用优化后的包：

```python
# main.py - 使用体验大大提升
import ecommerce

# 基本信息
print(f"使用 {ecommerce.__description__} 版本 {ecommerce.__version__}")

# 配置包
ecommerce.configure(api_key="my_secret_key")

# 直接使用，不需要记住复杂路径
user = ecommerce.create_user("张三", "zhangsan@example.com")
products = ecommerce.get_all_products()
order = ecommerce.Order(user, products[0])

# 简化操作
ecommerce.process_payment(order.id, 99.99)
```

#### 项目讲解：学校管理系统包

##### 项目要求

创建一个school_management包，结构如下：

```
school_management/
├── __init__.py
├── models/
│   ├── __init__.py
│   ├── student.py
│   ├── teacher.py
│   └── course.py
├── services/
│   ├── __init__.py
│   ├── enrollment.py
│   ├── grading.py
│   └── attendance.py
└── utils/
    ├── __init__.py
    ├── validators.py
    └── reports.py
```

##### 步骤一：**创建基本结构**

```python
# school_management/models/student.py
class Student:
    def __init__(self, student_id, name, grade):
        self.student_id = student_id
        self.name = name
        self.grade = grade
        self.courses = []
    
    def enroll_course(self, course):
        self.courses.append(course)
        return f"{self.name} 已注册 {course.name}"
```

##### **步骤2：设计包接口**

```python
# school_management/__init__.py
# 暴露主要功能给用户
from .models.student import Student
from .models.teacher import Teacher
from .models.course import Course
from .services.enrollment import enroll_student
from .services.grading import calculate_gpa
from .utils.reports import generate_report_card
```

##### **步骤3：使用包**

```python
# main.py
import school_management as sm

# 创建对象
student1 = sm.Student("S001", "李明", 10)
teacher1 = sm.Teacher("T001", "王老师", "数学")
math_course = sm.Course("M101", "高等数学", teacher1)

# 使用包的功能
sm.enroll_student(student1, math_course)
gpa = sm.calculate_gpa(student1)
report = sm.generate_report_card(student1)
```

##### 包的分发与安装

###### 创建可安装的包

```python
# 在项目根目录创建 setup.py
from setuptools import setup, find_packages

setup(
    name="school_management",
    version="1.0.0",
    description="一个学校管理系统包",
    author="你的名字",
    packages=find_packages(),  # 自动找到所有包
    install_requires=[],  # 依赖的其他包
    python_requires=">=3.6",
)
```

###### 安装自己的包

```python
# 在项目目录中运行
pip install -e .

# 之后就可以像使用标准库一样使用自己的包了
```

#### 包的设计原则

1. **单一职责**：每个包/模块只做一件事
2. **层次清晰**：合理的目录结构
3. **接口简洁**：通过 `__init__.py` 暴露主要功能
4. **避免循环**：注意导入关系

#### 包命名规范

```python
# 好的命名
school_management  # 小写字母，下划线分隔
data_processing
web_scraper

# 避免的命名
SchoolManagement  # 不要用大写开头
my-package       # 不要用连字符
123package       # 不要以数字开头
```



## 小项目：学生成绩管理系统

项目结构

```
project/
├── main.py              # 主程序
├── student.py           # 学生类模块
├── grade_calculator.py  # 成绩计算模块
└── file_manager.py      # 文件管理模块
```

```python
#student.py
class Student:
    def __init__(self, name, student_id):
        self.name = name
        self.student_id = student_id
        self.grades = {}
    
    def add_grade(self, subject, grade):
        self.grades[subject] = grade
    
    def get_average(self):
        # 计算平均分
        pass
```

main.py框架（可以自己改）

```python
from student import Student
import grade_calculator as gc
import file_manager as fm

def main():
    # 创建学生对象
    # 添加成绩
    # 计算统计信息
    # 保存到文件
    pass

if __name__ == "__main__":
    main()
```

## 学习资源推荐

1. **官方文档**：
   - Python标准库：[docs.python.org/3/library/](https://docs.python.org/3/library/)
   - PyPI：[pypi.org](https://pypi.org/)
2. **第三方库推荐**：
   - 数据处理：pandas, numpy
   - 网页开发：Django, Flask
   - 数据可视化：matplotlib, seaborn
   - 机器学习：scikit-learn, tensorflow
3. **学习平台**：
   - 菜鸟教程
   - 慕课网
   - Codecademy

## 作业

作业可以选择性完成，尽力而为，可以用ai工具辅助

level0：创建一个模块（可以自己决定想要什么，比如计算类的模块等等），并调用它

level1：复现示例：实现生日倒计时提醒

level2：运用模块，实现天气查询工具，不要求代码有多复杂，只要能够用上模块就行

level3：完成小项目：学生成绩管理系统
