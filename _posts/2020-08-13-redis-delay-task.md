---
layout: post
title: 利用redis的zset 实现延时任务
tags: redis
author: neal
---
* content
{:toc}
## Redis实现延时任务

利用zset，score保存时间戳：

* **取数据:** `ZRANGEBYSCORE [key] [min] [max] `  
* **删除数据:** `ZREMRANGEBYSCORE [key][min][max]`
* **存数据:** `ZADD [key] [member] [score]`
* **查询分数:** `ZSCORE [key] [member]` 









使用springboot的StringRedisTemplate实现；
由于score是double，返回时由于时间戳过长会转成 科学计数法，需要用`BigDecimal`转化一下；
因为业务上存在更新延时任务的情况，需要在更新score时进行判断，当前值小于原值时才进行更新，
所以用lua脚本简单处理：
```java
 	@Test
    public void testScript(){
        DefaultRedisScript script = new DefaultRedisScript();
        script.setResultType(String.class);
        script.setScriptSource(new ResourceScriptSource(new ClassPathResource("zCAS.lua")));
        List<String> keys = Lists.newArrayList();
        keys.add("zts");
        stringRedisTemplate.execute(script, keys, "c","5");
        
    }
```
zCAS.lua内容：
```lua
local tmpScore = redis.call('ZSCORE',KEYS[1],ARGV[1]);
local inputScore = tonumber(ARGV[2]);
if tonumber(tmpScore)>inputScore then
    redis.call('ZADD',KEYS[1],inputScore,ARGV[1]);
end;

```