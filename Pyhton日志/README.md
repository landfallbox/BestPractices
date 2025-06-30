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

logging提供的日志方法在真正输入日志信息前，都会先调用`logging.basicConfig(**kwargs)`方法，且不传递任何参数。在`basicConfig()`中就会对各个属性设置默认值。

因此，如果希望修改默认值，只需要在使用日志方法前手动调用`basicConfig()`方法，并传入参数替代默认值就可以了。

可以传入的参数包括：

- filename：指定日志输出目标文件的文件名，指定该设置项后日志信息就不会被输出到控制台了
  filemode：指定日志文件的打开模式，默认为'a'。需要注意的是，该选项要在filename指定时才有效
  format：指定日志格式字符串，即指定日志输出时所包含的字段信息以及它们的顺序。
  datefmt：指定日期/时间格式。需要注意的是，该选项要在format中包含时间字段%(asctime)s时才有效
  level：指定日志器的日志级别
  stream：指定日志输出目标stream，如sys.stdout、sys.stderr以及网络stream。需要说明的是，stream和filename不能同时提供，否则会引发 `ValueError`异常
  style：Python 3.2中新添加的配置项。指定format格式字符串的风格，可取值为'%'、'{'和'$'，默认为'%'
  handlers：Python 3.3中新添加的配置项。该选项如果被指定，它应该是一个创建了多个Handler的可迭代对象，这些handler将会被添加到root logger。需要说明的是：filename、stream和handlers这三个配置项只能有一个存在，不能同时出现2个或3个，否则会引发ValueError异常。