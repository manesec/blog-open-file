## SQLAlchemy 个人笔记

by Mane, 修改日期：2021年6月18日

## 在开始之前

``` python
>>> import sqlalchemy
>>> sqlalchemy.__version__  
```

检查版本号是否大于1.4

## 安装 mysqlclient

如果需要支援 Mysql 模块，就需要安装 `mysqlclient`

``` bash
sudo apt-get install libmysqlclient-dev python-dev
pip3 install mysqlclient
```

## Core VS ORM

传统比较底层的用法，缺点是很容易会有`SQL注入`和报错。

现代 `Object` 的写法，受到了 `JDB` or `OOP` 的启发，安全稳定，推荐使用。

## Connect 和 Begin 的区别

提交数据的话，如果是 `with engine.connect() as conn` 则需要调用 `Connection.commit()` 才可以提交，但如果是 `with engine.begin() as conn` 那就不需要调用 `Connection.commit()` 就可以自动提交了

> What’s “BEGIN (implicit)”?
>
> You might have noticed the log line “BEGIN (implicit)” at the start of a transaction block. “implicit” here means that SQLAlchemy **did not actually send any command** to the database; it just considers this to be the start of the DBAPI’s implicit transaction. You can register [event hooks](https://docs.sqlalchemy.org/en/14/core/events.html#core-sql-events) to intercept this event, for example.
>
> [参考官方文档](https://docs.sqlalchemy.org/en/14/tutorial/dbapi_transactions.html#tutorial-working-with-transactions)

但是提交可以用 `begin()`，查询就不可以用 `begin()` 了，因为没有和数据库连接也就不能够查询了。若要写入数据建议 `begin()` 和 `connect()` 一起用。

## Simple Example : Create Structure

```python
from sqlalchemy import MetaData
from sqlalchemy import Table,Column,Integer,String
from sqlalchemy import create_engine

myconnect = create_engine("mysql://manetest:manetest@localhost/manetest")
mymetadata = MetaData(myconnect)

usertable = Table(
    "user_account",
    mymetadata,
     Column('id', Integer, primary_key=True),
     Column('name', String(30)),
     Column('fullname', String(30))
     )

mymetadata.create_all()
mymetadata.drop_all()
```

## Simple Example : Registry

```python
from sqlalchemy import Column
from sqlalchemy import Integer,String,ForeignKey
from sqlalchemy.orm import registry
from sqlalchemy.orm import relationship

mapper_registry = registry()
Base = mapper_registry.generate_base()

class User(Base):
    __tablename__ = 'user_account'

    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String)
    
    addresses = relationship("Address", back_populates="user")

    def __repr__(self):
       return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"

class Address(Base):
    __tablename__ = 'address'

    id = Column(Integer, primary_key=True)
    email_address = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('user_account.id'))

    user = relationship("User", back_populates="addresses")

    def __repr__(self):
        return f"Address(id={self.id!r}, email_address={self.email_address!r})"

print(type(User))
print(User.__table__)
```

```python
<class 'sqlalchemy.orm.decl_api.DeclarativeMeta'>
user_account
```

## Core Select VS ORM Select

```python
from sqlalchemy import select
stmt = select(user_table).where(user_table.c.name == 'spongebob')

with engine.connect() as conn:
    for row in conn.execute(stmt):
        print(row)
		
>>> (1, 'spongebob', 'Spongebob Squarepants')
```

```python
stmt = select(User).where(User.name == 'spongebob')
with Session(engine) as session:
    for row in session.execute(stmt):
        print(row)
        
>>> (User(id=1, name='spongebob', fullname='Spongebob Squarepants'),)
```

如上得，`select` 这个函数可以支持 `Table` 类和 `Column` ，这里的 `User` 指的是一个大类。

> select() from a Table vs. ORM class
>
> While the SQL generated in these examples looks the same whether we invoke `select(user_table)` or `select(User)`, in the more general case they do not necessarily render the same thing, as an ORM-mapped class may be mapped to other kinds of “selectables” besides tables. The `select()` that’s against an ORM entity also indicates that ORM-mapped instances should be returned in a result, which is not the case when SELECTing from a [`Table`](https://docs.sqlalchemy.org/en/14/core/metadata.html#sqlalchemy.schema.Table) object.
>
> [参考官方文档](https://docs.sqlalchemy.org/en/14/tutorial/data_select.html#the-select-sql-expression-construct)

可以看到 Core 返回的是一个原始的矩阵而没有 mapping 到 ORM。

## Simple Example : 一个完整的ORM数据处理

```python
from sqlalchemy import Column,create_engine
from sqlalchemy import Integer,String
from sqlalchemy.orm import registry
from sqlalchemy.orm import Session

# 映射类的通用注册表
mapper_registry = registry()
Base = mapper_registry.generate_base()

# 定义表
class User(Base):
    __tablename__ = 'Mane_account'

    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    fullname = Column(String(30))

    def __repr__(self):
       return f"User(id={self.id!r}, name={self.name!r}, fullname={self.fullname!r})"

# 插入测试数据
squidward = User(name="squidward", fullname="Squidward Tentacles")
krabs = User(name="ehkrabs", fullname="Eugene H. Krabs")

# 创建引擎
myconnect = create_engine("mysql://manetest:manetest@localhost/manetest")

# 如果引擎（数据库）不存在就创建所有表
Base.metadata.create_all(myconnect)

# 插入数据
session = Session(myconnect)
session.add(squidward)
session.commit()
```

