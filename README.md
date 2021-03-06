aiogevent implements the asyncio API (PEP 3156) on top of gevent. It makes
possible to write asyncio code in a project currently written for gevent.

aiogevent allows to use greenlets in asyncio coroutines, and to use asyncio
coroutines, tasks and futures in greenlets: see ``yield_future()`` and
``wrap_greenlet()`` functions.

The main visible difference between aiogevent and trollius is the behaviour of
``run_forever()``: ``run_forever()`` blocks with trollius, whereas it runs in a
greenlet with aiogevent. It means that aiogevent event loop can run in an
greenlet while the Python main thread runs other greenlets in parallel.

* `aiogevent on Python Cheeseshop (PyPI)
  <https://pypi.python.org/pypi/aiogevent>`_
* `aiogevent at Bitbucket
  <https://bitbucket.org/haypo/aiogevent>`_
* `gevent <http://www.gevent.org/>`_
* Copyright/license: Open source, Apache 2.0. Enjoy!

See also the `aioeventlet project <http://aioeventlet.readthedocs.org/>`_.


Hello World
===========

```python
import aiogevent

def hello_world():
    print("Hello World")
    loop.stop()

asyncio.set_event_loop_policy(aiogevent.EventLoopPolicy())
loop = asyncio.get_event_loop()
loop.call_soon(hello_world)
loop.run_forever()
loop.close()
```


API
===

aiogevent specific functions:

yield_future
------------

yield_future(future, loop=None):

   Wait for a future, a task, or a coroutine object from a greenlet.

   Yield control other eligible greenlet until the future is done (finished
   successfully or failed with an exception).

   Return the result or raise the exception of the future.

   The function must not be called from the greenlet running the aiogreen
   event loop.

   Example of greenlet waiting for a trollius task. The ``progress()``
   callback is called regulary to see that the event loop in not blocked.

```python
import aiogevent
import gevent
import asyncio

def progress():
    print("computation in progress...")
    loop.call_later(0.5, progress)

async def coro_slow_sum(x, y):
    await asyncio.sleep(1.0)
    return x + y

def green_sum():
     loop.call_soon(progress)
     task = asyncio.ensure_future(coro_slow_sum(1, 2))
     value = aiogevent.yield_future(task)
     print("1 + 2 = %s" % value)
     loop.stop()

asyncio.set_event_loop_policy(aiogevent.EventLoopPolicy())
gevent.spawn(green_sum)
loop = asyncio.get_event_loop()
loop.run_forever()
loop.close()
```

   Output

```
computation in progress...
computation in progress...
computation in progress...
1 + 2 = 3
```

wrap_greenlet
-------------

wrap_greenlet(gt):

Wrap a greenlet into a Future object.

The Future object waits for the completion of a greenlet. The result or
the exception of the greenlet will be stored in the Future object.

Greenlet of greenlet and gevent modules are supported: ``gevent.greenlet``
and ``greenlet.greenlet`` objects.

The greenlet must be wrapped before its execution starts. If the
greenlet is running or already finished, an exception is raised.

For ``gevent.Greenlet``, the ``_run`` attribute must be set. For
``greenlet.greenlet``, the ``run`` attribute must be set.

Example of trollius coroutine waiting for a greenlet. The ``progress()``
callback is called regulary to see that the event loop in not blocked

```python
import aiogevent
import gevent
import trollius as asyncio
from trollius import From, Return

def progress():
    print("computation in progress...")
    loop.call_later(0.5, progress)

def slow_sum(x, y):
    gevent.sleep(1.0)
    return x + y

@asyncio.coroutine
def coro_sum():
    loop.call_soon(progress)

    gt = gevent.spawn(slow_sum, 1, 2)
    fut = aiogevent.wrap_greenlet(gt, loop=loop)

    result = yield From(fut)
    print("1 + 2 = %s" % result)

asyncio.set_event_loop_policy(aiogevent.EventLoopPolicy())
loop = asyncio.get_event_loop()
loop.run_until_complete(coro_sum())
loop.close()
```

Output

```
computation in progress...
computation in progress...
computation in progress...
1 + 2 = 3
```

Installation
============

Install aiogevent with pip
--------------------------

```
pip install aiogevent
```

Install aiogevent on Windows with pip
-------------------------------------

Procedure for Python 2.7:

* If pip is not installed yet, `install pip
  <http://www.pip-installer.org/en/latest/installing.html>`_: download
  ``get-pip.py`` and type::

  \Python27\python.exe get-pip.py

* Install aiogevent with pip::

  \Python27\python.exe -m pip install aiogevent

* pip also installs dependencies: ``eventlet`` and ``trollius``

Manual installation of aiogevent
--------------------------------

Requirements:

- Python 2.6 or 2.7
- gevent 0.13 or newer
- trollius 0.3 or newer (``pip install trollius``), but trollius 1.0 or newer
  is recommended

```
python setup.py install
```


To do
=====

* support gevent versions older than 0.13. With version older than 0.13, import
  gevent raise "ImportError: .../python2.7/site-packages/gevent/core.so:
  undefined symbol: current_base"
* support gevent monkey patching: enable py27_patch in tox.ini
* enable py33 environments in tox.ini: gevent 1.0.1 does not support Python 3,
  need a new release. Tests pass on the development (git) version of gevent.


gevent and Python 3
===================

The development version of gevent has an experimental support of Python 3.
See the `gevent issue #38: python3
<https://github.com/gevent/gevent/issues/38>`_.


Threading
=========

gevent does not support threads: the aiogevent event loop must run in the main
thread.


Changelog
=========

2014-12-18: Version 0.2
-----------------------

* Rename the ``link_future()`` function to ``yield_future()``
* Add run_aiotest.py
* tox now also runs aiotest test suite

2014-11-25: Version 0.1
-----------------------

* First public release
