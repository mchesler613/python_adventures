# Adventures in Python - Logging
![header](https://i.postimg.cc/sDLM50SC/Adventures-in-Python-Logging.png)
This article assumes that you are familiar with the following:

+ coding in Python 3
+ Python classes
+ Python logging library

The purpose of this article is to document how we can use the Python [logging](https://docs.python.org/3/howto/logging.html) library to record/capture/log whatever information we want in more than one destinations in the local filesystem.

At the end of this article you should be able to:

+ use the Python logging library to save information from your Python program in selected file destinations
+ log various types of information
+ configure the logging module to display certain types of information
+ configure the logging module to format the logged information
+ understand how not to use the Python logging library

# Background
As software developers, we often ourselves using the Python `print()` function for debugging and/or displaying information to ourselves and our program users. What if we want to also capture some of these information in the local filesystem to be studied later or hand off to a third-party? Instead of reinventing the wheel by creating our own library to handle logging, we can take advantage of some of the useful features in the standard Python logging package.

# Logging Levels

The Python logging system supports several [levels](https://docs.python.org/3/library/logging.html#logging-levels) of logging information -- `CRITICAL, ERROR, WARNING, INFO, DEBUG`, and `NOTSET`. For the purpose of this article, we are only interested in logging `INFO`-level data that we cherry pick from our programs.

# Logger Class

The [Logger](https://docs.python.org/3/library/logging.html#logger-objects) class is the crux of the logging library. After retrieving a `Logger` object from the logging library, via `getLogger()`, we can tell the Logger object to log information for us, via the following methods:

+ [warning()](https://docs.python.org/3/library/logging.html#logging.Logger.warning) --  logs `WARNING`-level message

+ [info()](https://docs.python.org/3/library/logging.html#logging.Logger.info) --  logs `INFO`-level message

+ [debug()](https://docs.python.org/3/library/logging.html#logging.Logger.debug) --  logs `DEBUG`-level message

+ [error()](https://docs.python.org/3/library/logging.html#logging.Logger.error) --  logs `ERROR`-level message

+ [critical()](https://docs.python.org/3/library/logging.html#logging.Logger.critical) --  logs `CRITICAL`-level message

  For example:

```bash
>>> import logging
>>> message = "Log this"
>>> print(message)
Log this
>>> logger = logging.getLogger()
>>> logger
<RootLogger root (WARNING)>   
# the default logger object logs at WARNING level
>>> logger.warning(message)
Log this
>>> logger.info(message)
>>> 
>>> # no INFO-level message is displayed on the console
```

## Root Logger

Notice that the default `logger` object that is returned from calling `logging.getLogger()` is of type `RootLogger` with a channel name of `root`  and logging-level of `WARNING`.  There can only be [one](https://github.com/python/cpython/blob/31b82abb5c545f1e0b2ebba5fa28281c2c298173/Lib/logging/__init__.py#L1945) `RootLogger` instance in an application.

> ```
> root = RootLogger(WARNING)
> ```

Because the root logger is configured for `WARNING`-level logging, it ignores the `INFO`-level message we pass to it.  

To log `INFO`-level messages, we can alter the basic configuration of the `logging` system with [basicConfig()](https://docs.python.org/3/library/logging.html#logging.basicConfig).  For example:

```python
>>> import logging
>>> logging.basicConfig(level=logging.INFO)
>>> logging.info('hello')                                                 
INFO:root:hello                                                           
>>> logging.warning('hello')                                           
WARNING:root:hello 
```

What this does is that it allows us to log both `INFO` and `WARNING` level messages. Notice that the logged messages include a label, `INFO:root:` or `WARNING:root:` before the message.  The basic configuration formats the log message as follows:

> BASIC_FORMAT = "%(levelname)s:%(name)s:%(message)s"

> Defaults to attributes `levelname`, `name` and `message` separated by colons.

We can customize the log format by passing a `format` keyword argument to `basicConfig()`. If we're only interested in logging the message as is, without any label, we can try this:

```bash
>>> import logging
>>> logging.basicConfig(format="%(message)s", level=logging.INFO)
>>> logging.info('hello')                                                 
hello                                                                      
>>> logger = logging.getLogger()
>>> logger                                                                  
<RootLogger root (INFO)>                                                   
>>> logger.info('hello')                                                    
hello 
```

## Difference between logging.info and logger.info

Both `logging.info()`and `logger.info()` in the above example were executed by the single `root` `Logger` object.  But this may not always be the case.  While`logging.info()` is always performed by the `root` logger, the logger instance returned by `logging.getLogger()` may not always be the `root` logger. How do we return a logger other than `root`?  Keep reading.

# File Handler

The prior examples show us how to log `INFO`-level messages on the console. What if we want to log these messages in a file instead?  The logging module provides us with a [FileHandler](https://docs.python.org/3/library/logging.handlers.html#filehandler) class.  Let's create a `FileHandler` object that will be used with a `Logger` object from the logging module. The following code shows us how.

```py
>>> import logging
>>> log_one = logging.getLogger('root.one')
>>> log_one
<Logger one (WARNING)>
>>> log_one.setLevel(logging.INFO)
>>> log_one
<Logger root.one (INFO)>
>>> fh_one = logging.FileHandler('log_one.txt', mode='w')
>>> fh_one
<FileHandler C:\python\logging\log_one.txt (NOTSET)>
>>> fh_one.setLevel(logging.INFO)
>>> fh_one
<FileHandler C:\python\logging\log_one.txt (INFO)>
>>> log_one.addHandler(fh_one)
>>> log_one.info('Hello One')
```

Let's see the content that is logged in the file, `log_one.txt`.

```bash
$ cat log_one.txt
Hello One
```

Let's analyze the above snippet line by line.

## logging.getLogger

By default, the `logging.getLogger()` function returns a `root` logger object if we don't supply any argument to it.  The `root` logger object as we have learned only logs `WARNING`-level messages on the console.  If we don't want `getLogger()` to return the `root` logger, we need to supply an argument, aka channel name to it.  The channel name has to be a string, in a hierachical format, e.g.  `a`, `a.b`, `a.c`, `a.b.c`, etc.  

Notice that we passed the name `root.one` to `logging.getLogger()`.  What we're doing here is that we are creating a child logger, `one`, whose parent is `root`.  The logging module will automatically create a parent logger for us, if we don't already, and associate it with the child logger.  If we're writing this code in a file, it would make sense to use `__name__`, instead of `root`, so that the parent logger will take the name of the file.

One benefit of creating child loggers is to be able to customize each one with its own logging mechanism.  When a child logger is created, it inherits the parent's logger's logging level, which is `WARNING`, by default. Hence, we need to explicitly change the logging level to `INFO` via the `setLevel()` method. 

## logging.FileHandler

Then, we create a `FileHandler` object with a filename, `log_one.txt`, and file writing mode, `w`, to overwrite the file content each time it's opened.  The default `FileHandler` has an file mode of `a`, which appends to the associated file's existing content.

>class FileHandler(StreamHandler):
>    """
>    A handler class which writes formatted logging records to disk files.
>    """
>
>​    def __init__(self, filename, mode='a', encoding=None, delay=False, errors=None):

The `FileHandler`, a child class of `StreamHandler`, subclasses from `Handler` which initializes the logging level to `NOTSET`.  

>class Handler(Filterer):
>"""
>Handler instances dispatch logging events to specific destinations.
>The base handler class. Acts as a placeholder which defines the Handler
>interface. Handlers can optionally use Formatter instances to format
>records as desired. By default, no formatter is specified; in this case,
>the 'raw' message as determined by record.message is logged.
>"""
>def __init__(self, level=NOTSET):

So, we have to explicitly set the `FileHandler`'s logging level to `INFO` via `setLevel()`.

## Logger.addHandler

We then register the `FileHandler` object with the child logger, via `addHandler()`.

Notice that we don't have to explicitly format the log record in the above example.  This is because a _default_ `Formatter` object does not format the message at all. If all we want is to log the raw message as it is, we can skip creating a custom `Formatter` object.

>[Formatter Objects](https://docs.python.org/3/library/logging.html#formatter-objects)

> [`Formatter`](https://docs.python.org/3/library/logging.html#logging.Formatter) objects have the following attributes and methods. They are responsible for converting a [`LogRecord`](https://docs.python.org/3/library/logging.html#logging.LogRecord) to (usually) a string which can be interpreted by either a human or an external system. The base [`Formatter`](https://docs.python.org/3/library/logging.html#logging.Formatter) allows a formatting string to be specified. If none is supplied, the default value of `'%(message)s'` is used, which just includes the message in the logging call.

# Demo Project

A sample demo project that uses the logging features above is available [here](https://github.com/mchesler613/logging).  In this demo, I created a `Logger` class that wraps functionality to log `INFO`-level messages into a file, using the `logging` module, a `FileHandler`class, `getLogger()` and `addHandler()` methods.  I also supplied a `Logger.log()` method as a convenient function to use by its clients.  The source for my `Logger` class is as follows.

```py
  1 import logging
  2
  3
  4 class Logger:
  5     """
  6     This class is a representation of a logging object
  7     with a custom file handler for logging INFO-level custom messages
  8     using the Python logging library
  9     """
 10
 11     def __init__(self, name: str) -> None:
 12         self.name = name
 13
 14         # retrieve a child logger from the logging library
 15         self.logger = logging.getLogger(f"{__name__}.{name}")
 16
 17         # set log level to INFO only
 18         self.logger.setLevel(logging.INFO)
 19
 20         # create a file handler with overwrite mode
 21         file_handler = logging.FileHandler(f"logs/{name}.log", mode="w")
 22
 23         # associate file handler to child logger
 24         self.logger.addHandler(file_handler)
 25
 26     def __str__(self):
 27         return f"Logger.{self.name}"
 28
 29     def log(self, message: str) -> None:
 30         """
 31         A convenient method for clients to use to log a raw message
 32         """
 33         self.logger.info(message)
```

In this demo, I created two `Logger` clients, `Busybody` and `TicTacToe`, two games that elicit responses from the user and log them.  Here is a sample log file from a tictactoe game session:

```
$ cat logs/tictactoe.log
1 2 3
4 5 6
7 8 9

You are o
1 2 3
4 o 6
7 x 9

You are o
o 2 x
4 o 6
7 x 9

You are o
o 2 x
4 o 6
7 x o

You won!
1 2 3
4 5 6
7 8 9

You are x
1 2 3
4 x 6
7 8 o

You are x
1 2 3
o x 6
x 8 o

You are x
1 2 x
o x 6
x 8 o

You won!
```

# Conclusion

The standard Python `logging` library is a flexible and versatile utility that lets a developer implement a simple file-logging mechanism with the least amount of code.  Feel free to reuse my `Logger` class in any logable application which requires:

- saving the transcript in a chatbot session
- saving data from one program to be fed to another program
- saving email drafts before sending
- saving documents in an editor
- saving application configuration or settings data
- saving game states

and much more. Let me know if you find this tutorial useful for your personal and commercial use.  Thank you for reading.
