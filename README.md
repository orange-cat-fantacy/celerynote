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

2.
