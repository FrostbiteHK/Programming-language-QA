### 装饰器
[廖雪峰](https://www.liaoxuefeng.com/wiki/1016959663602400/1017451662295584)  
由于函数也是一个对象，而且函数对象可以被赋值给变量，所以，通过变量也能调用该函数。
```python
def now(content = "I' OK"):
	print('2019-7-26 ' + content)
f = now # f 指针与 now 指针指向同一片内存
# 函数对象有一个__name__属性，可以看到函数的名字：
# 此时，f.__name__ 与 now.__name__ 都是 'now'
```

有时我们需要增强 now() 函数的功能，但是又不想修改 now() 函数。这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。  
比如，我们希望在函数调用前后自动打印日志，  

```python
def log(func):
    def wrapper(*args, **kw):
        print('call %s():' % func.__name__)
        return func(*args, **kw)
    return wrapper

@log
def now(content = "I' OK"):
	print('2019-7-26 ' + content)

now()
now('Good Luck!')
# 打印
# call now():
# 2019-7-26 I' OK
# call now():
# 2019-7-26 Good Luck!
```
把 @log 放到 now() 函数的定义处，相当于执行了语句：
```python
now = log(now)
```
由于log()是一个 decorator，返回一个函数 wrapper,原来的 now() 函数仍然存在，只是现在同名的 now 函数指针指向 wrapper() 函数所在的内存，于是调用 now() 将执行新函数，即在 log() 函数中返回的 wrapper() 函数；调用 now('Good Luck!') 即相当于调用 wrapper('Good Luck!')。  
wrapper() 函数的参数定义是(*args, **kw)，因此，wrapper() 函数可以接受任意参数的调用。在 wrapper() 函数内，首先打印日志，再紧接着调用原始函数。

### 带参数的装饰器

```python
def log(text):
    def decorator(func):
        def wrapper(*args, **kw):
            print('%s %s():' % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorator

@log('execute')
def now(content = "I' OK"):
	print('2019-7-26 ' + content)

now()
now('Good Luck!')
# 打印
# execute now():
# 2019-7-26 I' OK
# execute now():
# 2019-7-26 Good Luck!
```
把 @log('execute') 放到 now() 函数的定义处，相当于执行了语句：
```python
now = log('execute')(now)
```

### 使用装饰器计算程序运行的时间
```python
import functools  # 这句不懂可以看廖雪峰装饰器这一章节的教程
import time

def metric(fn):
    @functools.wraps(fn)
	# 经过装饰之后的函数，他的 __name__ 属性会改变，对本例来说无影响，
	# 但对某些依赖函数签名的代码执行就会出错。
    def wrapper(*args, **kw):
        start = time.time()
        res = fn(*args, **kw)
        print('%s executed in %f s' % (fn.__name__, time.time()-start))
        return res
    return wrapper
# 以下是测试效果
@metric
def test1():
    time.sleep(2)
test1()  # test1 executed in 2.002045 s


@metric
def test2():
    time.sleep(3)
test2()  # test2 executed in 3.003093 s
```


