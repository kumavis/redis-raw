``` 

    __      ___      ___   / ( )  ___            __      ___                 
  //  ) ) //___) ) //   ) / / / ((   ) ) ____  //  ) ) //   ) ) //  / /  / / 
 //      //       //   / / / /   \ \          //      //   / / //  / /  / /  
//      ((____   ((___/ / / / //   ) )       //      ((___( ( ((__( (__/ /   

```
# RedisRaw

##a simple redis client

I wrote this because the other available nodejs redis clients where not <em>correct</em> enough for me.  
The way redis replys to pub/sub commands are a bit weird, and the other libraries did not handle them  
in what I consider to be the <em>nodejs way</em>.  (always respond to every callback)

RedisRaw just tries to be idiomatic nodejs,  
so that you can concentrate on learning redis,  
not learning bugs in the redis client.  

## create a new client

`redis-raw.Redis` takes the same arguments as [`net.createConnection`](http://nodejs.org/api/net.html#net.createConnection).  
returns a redis client, callback is optional.  


``` js
  var Redis = require('redis-raw').Redis
    , r = Redis(port, host, callback)
```

## redis.req(command, callback)

``` js
  r.req(['GET', key, value], function (err, reply) {...})
```
`command` _must_ be an array of strings.  
commands are described in the [redis documentation](redis.io/commands)

When the `reply` is a single value, `reply` will be a primitive.
when redis returns many values, `reply` will be an array.

`err` will be `null` or a `string`.

 * RedisRaw does not stringify objects for you. <em>do your own json</em>

## Pub Sub

### SUBSCRIBE & PSUBSCRIBE

when redis gets a [SUBSCRIBE]((http://redis.io/commands/subscribe) or [PSUBSCRIBE](http://redis.io/commands/psubscribe) command,  
it goes into subscribe mode and non Pub/Sub commands will callback an error.

### UNSUBSCRIBE & PUNSUBSCRIBE

calls to [UNSUBSCRIBE]((http://redis.io/commands/unsubscribe)  
[PUNSUBSCRIBE]((http://redis.io/commands/punsubscribe) will callback `(error, [events, subscriptionCount])`

where `events` is the list of events that you have unsubscribed to.  

> redis has strange behavior with the handling of these commands, and may reply many times.  
> however, RedisRaw keeps it's own count of subscriptions, and will callback only once, in the normal nodejs way.
> (it was my frustration with other redis clients on this matter that decided me to write this package)

### redis.onMessage & redis.onPMessage

capture messages that you are subscribed to by overwriting `redis.onMessage` and `redis.onPMessage`
see [http://redis.io/topics/pubsub](http://redis.io/topics/pubsub)
``` js

  var Redis = require('redis-raw').Redis
    , r = Redis(port, host, callback)

  r.req(['PSUBSCRIBE', '*'], function () {...})
  
  r.onMessage = function (event, message)
  r.onPMessage = function (event, pattern, message)

```

PSUBSCRIBE supports all [glob patterns](http://en.wikipedia.org/wiki/Glob_(programming)).

see 

## 'error', 'close', and closing the connection.

users should directly access the underling [net.Socket](http://nodejs.org/api/net.html#net.Socket)  
and add listeners and end or destroy the connection there.  
(or, let the server close the connection `r.req(['QUIT'], callback)`)

``` js
var Redis = require('redis-raw').Redis
  , r = Redis(port, host, callback)

  //listen for connection closing
  r.socket.on('close', function () {...})

  //end connection
  r.socket.end()
  //this will half close the connection, 
  //redis will still write replys to any outstanding requests
  //and then close the connection.

  //force the connection to close immediately
  r.socket.destroy()

```
