# Fast API 个人笔记

快速学习下近年现有的 Fast API 技术，仅供参考，By Mane。

更新日期：2021年6月17日

## 可选参数

使用 `name: Optional[str] = None` 指定

``` python
def say_hi(name: Optional[str] = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```

## 路径函数

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

如果是 `item_id:int`  就会自动变成了int类型

##  保存 Cookie / 读取 Cookie / 删除 Cookie

``` python
from typing import Optional
from fastapi import FastAPI
from pydantic import BaseModel
from starlette import responses

# 一定要 import 这两个类
from fastapi import Cookie,Response
app = FastAPI()

@app.get("/mane/")

# 这里要设置 Cookie(None) 不然就会被当作 输入参数
async def create_item(response:Response, testcookie:Optional[str] = Cookie(None)):
    
    # 设置 Cookie
    response.set_cookie("testcookie","acookie")
    
    # 获取 Cookie 要先声明
    get_cookie = testcookie
    print(get_cookie)
    
    # 删除 Cookie
    response.delete_cookie("testcookie") 
    
    return "ok"
```

坑爹的是 Cookie的名字要和变量名一样才会被读取到。

用 `set_cookie` 设置了之后并不会更新 `testcookie` 变量，删除 `delete_cookie` 如果删掉不存在的cookie 就会新增一个cookie值为 `""`。

## 读取 Header 内容

参考 [官方文档](https://fastapi.tiangolo.com/zh/tutorial/header-params/#_1)，值得注意的是这一句话

> 大多数标准的 headers 用 "连字符" 分隔，也称为 "减号" (`-`)。
>
> 但是像 `user-agent` 这样的变量在 Python 中是无效的。
>
> 因此，默认情况下，`Header` 将把参数名称的字符从下划线 (`_`) 转换为连字符 (`-`) 来提取并记录 headers.

## 返回错误代码

```python
from fastapi import FastAPI, HTTPException
raise HTTPException(status_code=404, detail="Mane Say ERROR")
```

## 关闭文档

```python
from fastapi import FastAPI
app = FastAPI(docs_url="/documentation",docs_url=None, redoc_url=None)
```

参考[官方连接](https://fastapi.tiangolo.com/tutorial/metadata/#openapi-url)

## 静态文件

``` python
app.mount("/html",StaticFiles(directory="static"))
```

## 启动关闭事件

```python
@app.on_event("startup")
def startup():
    print("Mane) Service Starting up ...")

@app.on_event("shutdown")
def shutdown():
    print("Mane) Service shuting down ...")
```

## 分割成小文件

`main.py`

```python
from fastapi import FastAPI
import subapi.funa as funa
import subapi.funb as funb

app = FastAPI()
app.include_router(funa.route,prefix="/funa")
app.include_router(funb.route,prefix="/funb")
```

`funa.py` / `funb.py`

```python
from fastapi import FastAPI, APIRouter
route = APIRouter()
@route.get("/hello")
async def hello():
    print("hello funa.py")
```

# 坑？

记录自己踩过的坑

## Postman 与 Fast API 的兼容

```python
from typing import Optional
from fastapi import FastAPI
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```

如果你用传统的 `form-data` 或者 `x-www-form-urlencoded` 就会报错，返回

```json
{
    "detail": [
        {
            "loc": [
                "body"
            ],
            "msg": "value is not a valid dict",
            "type": "type_error.dict"
        }
    ]
}
```

原因是 Fast API 的结构体用 json格式 装着的，但提交的POST请求不是json，所以就报错了。

**解决办法**：Postman 选 `Raw`，伪造POST请求内容是json格式就可以了，如下

``` json
{
    "name":"isname",
    "description":"fff",
    "price":1234,
    "tax":5678
}
```

