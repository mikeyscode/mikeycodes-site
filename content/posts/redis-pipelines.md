---
title: "Redis Pipelines"
date: 2018-01-23T19:14:08Z
draft: false
tags: ["technical", "redis"]
---


# Introduction to Redis Pipelines

![Redis logo](https://upload.wikimedia.org/wikipedia/en/thumb/6/6b/Redis_Logo.svg/1280px-Redis_Logo.svg.png "Redis")



## What are Pipelines?
At the very basic level Redis Pipelines allow us to send multiple commands to our server, have them executed and then read all the replies. If you are wondering why that’s different to how it normally works, think of it this way:

We have a bridge, on one side of the bridge is all the commands we want to run. Our helper memorises a single command, carries it across the bridge and executes it. He then comes back across the bridge and tells us the result of the command.

Now that’s all well and good if we have a single or even a few commands, our helper is a pretty fast runner so we get the results from our commands pretty quick, but what if we have hundreds, thousands or even millions of commands?

Well let's teach our helper about Pipelines. This time rather than memorising one command and crossing the bridge, he’s going to memorise them all first, then cross the bridge and execute them all together, one after the other. Then he’s going to return with a list of all the results when he’s done.

In Redis terms every time we crossed the bridge we had to connect to Redis again. So you can imagine how much faster only crossing the bridge once is against hundred or even a thousand times.

The keen eyed ones among you may have spotted something about Pipelines though, our helper is having to store in memory a lot of commands, as the amount of commands increases so does the load on our helper's brain. So what can we do to reduce that?

Well depending on how good your helper is and how much he can store at once we can try and limit the amount of commands he has to remember through batching. Rather than sending all our requests in one go, we can send across 250 at a time. This’ll still be faster than our original method of one at a time, but limit the amount of memory our helper needs.


## So how do you implement Pipelines?
Well it will vary depending on your implementation and language of choice but below I’ll demonstrate a simple example utilising [phpredis](https://github.com/phpredis/phpredis).

In our basic example we have a Book class which has a cache method. This method takes in an array of books that we loop through and store in redis as hashes. You will notice I haven’t implemented batching for this example, I'll leave that for you to figure out.

```php
<?php

class Book
{
    const REDIS_NAMESPACE = 'library:books';
    const DEFAULT_TTL 	  = 14400;
	
    public function cache(array $books)
    {
        $redis    = new Redis();
        $pipeline = $redis->multi(Redis::PIPELINE);

        foreach ($books as $book) {
            $bookNamespace = sprintf('%s:%s', self::REDIS_NAMESPACE, $book->id);
            $pipeline->hMSet($bookNamespace, [
                'id' 		 => $book->id,
                'title'      => $book->title,
                'returnDate' => $book->returnDate
            ]);

            $pipeline->expire($bookNamespace, self::DEFAULT_TTL);
        }

        $pipeline->exec();	
    }
}
```

So lets go over the key things we are doing here:

```php
$pipeline = $redis->multi(Redis::PIPELINE);
```
First we start out by creating our Pipeline, in phpredis to do this we just call `->multi()` and pass through the pipeline constant. This will create a handle for us that we can execute our commands on.


```
$pipeline->hMSet($bookNamespace, [
    'id'         => $book->id,
    'title'      => $book->title,
    'returnDate' => $book->returnDate
]);

$pipeline->expire($bookNamespace, self::DEFAULT_TTL);
```
Now we have our pipeline created we can throw commands at it, so that's exactly what we do. While looping through each of our books we add a `->hMSet()` and an `->expire()` command to our pipeline, this stores the commands and all the information they need to execute up until the next step.


```
$pipeline->exec();
```
Now our pipeline has a bunch of commands that we want to execute, so all we have to do is call `exec()` and it will start firing through the commands in a queue system (first in, first out).


And that is all there is to it, we do not have any commands that will return any useful information so we do not need to do anything with the result from `exec()`. If you were however retreiving rather than than setting, you would access your results like so: `$results = $pipeline->exec();`.

This is just a basic introduction to Redis Pipelines, there is a lot more to them than I've covered here so I highly encourage you read the redis.io topic to further your knowledge on them because they are a fundamental tool in day to day Redis operation: https://redis.io/topics/pipelining