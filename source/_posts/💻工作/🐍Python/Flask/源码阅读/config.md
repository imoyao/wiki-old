---
title: Flask 源码解析：Config
toc: true
tags:
  - Flask
  - web 开发
  - Python
categories:
  - "\U0001F4BB 工作"
  - "\U0001F40DPython"
  - Flask
  - 源码阅读
date: 2019-08-26 12:27:56
---


Flask 配置导入对于其他项目的配置导入有很好的借鉴意义，所以我这里还是作为一个单独的章节进行源码学习。Flask 常用的四种方式进行项目参数的配置，如例所示:

1. 直接配置参数
```python
app.config['SECRET_KEY'] = 'YOUCANNOTGUESSME'
```
2. 从环境变量中获得配置文件名并导入配置参数
```bash
export MyAppConfig=/path/to/settings.cfg #linux
set MyAppConfig=d:\settings.cfg#不能立即生效，不建议windows下通过这种方式获得环境变量。
app.config.from_envvar('MyAppConfig')
```
3. 从对象中获得配置
```python
class Config(object):
    DEBUG = False
    TESTING = False
    DATABASE_URI = 'sqlite://:memory:'
class ProductionConfig(Config):
    DATABASE_URI = 'mysql://user@localhost/foo'
app.config.from_object(ProductionConfig)
print app.config.get('DATABASE_URI')
```
4. 从文件中获得配置参数
```plain
# default_config.py
HOST = 'localhost'
PORT = 5000
DEBUG = True

# flask中使用
app.config.from_pyfile('default_config.py')
```
Flask 已经默认自带的配置包括：
```plain
['JSON_AS_ASCII', 'USE_X_SENDFILE', 'SESSION_COOKIE_PATH', 'SESSION_COOKIE_DOMAIN', 'SESSION_COOKIE_NAME', 'SESSION_REFRESH_EACH_REQUEST', 'LOGGER_HANDLER_POLICY', 'LOGGER_NAME', 'DEBUG', 'SECRET_KEY', 'EXPLAIN_TEMPLATE_LOADING', 'MAX_CONTENT_LENGTH', 'APPLICATION_ROOT', 'SERVER_NAME', 'PREFERRED_URL_SCHEME', 'JSONIFY_PRETTYPRINT_REGULAR', 'TESTING', 'PERMANENT_SESSION_LIFETIME', 'PROPAGATE_EXCEPTIONS', 'TEMPLATES_AUTO_RELOAD', 'TRAP_BAD_REQUEST_ERRORS', 'JSON_SORT_KEYS', 'JSONIFY_MIMETYPE', 'SESSION_COOKIE_HTTPONLY', 'SEND_FILE_MAX_AGE_DEFAULT', 'PRESERVE_CONTEXT_ON_EXCEPTION', 'SESSION_COOKIE_SECURE', 'TRAP_HTTP_EXCEPTIONS']
```
其中关于 debug 这个参数要特别的进行说明，当我们设置为 app.config["DEBUG"]=True 时候，flask 服务启动后进入调试模式，在调试模式下服务器的内部错误会展示到 web 前台，举例说明:

