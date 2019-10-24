# Python Exceptions

---
title: Python - Exceptions
date: 2018-01-10 18:41:33
categories: Backend
tags:
- Python
---

> 参考引用：《Beginning Python - From Novice to Professional Third Edition 2017》  
> 整理笔记：CK

# 简单抛出一个异常
```py
# 指明异常类
>>> raise Exception
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
Exception
# 指明异常类，外加参数
>>> raise Exception('hyperdrive overload')
Traceback (most recent call last):
  File "<stdin>", line 1, in ?
Exception: hyperdrive overload
```
**也可以自定义一个异常类**

`class SomeCustomException(Exception): pass`

# 为什么需要异常处理：
1. 为什么不用`if`语句
  
  - 用`try`包住代码段可以一次捕获多个异常，用`if`语句要一个一个写分别处理。
2. 异常处理可以让程序即使发生致命错误也不必退出：
  - 在交互式的程序里，如果发生错误可以重试
  ```py
  try:
        x = int(input('Enter the first number: '))
        y = int(input('Enter the second number: '))
        print(x / y)
  except ZeroDivisionError:
        print("The second number can't be zero!")
  ```
  Here，尽管出错了但异常被处理了，没有继续向上抛出异常，程序也没有退出。
  - 尽管如此，如果异常发生在程序内部，还是抛出异常退出比较好，出错马上附带信息地退出，比留着隐患不知道什么时候异常退出要好。
3.


# 异常传递方式

一层一层网上传，从异常出现的地方冒泡网上传，传到调用它的函数，直至传到main函数，如果都没有被处理则最终退出程序

传的过程中是包含情景信息的

# 处理方式：
**如何处理异常写在`except`中：**
## 继续抛出异常
1. `raise` 不带参数，在不知道如何处理异常的时候
2. `raise` 其他异常类，这样原来的异常就回保持在上下文中带入新的异常，成为新异常信息的一部分抛出去
  ```py
  try:
  ...     1/0
  ... except ZeroDivisionError:
  ...     raise ValueError
  ...
  Traceback (most recent call last):
    File "<stdin>", line 2, in <module>
  ZeroDivisionError: division by zero

  During handling of the above exception, another exception occurred:

  Traceback (most recent call last):
    File "<stdin>", line 4, in <module>
  ValueError
  ```
## 捕获不同异常分别处理
  ```py
  >>> try:
  ...     x = int(input('Enter the first number: '))
  ...     y = int(input('Enter the second number: '))
  ...     print(x / y)
  ... except ZeroDivisionError:
  ...     print("The second number can't be zero!")
  ... except TypeError:
  ...     print("That wasn't a number, was it?")
  ...
  Enter the first number: 1
  Enter the second number: 0
  The second number can't be zero!
  ```
+ 慢慢感受到跟`if` statements 相比的优越性了，后面还有！


## 捕获不同异常同等处理

+ 原理跟第2点差不多，只是多个异常类型写在一个tuple里，这样可以一并处理了。  
```py
try:
    x = int(input('Enter the first number: '))
    y = int(input('Enter the second number: '))
    print(x / y)
except (ZeroDivisionError, TypeError, NameError):
    print('Your numbers were bogus ...')
```

## 将捕获的异常对象存到一个变量里
+ 这样可以自己决定如何利用这个异常信息

```python
try:
    x = int(input('Enter the first number: '))
    y = int(input('Enter the second number: '))
    print(x / y)
except (ZeroDivisionError, TypeError) as e:
    print(e)
```

## 捕获所有异常
+ 虽然可以处理多个异常，但总有可能漏掉某些无法预估的异常，所以`except`后留空捕获所有异常

```python
try:
    x = int(input('Enter the first number: '))
    y = int(input('Enter the second number: '))
    print(x / y)
except:
    print('Something wrong happened ...')
```
+ ⚠️这样也有危险，因为这样会将一些本来没有预计到的异常情况，全部当作一直的异常一视同仁的处理了，例如 Ctrl-C 终止运行会被捕获并当作：'Something wrong happened ...' 处理。而用户退出并不算输入错误或者什么异常。

为了保证处理方法只针对主要的异常，最好用`except Exception as e:` 检查 e
```python
try:
    x = int(input('Enter the first number: '))
    y = int(input('Enter the second number: '))
    print(x / y)
except Exception as e:
    print('e')
```
这样就不会去处理类似`SystemExit` 和 `KeyboardInterrupt` 的异常，因为他们是 `BaseException` 的子类，`Exception` 也是其子类。

## 设定没有异常执行什么
```py
while True:
  try:
    x = int(input('Enter the first number: '))
    y = int(input('Enter the second number: '))
    value = x / y
    print('x / y is', value)
  except:
    print('Invalid input. Please try again.')
  else:
      break
```
+ 根据5点，这里捕获一切异常，因此输入x或y的过程中即使Ctrl-C也是无法退出的，会当作非法输入处理⚠️，如果这不是你希望看到的，最好用`except Exception as e:`

## 打扫卫生
+ 举了个没什么意义的例子：
```py
try:
    x = 1 / 0
except NameError:
    print("Unknown variable")
else:
    print("That went well!")
finally:
    print('Cleaning up ...')
```

# 异常处理的哲学思想
+ 求原谅比求允许容易  
比如下面的代码，对于大部分有值的情况，这段代码都要查询字典两次
```py
def describe_person(person):
    print('Description of', person['name'])
    print('Age:', person['age'])
    if 'occupation' in person:
        print('Occupation:', person['occupation'])
```
如果用异常处理，只要简单假设是有值的，只需要执行一遍，而且还有用`else` `finally`等辅助判断，非常高效
```py
def describe_person(person):
    print('Description of', person['name'])
    print('Age:', person['age'])
    try:
        print('Occupation:', person['occupation'])
    except KeyError: pass
```
因此Python中使用try/except 比 if/else 更自然。

# 警告`warnings`
警告不会退出程序
```
>>> from warnings import warn
>>> warn("I've got a bad feeling about this.")
__main__:1: UserWarning: I've got a bad feeling about this.
```
