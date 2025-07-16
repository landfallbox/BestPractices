# Python日志

在 Python 中可以直接使用 `print` 函数在控制台中输入调试信息，这样做的好处在于简单、易用，坏处在于 `print` 函数能够打印的信息有限。最佳实践是，在较大型的项目中，或者如果希望在调试信息中加入更多信息，如事件发生时间、事件发生位置、事件的严重程度等，应当使用 Python 内嵌的另一个模块 `logging` 。

## 日志级别

Python 允许自定义日志级别，但这是不推荐，应当直接使用默认的几个日志级别：

- DEBUG：最详细的日志信息
- INFO：信息详细程度仅次于DEBUG，通常只记录关键节点信息，用于确认一切都是按照我们预期的那样进行工作
- WARNING：当某些不期望的事情发生时记录的信息（如，磁盘可用空间较低），但是此时应用程序还是正常运行的
- ERROR：由于一个更严重的问题导致某些功能不能正常运行时记录的信息
- CRITICAL：当发生严重错误，导致应用程序不能继续运行时记录的信息

一般在应用程序的配置文件中指定日志级别。指定后，应用程序会记录所有日志级别大于或等于指定日志级别的日志信息，小于该等级的日志记录将会被丢弃。

一般而言，开发应用程序或部署开发环境时，可以使用DEBUG或INFO级别的日志获取尽可能详细的日志信息来进行开发或部署调试；应用上线或部署生产环境时，应该使用WARNING或ERROR或CRITICAL级别的日志来降低机器的I/O压力和提高获取错误日志信息的效率。

## 最简单的使用

可以直接使用logging模块提供的日志方法：

```python
import logging

logging.debug("logging test")
logging.info("logging test")
logging.warning("logging test")
logging.error("logging test")
logging.critical("logging test")
```

```
WARNING:root:logging test
ERROR:root:logging test
CRITICAL:root:logging test
```

### 默认日志级别

由于logging的默认之日志级别是WARNING，所以上面的代码运行时前两行被忽略了。

### 默认日志格式

logging模块的日志格式默认是BASIC_FORMAT，其值为：

```
"%(levelname)s:%(name)s:%(message)s"
```

- levelname：日志级别
- name：日志器名称，默认是root
- message：日志内容

## 修改默认配置

logging提供的日志方法在真正输入日志信息前，都会先调用 `logging.basicConfig(**kwargs)` 方法，且不传递任何参数。在 `basicConfig()` 中就会对各个属性设置默认值。

因此，如果希望修改默认值，只需要在使用日志方法前手动调用 `basicConfig()` 方法，并传入参数替代默认值就可以了。

可以传入的参数包括：

- filename：指定日志输出目标文件的文件名，指定该设置项后日志信息就不会被输出到控制台了
- filemode：指定日志文件的打开模式，默认为'a'。需要注意的是，该选项要在filename指定时才有效
- format：指定日志格式字符串，即指定日志输出时所包含的字段信息以及它们的顺序。
- datefmt：指定日期/时间格式。需要注意的是，该选项要在format中包含时间字段%(asctime)s时才有效
- level：指定日志器的日志级别
- stream：指定日志输出目标stream，如sys.stdout、sys.stderr以及网络stream。需要说明的是，stream和filename不能同时提供，否则会引发 `ValueError` 异常
- style：Python 3.2中新添加的配置项。指定format格式字符串的风格，可取值为'%'、'{'和'$'，默认为'%'
- handlers：Python 3.3中新添加的配置项。该选项如果被指定，它应该是一个创建了多个Handler的可迭代对象，这些handler将会被添加到root logger。需要说明的是：filename、stream和handlers这三个配置项只能有一个存在，不能同时出现2个或3个，否则会引发ValueError异常。

其中最重要的参数是 format，它指定了日志的输出格式。

### format

| 字段名          | 占位符                | 说明                                                         |
| :-------------- | :-------------------- | :----------------------------------------------------------- |
| asctime         | `%(asctime)s`         | 日志事件发生的时间--人类可读时间，如：2003-07-08 16:49:45,896 |
| created         | `%(created)f`         | 日志事件发生的时间--时间戳，就是当时调用time.time()函数返回的值 |
| relativeCreated | `%(relativeCreated)d` | 日志事件发生的时间相对于logging模块加载时间的相对毫秒数（目前还不知道干嘛用的） |
| msecs           | `%(msecs)d`           | 日志事件发生事件的毫秒部分                                   |
| levelname       | `%(levelname)s`       | 该日志记录的文字形式的日志级别（'DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL'） |
| levelno         | `%(levelno)s`         | 该日志记录的数字形式的日志级别（10, 20, 30, 40, 50）         |
| name            | `%(name)s`            | 所使用的日志器名称，默认是'root'，因为默认使用的是 rootLogger |
| message         | `%(message)s`         | 日志记录的文本内容，通过 `msg % args` 计算得到的             |
| pathname        | `%(pathname)s`        | 调用日志记录函数的源码文件的全路径                           |
| filename        | `%(filename)s`        | pathname的文件名部分，包含文件后缀                           |
| module          | `%(module)s`          | filename的名称部分，不包含后缀                               |
| lineno          | `%(lineno)d`          | 调用日志记录函数的源代码所在的行号                           |
| funcName        | `%(funcName)s`        | 调用日志记录函数的函数名                                     |
| process         | `%(process)d`         | 进程ID                                                       |
| processName     | `%(processName)s`     | 进程名称，Python 3.1新增                                     |
| thread          | `%(thread)d`          | 线程ID                                                       |
| threadName      | `%(thread)s`          | 线程名称                                                     |

