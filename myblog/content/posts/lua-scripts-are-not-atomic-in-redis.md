---
title: "Lua Scripts Are Not Atomic in Redis"
date: 2024-04-08T14:15:00
draft: false
tags: ["Redis", "Lua scripts", "Atomic operations"]
---

Redis is an in-memory data structure store, very powerful, with an excellent abstraction via its API to manipulate its data structure. What makes Redis very powerful, in my opinion, is the variety of data structures it provides and the performance of its operations. This last feature is an effect of being an in-memory store.

As with every software, Redis also has technical decisions and limitations that we need to be aware of and consider when choosing it. In this post, I will not describe a technical limitation but a poor documentation description of one feature.

I have used Redis in production for over ten years and rely on Lua scripts to run atomic operations. However, I recently discovered a bug in production, and the root cause was that Lua scripts are not atomic, despite the documentation’s assurance that they are.

The [Redis documentation](https://redis.io/docs/interact/programmability/eval-intro/), also [here in the web.archive](https://web.archive.org/web/20240406001433/https://redis.io/docs/interact/programmability/eval-intro/) for the history, mentioned that we can modify multiple keys atomically. I’ve been relying on this statement all this time and recently discovered that this is not true.


![](/lua-scripts-are-not-atomic-in-redis/redis_doc.png)
*The Redis documentation excerpt explaining Lua scripts are atomic*

# Scripts are not atomic in Redis

I have some bad news for you if, like me, you have been using Lua scripts with Redis in production to ensure atomicity.

To demonstrate that Lua scripts are not atomic, let’s create three scripts in Lua and execute them to prove this point.

The first script initializes the variables `a` and `b` with the value 42; let’s call this `init.lua`, and its implementation is below:
```Lua
redis.call('SET', 'a', '42')
redis.call('SET', 'b', '42')
```


The second script gets these variables’ values; let’s call it `get.lua` with the following implementation:
```lua
local values = {}
table.insert(values, redis.call('GET', 'a'))
table.insert(values, redis.call('GET', 'b'))
return values
```

Finally, the last script increments the variables `a` and `b`; let's call it `inc.lua`. Besides incrementing these variables, this script intentionally executes a command that fails after incrementing `a` and before incrementing `b.`

```lua
redis.call('INCR', 'a')
-- The following statement will raise an error because the variable result doesn't exist yet.
local result = result + 1
redis.call('INCR', 'b')
```

If Lua scripts are atomic, I expect the variables `a` and `b` wouldn't change after executing this last script. However, this is not the case. What happened is that the variable `b` wasn't incremented as expected, but the variable `a` was.

The code below is the call of the `get.lua` script with its result. The last two lines are the states of the variables `a` and `b` after the script `init.lua` was executed.
```code
redis-cli --eval get.lua
1) "42"
2) "42"
```

In addition, I highlighted below the error after running the script `inc.lua`.
```code
(error) ERR user_script:2: Script attempted to access nonexistent global variable 'result' script: d0e480e1285ebff8a062967eb0fa7d2ada562d97, on @user_script:2.
```

Finally, below are the states of the variables `a` and `b` after the script `inc.lua` was executed. Notice that the variable `a` has a value of 43, and the variable `b` is still equal to 42. Therefore, the script was partially executed.
```code
1) "43"
2) "42"
```

# What are the Lua script guarantees in Redis?

The Lua scripts in Redis block the entire server during its execution. So, any other requests in Redis are blocked to reading and writing variables. This is the [serializable isolation level](https://en.wikipedia.org/wiki/Isolation_\(database_systems\)#Serializable) in database transactions; however, they are not atomic. We can update some variables in the same script while leaving others unchanged.

Not being atomic is only a decision and not a problem. However, communicating this in the documentation could confuse people and create many problems.

If you are running Lua scripts in Redis to modify multiple keys and finding strange bugs in production, what I described here could be one reason.
