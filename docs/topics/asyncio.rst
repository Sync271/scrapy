.. _using-asyncio:

=======
asyncio
=======

.. versionadded:: 2.0

Scrapy has partial support for :mod:`asyncio`. After you :ref:`install the
asyncio reactor <install-asyncio>`, you may use :mod:`asyncio` and
:mod:`asyncio`-powered libraries in any :doc:`coroutine <coroutines>`.

.. _install-asyncio:

Installing the asyncio reactor
==============================

To enable :mod:`asyncio` support, set the :setting:`TWISTED_REACTOR` setting to
``'twisted.internet.asyncioreactor.AsyncioSelectorReactor'``.

If you are using :class:`~scrapy.crawler.CrawlerRunner`, you also need to
install the :class:`~twisted.internet.asyncioreactor.AsyncioSelectorReactor`
reactor manually. You can do that using
:func:`~scrapy.utils.reactor.install_reactor`::

    install_reactor('twisted.internet.asyncioreactor.AsyncioSelectorReactor')

.. _using-custom-loops:

Using custom asyncio loops
==========================    

You can also use custom asyncio event loops with the asyncio reactor. Set the
:setting:`ASYNCIO_EVENT_LOOP` setting to the import path of the desired event loop class to
use it instead of the default asyncio event loop.

.. _asyncio-await-dfd:

Windows-specific notes
======================

The Windows implementation of :mod:`asyncio` can use two event loop
implementations: :class:`~asyncio.SelectorEventLoop` (default before Python
3.8, required when using Twisted) and :class:`~asyncio.ProactorEventLoop`
(default since Python 3.8, cannot work with Twisted). So on Python 3.8+ the
event loop class needs to be changed. Scrapy since VERSION does this
automatically when you change the :setting:`TWISTED_REACTOR` setting or call
:func:`~scrapy.utils.reactor.install_reactor`, but if you install the reactor
by other means or use an older Scrapy version you need to call the following
code before installing the reactor::

    import asyncio
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())

You can put this in the same function that installs the reactor, if you do that
yourself, or in some code that runs before the reactor is installed, e.g.
``settings.py``.

.. note:: Other libraries you use may require
          :class:`~asyncio.ProactorEventLoop`, e.g. because it supports
          subprocesses (this is the case with `playwright`_), so you cannot use
          them together with Scrapy on Windows (but you should be able to use
          them on WSL or native Linux).

.. _playwright: https://github.com/microsoft/playwright-python

Awaiting on Deferreds
=====================

When the asyncio reactor isn't installed, you can await on Deferreds in the
coroutines directly. When it is installed, this is not possible anymore, due to
specifics of the Scrapy coroutine integration (the coroutines are wrapped into
:class:`asyncio.Future` objects, not into
:class:`~twisted.internet.defer.Deferred` directly), and you need to wrap them into
Futures. Scrapy provides two helpers for this:

.. autofunction:: scrapy.utils.defer.deferred_to_future
.. autofunction:: scrapy.utils.defer.maybe_deferred_to_future
.. tip:: If you need to use these functions in code that aims to be compatible
         with lower versions of Scrapy that do not provide these functions,
         down to Scrapy 2.0 (earlier versions do not support
         :mod:`asyncio`), you can copy the implementation of these functions
         into your own code.
