---
layout: post
title: "Redis Cookbook"
date: 2012-03-05 16:06
comments: true
categories: Book Reading
published: false
---
## I. An Introduction to Redis ##

### When to use Redis ###

- SQL or NoSQL? 
    - relational data or not.
    - denormalize data for performance?
    - NoSQLs do not provide ACID.
    - redis: partial ACID
        - single threaded -> <u>CI</u>;
        - if `appendfsync always` -> <u>D</u>.
    - don't believe the **hype**.
    - NoSQLs diverse;
    - what your data looks like and what your usage pattern is.
    - redis is suited for: *write-heavy*, *changes often*, *data fits in redis's data structures*.

### Using Redis Data Types ###

- designing key structure:
    - be consistent when defining key space;
    - try to limit keys to a reasonable size;
    - no big performance improvements for extreamely small keys.
    - e.g.: *c:p:319:t*, *user 123*, *cache:project:319:tasks*.

- Strings, Lists, Hashes, Sets, and Sorted Sets

## II. Clients ##

- command line: `redis-cli` shipped.
- python: `redis-py`
    - `pip install redis-py`
    - `easy_install redis`

``` python
import redis
redis = redis.Redis(host='localhost', port=6379, db=0)
redis.smembers('circle:jdoe:soccer')
redis.sadd('circle:jdoe:soccer', 'users:fred')
redis.smembers('circle:jdoe:soccer')
```

- ruby: `gem install redis`
    
``` ruby
require 'rubygems'
require 'redis'
h = {
    :host => "brick5",
    :port => 6379
}
r = Redis.new h
r.smembers 'avail_server'
```
- RoR: `gem 'redis'`, `$redis = Redis.new`, `rails console`

- adding redis func to ActiveRecord models
    
``` ruby
class User < ActiveRecord::Base
    def books
        b = $redis.smembers("books:#{self.id}")
        Book.where :id => b
    end
    def addbook(book)
        $redis.sadd("books:#{self.id}", book.id)
    end
    def delbook(book)
        $redis.srem("books:#{self.id}", book.id)
    end
    def common(user)
        c = $redis.sinter("books:#{self.id}", "books:#{user.id}")
        Books.where :id => c
    end
end
```

## III. Leverage Redis ##

### key-value store ###

- build keys out of keywords separated by colons -> *maybe a good idea*;
- hash management commands: *HSET*, *HGET*, *HGETALL*, *HINCRBY*, etc;

### inspecting ###

- mysql: `SELECT * FROM table WHERE conditions`;
- redis: 
    - `KEYS *`: support some regex; scan all keys;
    - `TYPE keyname`;
    - `monitor`.

### impl OAuth model ###

- consumer keys
- consumer secrets
- request tokens
- access tokens
- nonces

        