通过 format 可以自定义日志的输出格式：

```
import logging

LOG_FORMAT = "%(asctime)s - %(levelname)s - %(message)s"
DATE_FORMAT = "%m/%d/%Y %H:%M:%S %p"

logging.basicConfig(filename='devlog.log', level=logging.DEBUG, format=LOG_FORMAT, datefmt=DATE_FORMAT)

logging.debug("logging test")
logging.info("logging test")
logging.warning("logging test")
logging.error("logging test")
logging.critical("logging test")
```

日志将会输出到devlog.log文件中：

```
07/14/2025 15:29:27 PM - DEBUG - logging test
07/14/2025 15:29:27 PM - INFO - logging test
07/14/2025 15:29:27 PM - WARNING - logging test
07/14/2025 15:29:27 PM - ERROR - logging test
07/14/2025 15:29:27 PM - CRITICAL - logging test
```

`logging.basicConfig()` 函数是一个一次性的简单配置工具，也就是说只有在第一次调用该函数时会起作用，后续再次调用该函数时完全不会产生任何操作的，多次调用的设置并不是累加操作。

日志器（Logger）是有层级关系的，上面调用的logging模块级别的函数所使用的日志器是 `RootLogger` 类的实例，其名称为'root'，它是处于日志器层级关系最顶层的日志器，且该实例是以单例模式存在的。

如果要记录的日志中包含变量数据，可使用一个格式字符串作为这个事件的描述消息，将变量数据作为第二个参数 `*args` 的值进行传递，如: `logging.warning('%s is %d years old.', 'Tom', 10)` ，输出内容为 `WARNING:root:Tom is 10 years old.` 

`logging.debug()` `logging.info()` 等方法的定义中，除了 `msg` 和 `args` 参数外，还有一个 `**kwargs` 参数。它们支持3个关键字参数: `exc_info, stack_info, extra`：

- ***exc_info：*** 其值为布尔值，如果该参数的值设置为True，则会将异常信息添加到日志消息中。如果没有异常信息则添加None到日志信息中。
- ***stack_info：*** 其值也为布尔值，默认值为False。如果该参数的值设置为True，栈信息将会被添加到日志信息中。
- ***extra：*** 这是一个字典（dict）参数，它可以用来自定义消息格式中所包含的字段，但是它的key不能与logging模块定义的字段冲突。

```python
LOG_FORMAT = "%(asctime)s - %(levelname)s - %(user)s[%(ip)s] - %(message)s"
DATE_FORMAT = "%m/%d/%Y %H:%M:%S %p"

logging.basicConfig(format=LOG_FORMAT, datefmt=DATE_FORMAT)
logging.warning("Some one delete the log file.", exc_info=True, stack_info=True, extra={'user': 'Tom', 'ip':'47.98.53.222'})
```

```
07/14/2025 16:12:52 PM - WARNING - Tom[47.98.53.222] - Some one delete the log file.
NoneType: None
Stack (most recent call last):
  File "D:\Local_Repositories\BestPractices\Pyhton日志\ExampleCode\exp1.py", line 7, in <module>
    logging.warning("Some one delete the log file.", exc_info=True, stack_info=True,
```

### 常用日志配置

```python
LOG_FORMAT = "%(asctime)s - %(levelname)s - %(message)s"
DATE_FORMAT = "%m/%d/%Y %H:%M:%S %p"

logging.basicConfig(filename='devlog.log', level=logging.DEBUG, format=LOG_FORMAT, datefmt=DATE_FORMAT)
logging.warning("Some one delete the log file.", stack_info=True)
```

## 最佳实践

```python
import logging

# 获取一个命名日志器
logger = logging.getLogger('dev_logger')
logger.setLevel(logging.DEBUG)

# 创建一个控制台处理器
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)

# 创建一个文件处理器
file_handler = logging.FileHandler('dev.log', mode='a', encoding='utf-8')
file_handler.setLevel(logging.DEBUG)

# 设置日志格式
formatter = logging.Formatter(fmt='%(asctime)s - %(name)s - %(levelname)s - %(message)s', datefmt='%Y-%m-%d %H:%M:%S')
console_handler.setFormatter(formatter)
file_handler.setFormatter(formatter)

# 将处理器添加到日志器
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 使用日志器记录日志
logger.info("This is an info message.")
logger.debug("This is a debug message.")
```

