# celerynote
与集成框架使用，celery是异步任务队列，有三个核心组件
*celery客户端：发布后台zhuo'yei
*celeryworks：运行后台作业进程。 
*消息代理：客户端通过消息队列和works通信，支持多种方式实现队列。常用rabbitMQ和Redis

安装（redis，rabbitmq，librabbitmq[C++实现]，magpack，gevevt）省略

初步使用：
redis做结果缓存，rabbitmq做任务队列
1.创建发送异步任务
#初始化
from celery import Celery
app = Celery('task',broker='amqp://username@ip:port/varhost',backend='redis://username:passwd@ip:6390/db')
@app.task
def add(x+y):
  return x+y

if __name__ == '__main__':
  result = add.delay(30,42)
  
  
过程描述：app.task装饰addhanshu成一个Task实例（多个装饰器时，保证app.task在最外层），add.delay将task序列化后，通过librabbimq库fangfa将任务发送到rabbitmq；

该过程创建了一个名为celery的exchange交换机，类型为direct；创建一个名为celery的queue，队列和交换机用路由键celery绑定

打开rabbimq，消息已经在celery队列中；

如果使用redis作为任务队列中间人，在redis中存在两个建celery和kombu.binding.celery(表示一名为celery的任务队列)，而键celery为默认队列中的任务列表，使用list类型



2.开启work执行任务
在项目目录执行以下任务
celery -A app.celery_task.celery worker -Q queue --loglever=info
# -A参数指定创建的celery对象的位置，该app.celery_tasks.celery指的是app包下面的celery_tasks.py模块的celery实例，注意一定是初始化后的实例，后面加worker表示该实例就是任务执行者；
# -Q参数指的是该worker接收指定的队列的任务，这是为了当多个队列有不同的任务时可以独立；如果不设会接收所有的队列的任务；
# -l参数指定worker输出的日志级别；

任务完毕后存在Redis中（是一个string类型的键值数），默认24小时失效

分析序列化的消息
add.delay将Task实例序列化后发送到rabbitmq（{"body": "gAJ9cQAoWAQAAAB0YXNrcQFYGAAAAHRlc3RfY2VsZXJ5LmFkZF90b2dldGhlcnECWAIAAABpZHEDWCQAAAA2NmQ1YTg2Yi0xZDM5LTRjODgtYmM5OC0yYzE4YjJjOThhMjFxBFgEAAAAYXJnc3EFSwlLKoZxBlgGAAAAa3dhcmdzcQd9cQhYBwAAAHJldHJpZXNxCUsAWAMAAABldGFxCk5YBwAAAGV4cGlyZXNxC05YAwAAAHV0Y3EMiFgJAAAAY2FsbGJhY2tzcQ1OWAgAAABlcnJiYWNrc3EOTlgJAAAAdGltZWxpbWl0cQ9OToZxEFgHAAAAdGFza3NldHERTlgFAAAAY2hvcmRxEk51Lg==",  
# body是序列化后使用base64编码的信息，包括具体的任务参数，其中包括了需要执行的方法、参数和一些任务基本信息
"content-encoding": "binary", # 序列化数据的编码方式
"content-type": "application/x-python-serialize",  # 任务数据的序列化方式，默认使用python内置的序列化模块pickle
"headers": {}, 
"properties": 
        {"reply_to": "b7580727-07e5-307b-b1d0-4b731a796652",       # 结果的唯一id
        "correlation_id": "66d5a86b-1d39-4c88-bc98-2c18b2c98a21",  # 任务的唯一id
        "delivery_mode": 2, 
        "delivery_info": {"priority": 0, "exchange": "celery", "routing_key": "celery"},  # 指定交换机名称，路由键，属性
        "body_encoding": "base64", # body的编码方式
        "delivery_tag": "bfcfe35d-b65b-4088-bcb5-7a1bb8c9afd9"}}）
        
        
        
序列化之后反序列化

import pickle
import base64
result = base64.b64decode('')
print(pickle.load(result))
#结果
{
    'task': 'test_celery.add_together',  # 需要执行的任务
    'id': '66d5a86b-1d39-4c88-bc98-2c18b2c98a21',  # 任务的唯一id
    'args': (9, 42),   # 任务的参数
    'kwargs': {},      
    'retries': 0, 
    'eta': None, 
    'expires': None, # 任务失效时间
    'utc': True, 
    'callbacks': None, # 完成后的回调
    'errbacks': None,  # 任务失败后的回调
    'timelimit': (None, None), # 超时时间
    'taskset': None, 
    'chord': None
}

我们可以看到body里面有我们需要执行的函数的一切信息，celery的worker接收到消息后就会反序列化body数据，执行相应的方法。

常见的数据序列化方式
binary: 二进制序列化方式；python的pickle默认的序列化方法；
json:json 支持多种语言, 可用于跨语言方案，但好像不支持自定义的类对象；
XML:类似标签语言；
msgpack:二进制的类 json 序列化方案, 但比 json 的数据结构更小, 更快；
yaml:yaml 表达能力更强, 支持的数据类型较 json 多, 但是 python 客户端的性能不如 json