```python
app.config["DEBUG"]=True

@app.route('/')
def hello_world():
    a=3/0
    return 'Hello World!'
```
打开页面我们会看到
![flask错误信息](/images/849997-20171011132245934-888527208.png)
除了显示错误信息以外，Flask 还支持从 web 中提供 console 进行调试（需要输入 pin 码），破解 pin 码很简单，这意味着用户可以对部署服务器执行任意的代码，所以如果 Flask 发布到生产环境，必须确保 DEBUG=False。
嗯，有空再写一篇关于 Flask 的安全篇。另外，关于如何配置 Flask 参数让网站更加安全，可以参考[这篇博文](http://www.cnblogs.com/m0m0/p/5624315.html)，写的很好。
接下来继续研究 Flask 源码中关于配置的部分。可以发现 config 是 app 的一个属性，而 app 是 Flask 类的一个示例，并且可以通过 app.config["DEBUG"]=True 来设置属性，可以大胆猜测 config 应该是一个字典类型的类属性变量，这一点在源码中验证了：
```python
#: The configuration dictionary as :class:`Config`.  This behaves
#: exactly like a regular dictionary but supports additional methods
#: to load a config from files.
self.config = self.make_config(instance_relative_config)
```
我们进一步看看 make_config 函数的定义:

```python
def make_config(self, instance_relative=False):
    """Used to create the config attribute by the Flask constructor.
    The `instance_relative` parameter is passed in from the constructor
    of Flask (there named `instance_relative_config`) and indicates if
    the config should be relative to the instance path or the root path
    of the application.

    .. versionadded:: 0.8
    """
    root_path = self.root_path
    if instance_relative:
        root_path = self.instance_path
    return self.config_class(root_path, self.default_config)

config_class = Config
```
其中有两个路径要选择其中一个作为配置导入的默认路径，这个用法在上面推荐的博文中用到过，感兴趣的看看，make_config 真正功能是返回 config_class 的函数，而这个函数直接指向 Config 类，也就是说 make_config 返回的是 Config 类的实例。似乎这里面有一些设计模式在里面，后续再研究一下。接下来是 Config 类的定义：
```python
class Config(dict):
      def __init__(self, root_path, defaults=None):
        dict.__init__(self, defaults or {})
        self.root_path = root_path
```
root_path 代表的是项目配置文件所在的目录。defaults 是 Flask 默认的参数，用的是 immutabledict 数据结构，是 dict 的子类，其中 default 中定义为：

```plain
#: Default configuration parameters.
    default_config = ImmutableDict({
        'DEBUG':                                get_debug_flag(default=False),
        'TESTING':                              False,
        'PROPAGATE_EXCEPTIONS':                None,
        'PRESERVE_CONTEXT_ON_EXCEPTION':        None,
        'SECRET_KEY':                          None,
        'PERMANENT_SESSION_LIFETIME':          timedelta(days=31),
        'USE_X_SENDFILE':                      False,
        'LOGGER_NAME':                          None,
        'LOGGER_HANDLER_POLICY':              'always',
        'SERVER_NAME':                          None,
        'APPLICATION_ROOT':                    None,
        'SESSION_COOKIE_NAME':                  'session',
        'SESSION_COOKIE_DOMAIN':                None,
        'SESSION_COOKIE_PATH':                  None,
        'SESSION_COOKIE_HTTPONLY':              True,
        'SESSION_COOKIE_SECURE':                False,
        'SESSION_REFRESH_EACH_REQUEST':        True,
        'MAX_CONTENT_LENGTH':                  None,
        'SEND_FILE_MAX_AGE_DEFAULT':            timedelta(hours=12),
        'TRAP_BAD_REQUEST_ERRORS':              False,
        'TRAP_HTTP_EXCEPTIONS':                False,
        'EXPLAIN_TEMPLATE_LOADING':            False,
        'PREFERRED_URL_SCHEME':                'http',
        'JSON_AS_ASCII':                        True,
        'JSON_SORT_KEYS':                      True,
        'JSONIFY_PRETTYPRINT_REGULAR':          True,
        'JSONIFY_MIMETYPE':                    'application/json',
        'TEMPLATES_AUTO_RELOAD':                None,
    })

```
我们再看看 Config 的三个导入函数 from_envvar,from_pyfile, from_object。from_envvar 相当于在 from_pyfile 外面包了一层壳子，从环境变量中获得，其函数注释中也提到了这一点。而 from_pyfile 最终也是调用 from_object。所以我们的重点是看 from_object 这个函数的细节。
from_pyfile 源码中有一句特别难懂，如下。config_file 是读取的文件头，file_name 是文件名称。
```python
exec(compile(config_file.read(), filename, 'exec'), d.__dict__)
```
`__dict__`是 python 的内置属性，包含了该对象（python 万事万物都是对象）的属性变量。类的实例对象的__dict__只包括类实例后的变量，而类对象本身的__dict__还包括包括一些类内置属性和类变量 clsvar 以及构造方法__init。
再理解 exec 函数，exec 语句用来执行存储在代码对象、字符串、文件中的 Python 语句，eval 语句用来计算存储在代码对象或字符串中的有效的 Python 表达式，而 compile 语句则提供了字节编码的预编译。：
```python
exec(object[, globals[, locals]]) #内置函数
```
其中参数 obejctobj 对象可以是字符串（如单一语句、语句块），文件对象，也可以是已经由 compile 预编译过的代码对象，本文就是最后一种。参数 globals 是全局命名空间，用来指定执行语句时可以访问的全局命名空间；参数 locals 是局部命名空间，用来指定执行语句时可以访问的局部作用域的命名空间。按照这个解释，上述的语句其实是转化成了这个语法：
```python
import types
var2=types.ModuleType("test")
exec("A='bb'",var2.__dict__)
```
把配置文件中定义的参数写入到了定义为 config Module 类型的变量 d 的内置属性__dict__中。
再看看 complie 函数 compile( str, file, type )，
compile 语句是从 type 类型（包括’eval’: 配合 eval 使用，’single’: 配合单一语句的 exec 使用，’exec’: 配合多语句的 exec 使用）中将 str 里面的语句创建成代码对象。file 是代码存放的地方，通常为”。compile 语句的目的是提供一次性的字节码编译，就不用在以后的每次调用中重新进行编译了。

from_object 源码中将输入的参数进行类型判断，如果是 object 类型的，则说明是通过 from_pyfile 中传过来的，只要遍历 from_pyfile 传输过来的 d 比变量的内置属性__dict__即可。如果输入的 string 类型，意味着这个是要从默认的 config.py 文件中导入，用户需要输入 app.config.from_object("config")进行明确，这时候根据 config 直接导入 config.py 配置。

具体的源码细节如下：

```python
def from_envvar(self, variable_name, silent=False):
    rv = os.environ.get(variable_name)
    if not rv:
        if silent:
            return False
        raise RuntimeError('The environment variable %r is not set '
                          'and as such configuration could not be '
                          'loaded.  Set this variable and make it '
                          'point to a configuration file' %
                          variable_name)
    return self.from_pyfile(rv, silent=silent)


def from_pyfile(self, filename, silent=False):
    filename = os.path.join(self.root_path, filename)
    d = types.ModuleType('config')
    d.__file__ = filename
    try:
        with open(filename) as config_file:
            exec (compile(config_file.read(), filename, 'exec'), d.__dict__)
    except IOError as e:
        if silent and e.errno in (errno.ENOENT, errno.EISDIR):
            return False
        e.strerror = 'Unable to load configuration file (%s)' % e.strerror
        raise
    self.from_object(d)
    return True


def from_object(self, obj):
    """Updates the values from the given object.  An object can be of one
    of the following two types:

    -  a string: in this case the object with that name will be imported
    -  an actual object reference: that object is used directly

    Objects are usually either modules or classes. :meth:`from_object`
    loads only the uppercase attributes of the module/class. A ``dict``
    object will not work with :meth:`from_object` because the keys of a
    ``dict`` are not attributes of the ``dict`` class.

    Example of module-based configuration::

        app.config.from_object('yourapplication.default_config')
        from yourapplication import default_config
        app.config.from_object(default_config)

    You should not use this function to load the actual configuration but
    rather configuration defaults.  The actual config should be loaded
    with :meth:`from_pyfile` and ideally from a location not within the
    package because the package might be installed system wide.

    See :ref:`config-dev-prod` for an example of class-based configuration
    using :meth:`from_object`.

    :param obj: an import name or object
    """
    if isinstance(obj, string_types):
        obj = import_string(obj)
    for key in dir(obj):
        if key.isupper():
            self[key] = getattr(obj, key)
```
根据源码分析，from_envvar 和 from_pyfile 两个函数的输入配置文件必须是可以执行的 py 文件，py 文件中变量名必须是大写，只有这样配置变量参数才能顺利的导入到 Flask 中。

## 参考链接
[Python flask 中的配置 - Python 初学者 - 博客园](https://www.cnblogs.com/m0m0/p/5624315.html)
