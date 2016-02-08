# aiohttp-json-rpc

## Installation
`pip install aiohttp-json-rpc`

## Usage
```python
import asyncio
import aiohttp
from aiohttp.web import Application, WebSocketResponse
from aiohttp_json_rpc import PublishSubscribeJsonRpc
import datetime


class MyRpc(PublishSubscribeJsonRpc):
    @asyncio.coroutine
    def ping(self, ws, params):
        """
        all rpc-methods have to be coroutines.
        """

        return 'pong'

    def _request_is_valid(self, request):
        # Yep! seems legit.
        return True

    def _msg_is_valid(self, msg):
        # Yep! seems legit.
        return True

    def _on_open(self, ws):
        # Hi, what brought you along today?
        pass

    def _on_close(self, ws):
        # Bye, it was a pleasure to serve you.
        pass

    def _on_error(self, ws, msg=None, exception=None):
        # Huh! nevermind...
        super()._on_error(ws, msg, exception)


@asyncio.coroutine
def clock(rpc):
    while True:
        rpc.notify('clock', str(datetime.datetime.now()))

        yield from asyncio.sleep(1)


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    myrpc = MyRpc()

    app = Application(loop=loop)
    app.router.add_route('*', '/', myrpc)

    handler = app.make_handler()
    server = loop.run_until_complete(
        loop.create_server(handler, '0.0.0.0', 8080))

    # create tasks
    tasks = [
        loop.create_task(clock(myrpc)),
    ]

    # serve
    try:
        print('Starting server on 0.0.0.0:8080')
        loop.run_forever()

    except KeyboardInterrupt:
        print('Stopping server')

    finally:
        loop.run_until_complete(handler.finish_connections(1.0))
        server.close()
        loop.run_until_complete(server.wait_closed())
        loop.run_until_complete(app.finish())

        for task in tasks:
            task.cancel()

        loop.run_until_complete(asyncio.wait(tasks))

    loop.close()
```

**Example JSON RPC Messages**
```
> {"jsonrpc": "2.0", "method": "list_methods", "id": 1}
< {"result": ["list_methods", "unsubscribe", "subscribe", "list_subscriptions", "ping"], "id": 1, "jsonrpc": "2.0"}

> {"jsonrpc": "2.0", "method": "ping", "id": 2}
< {"jsonrpc": "2.0", "method": "ping", "params": "pong", "id": 2}

> {"jsonrpc": "2.0", "method": "subscribe", "params": "clock", "id": 3}
< {"id": 3, "result": true, "jsonrpc": "2.0"}
< {"method": "clock", "id": null, "params": "2015-11-17 20:05:46.721194", "jsonrpc": "2.0"}
< {"method": "clock", "id": null, "params": "2015-11-17 20:05:47.722591", "jsonrpc": "2.0"}
< {"method": "clock", "id": null, "params": "2015-11-17 20:05:48.724003", "jsonrpc": "2.0"}
< {"method": "clock", "id": null, "params": "2015-11-17 20:05:49.725479", "jsonrpc": "2.0"}
< {"method": "clock", "id": null, "params": "2015-11-17 20:05:50.726987", "jsonrpc": "2.0"}
```

## RPC-Classes
1. aiohttp_json_rpc.JsonRpc
  - **Methods:** list_methods

2. aiohttp_json_rpc.PublishSubscribeJsonRpc
  - **Methods:** list_methods, subscribe, unsubscribe, list_subscriptions

3. aiohttp_json_rpc.django.DjangoAuthJsonRpc
  - **Methods:** list_methods
  - checks Django csrf-cookie

4. aiohttp_json_rpc.django.DjangoAuthPublishSubscribeJsonRpc
  - **Methods:** list_methods, subscribe, unsubscribe, list_subscriptions
  - checks Django csrf-cookie
