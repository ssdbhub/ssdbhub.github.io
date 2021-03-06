
We provide [SSDB](http://ssdb.io) as a service.

SSDB is a high performance NoSQL database.

SSDBHub provides one click solution for provisioning of highly available SSDB cluster, 
which may be accessed directly via SSDB or Redis clients.

SSDB clients are available in many languages.
Below there are basic code samples of how to connect to database, 
authenticate the connection and start to send commands 
to SSDB using the following languages: 

C++, 
C#, 
[Java](#using-ssdb-with-different-languages-using-with-java), 
[Python](#using-ssdb-with-different-languages-using-with-python), 
[Node.js](#using-ssdb-with-different-languages-using-with-node), 
[Ruby](#using-ssdb-with-different-languages-using-with-ruby), 
[PHP](#using-ssdb-with-different-languages-using-with-php),
[Go](#using-ssdb-with-different-languages-using-with-go).

As mentioned above, it's possible to communicate with SSDB using Redis clients.
Below there is a [list of supported Redis commands](#Key-Value) and how they map to SSDB.

# Heroku Platform

* **Warning!** Heroku SSDBHub add-on is currently at beta stage, therefore there might be changes to this document.

## Provisioning add-on

SSDBHub plugin can be attached to a Heroku application via the CLI:

[Here](https://elements.heroku.com/addons/ssdb) the list of all available plans.

```bash
$ heroku addons:create ssdb
-----> Adding ssdb to sharp-mountain-4005... done, v18 (free)
```

Once SSDB is added to your app, the following variables will be added to environment 
configuration:
 
```bash
SSDB_HOST
SSDB_PORT
SSDB_URL
SSDB_PASSWORD
```
   
These should be used to access the newly provisioned SSDB instance. 
Configuration variables can be confirmed using the heroku `config:get` command.

```bash
$ heroku config:get SSDB_HOST
172.217.18.14

$ heroku config:get SSDB_PORT
32612

$ heroku config:get SSDB_URL
http://172.217.18.14:32612

$ heroku config:get SSDB_PASSWORD
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoZWxsbyI6InRlc3QifQ..
```

After installing SSDBHub the application should be configured to fully integrate with the add-on.
Please note that SSDB_URL is only the combination of SSDB_HOST and SSDB_PORT.
Some clients will require both of them separate, others will use full url as a connection string.

## Local setup

### Environment setup

After provisioning the add-on it's necessary to locally replicate the 
config vars so your development environment operate against the service.

Use the Heroku Local command-line tool to configure, run and manage process 
types specified in your app's [Procfile](https://devcenter.heroku.com/articles/procfile).
Heroku Local reads configuration variables from a `.env` file. 
To view all of your app's config vars, type `heroku config`. 
Use the following command for each value you want to add to your `.env` file.

```bash
$ heroku config:get SSDB_HOST -s  >> .env
$ heroku config:get SSDB_PORT -s  >> .env
$ heroku config:get SSDB_PASSWORD -s  >> .env
$ heroku config:get SSDB_URL -s >> .env
```

* Credentials and other sensitive configuration values should not be committed to source-control.
In Git exclude the `.env` file with: `echo .env >> .gitignore`.

For more information, see [Heroku Local](https://devcenter.heroku.com/articles/heroku-local) article.

# Using SSDB with Different Languages

As already mentioned above SSDB has many client libraries in different programming languages. 
Below are couple of examples on how you can apply them. 

## Using with Java

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
official ★ | ideawu | [Repository](https://github.com/ssdb/javassdb) | This is the official client
ssdb4j | nutzam | [Repository](https://github.com/nutzam/ssdb4j) | Yet another SSDB client for Java
another ssdb4j | jbakwd | [Repository](http://git.oschina.net/jbakwd/ssdbj) | 	
hydrogen-ssdb | yiding-he | [Repository](https://github.com/yiding-he/hydrogen-ssdb) | Supports client-side load-balance

The following example uses official Java client, which needs to 
be imported within the application.

```java
import com.udpwork.ssdb.*;
```

Then the client library should be created using environment variables, which were discussed 
previously.

```java
String host = System.getenv("SSDB_HOST");
String port = System.getenv("SSDB_PORT");
SSDB ssdb = new SSDB(host, Integer.parseInt(port));
```

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```java
String pass = System.getenv("SSDB_PASSWORD");
Response resp = ssdb.request("auth", pass);
if(!resp.ok()){
	resp.exception();
}
```

Now when client is authenticated it's ready to be used

```java
ssdb.set("mykey", "myvalue");
byte[] val = ssdb.get("myvalue");
```


## Using with Python

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
built-in ★ | ideawu | [Repository](https://github.com/ideawu/ssdb/tree/master/api/python) | This is the official client
pyssdb | ifduyue | [Repository](https://github.com/ifduyue/pyssdb) | A SSDB Client Library for Python
ssdb-py | wrongwaycn | [Repository](https://github.com/wrongwaycn/ssdb-py) | SSDB Python Client like Redis-Py
ssdb.py | hit9 | [Repository](https://github.com/hit9/ssdb.py) | SSDB Python Client Library by hit9


For this example the ssdb-py client is use.
Before using the package it should be installed as follows:

```bash
$ pip install ssdb
```

Application will need to import the library

```python
import ssdb
```

Then the client should be created using environment variables.

```python
import os

host = os.env["SSDB_HOST"]
port = os.env["SSDB_PORT"]
ssdb = ssdb.SSDB(host, port)
```

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```python 
pass = os.env["SSDB_PASSWORD"];
resp = ssdb.execute_command("auth", pass);
if resp[0] != "ok":
    raise Exception("Invalid Password")
```

Now when client is authenticated it's ready to be used

```python
ssdb.set("mykey", "myvalue");
res = ssdb.get("mykey");
```

## Using with Node

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
official ★ | ideawu | [Repository](https://github.com/ssdb/nodessdb) | This is the official client
node-ssdb by @hit9 | hit9 | [Repository](https://github.com/eleme/node-ssdb) | node-ssdb by @hit9

For this example the node-ssdb library is use.
Before using the package it should be installed 

```bash
$ npm install ssdb
```

Application will need to import the library

```javascript
var ssdb = require('ssdb');
```

Then the connection pool should be created using environment variables.

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```javascript

var host = process.env.SSDB_HOST
var port = process.env.SSDB_PORT
var pass = process.env.SSDB_PASSWORD

var pool = ssdb.createPool({
  host: host,
  port: port,
  auth: pass
  });

```

Now when connection pool is created it's ready to be used.

```javascript
pool.acquire().set('key', 'val', function(err, data) {
  if (err && err instanceof ssdb.SSDBError)
    throw err;  // ssdb error
});
```


## Using with Go

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
official ★ | ideawu | [Repository](https://github.com/ssdb/gossdb) | This is the official client
hissdb | Eryx | [Repository](https://github.com/lessos/lessgo/tree/master/data/hissdb) | hissdb in lessos/lessgo, supports connection pool.
gossdb | seefan | [Repository](https://github.com/seefan/gossdb) | From the official client derived from the client, supports the connection pool and set, get, habits and most client consistent.

For this example the hissdb library is use.
Application will need to import the library

```go
package main

import (
    "fmt"
    "github.com/lessos/lessgo/data/hissdb"
)

```

Then the connection should be created using environment variables.

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```go

import os
import strconv

func main() {
    host := os.Getenv("SSDB_HOST")
    port := os.Getenv("SSDB_PORT")
    auth := os.Getenv("SSDB_PASSWORD")
    conn, err := hissdb.NewConnector(hissdb.Config{
        Host:    host,
        Port:    strconv.ParseInt(port),
        Auth:    auth,
        Timeout: 3,  // timeout in second, default to 10
        MaxConn: 10, // max connection number, default to 1
    })
    if err != nil {
        fmt.Println("Connect Error:", err)
        return
    }
    defer conn.Close()

```

Now when connection is created it's ready to be used.
 
```go
    conn.Cmd("set", "mykey", "myvalue")
    if rs := conn.Cmd("get", "mykey"); rs.State == "ok" {
        fmt.Println("get OK\n\t", rs.String())
    }
 }
```


## Using with Ruby

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
ssdb-rb | bsm | [Repository](https://github.com/bsm/ssdb-rb) | Ruby client library for SSDB 

Application will need to import the library

```bash
gem install ssdb
```

Then create the connection using environment variables.
```ruby

require "ssdb"
ssdb = SSDB.new url: ENV['SSDB_URL']
```

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```ruby
ssdb.perform("auth", ENV['SSDB_PASSWORD'])
```

Now when connection is created it's ready to be used.

```ruby
ssdb.set("mykey", "myvalue")
# => true

ssdb.get("mykey")
# => "myvalue"
```


## Using with PHP

Available clients

Package | Author | Repository | Description
--- | --- | --- | --- | 
built-in ★ | ideawu | [Repository](https://github.com/ideawu/ssdb/tree/master/api/php) | This is the official client 

Application will need to import the library

```php
include(dirname(__FILE__) . '/SSDB.php');
```

Then create the connection using environment variables

```php

$host = getenv('SSDB_HOST');
$port = getenv('SSDB_PORT');

try{
	$ssdb = new SimpleSSDB($host, $port);
}catch(Exception $e){
	die(__LINE__ . ' ' . $e->getMessage());
}

```

SSDBHub enforces authentication on SSDB instance. After the client is 
created, it should be authenticated with the password which is
returned in SSDB_PASSWORD configuration variable.

```php
$password = getenv('SSDB_PASSWORD');
$ssdb->auth($password);
```

Now when connection is created it's ready to be used.

```php
$ssdb->set("mykey", "myvalue")
# => true

echo $ssdb->get("mykey")
# => "myvalue"
```


## Using with Redis Clients

Even though SSDB has its own original client libraries for different programming 
languages, it also supports Redis protocol, which makes possible to 
communicate with SSDB database using Redis clients.

Below is the list of supported Redis commands and how they map to SSDB.

### Key-Value

<table class="table table-striped">
<thead>
	<tr>
		<th width="200">Redis</th>
		<th>SSDB</th>
	</tr>
</thead>
<tbody>
	<tr><td>get</td><td>get</td></tr>
	<tr><td>set</td><td>set</td></tr>
	<tr><td>setex</td><td>setx(for kv type only)</td></tr>
	<tr><td>del</td><td>del</td></tr>
	<tr><td>incr/incrBy</td><td>incr</td></tr>
	<tr><td>decr/decrBy</td><td>decr</td></tr>
	<tr><td>mget/getMultiple</td><td>multi_get</td></tr>
	<tr><td>setMultiple</td><td>multi_set</td></tr>
	<tr><td>del(multiple)</td><td>multi_del</td></tr>
	<tr><td>keys</td><td>keys(for kv type only)</td></tr>
	<tr><td>getset</td><td>getset</td></tr>
	<tr><td>setnx</td><td>setnx</td></tr>
	<tr><td>exists</td><td>exists</td></tr>
	<tr><td>ttl</td><td>ttl</td></tr>
	<tr><td>expire</td><td>expire</td></tr>
	<tr><td>getbit</td><td>getbit</td></tr>
	<tr><td>setbit</td><td>setbit</td></tr>
	<tr><td>bitcount</td><td>redis_bitcount, countbit</td></tr>
	<tr><td>strlen</td><td>strlen</td></tr>
	<tr><td>getrange</td><td>getrange</td></tr>
</tbody>
</table>

### Key-(Key-Value)

<table class="table table-striped">
<thead>
	<tr>
		<th width="200">Redis</th>
		<th>SSDB</th>
	</tr>
</thead>
<tbody>
	<tr><td>del(<b>not supported</b>)</td><td>hclear</td></tr>
	<tr><td>hget</td><td>hget</td></tr>
	<tr><td>hset</td><td>hset</td></tr>
	<tr><td>hdel, hmdel</td><td>hdel, multi_hdel</td></tr>
	<tr><td>hIncrBy</td><td>hincr</td></tr>
	<tr><td>hDecrBy</td><td>hdecr</td></tr>
	<tr><td>hKeys</td><td>hkeys</td></tr>
	<tr><td>hVals</td><td>hscan</td></tr>
	<tr><td>hMGet</td><td>multi_hget</td></tr>
	<tr><td>hMSet</td><td>multi_hset</td></tr>
	<tr><td>hLen</td><td>hsize</td></tr>
	<tr><td>hExists</td><td>hexists</td></tr>
	<tr><td>keys</td><td>hlist(for hash type only)</td></tr>
</tbody>
</table>


### Key - (Sorted List)

<table class="table table-striped">
<thead>
	<tr>
		<th width="200">Redis</th>
		<th>SSDB</th>
	</tr>
</thead>
<tbody>
	<tr><td>del(<b>not supported</b>)</td><td>zclear</td></tr>
	<tr><td>zScore</td><td>zget</td></tr>
	<tr><td>zAdd</td><td>zset</td></tr>
	<tr><td>zRem</td><td>zdel</td></tr>
	<tr><td>zRange</td><td>zrange</td></tr>
	<tr><td>zRevRange</td><td>zrrange</td></tr>
	<tr><td>zRangeByScore</td><td>zscan</td></tr>
	<tr><td>zRevRangeByScore</td><td>zrscan</td></tr>
	<tr><td>zIncrBy</td><td>zincr</td></tr>
	<tr><td>zDecrBy</td><td>zdecr</td></tr>
	<tr><td>zCount</td><td>zcount</td></tr>
	<tr><td>zSum</td><td>zsum</td></tr>
	<tr><td>zAvg</td><td>zavg</td></tr>
	<tr><td>zCard</td><td>zsize</td></tr>
	<tr><td>zRank</td><td>zrank</td></tr>
	<tr><td>zRemRangeByRank</td><td>zremrangebyrank</td></tr>
	<tr><td>zRemRangeByScore</td><td>zremrangebyscore</td></tr>
	<tr><td>keys</td><td>zlist(for zset type only)</td></tr>
</tbody>
</table>

### Key-Queue

<table class="table table-striped">
<thead>
	<tr>
		<th width="200">Redis</th>
		<th>SSDB</th>
	</tr>
</thead>
<tbody>
	<tr><td>del(<b>not supported</b>)</td><td>qclear</td></tr>
	<tr><td>llen/lsize</td><td>qsize</td></tr>
	<tr><td>lpush</td><td>qpush_front</td></tr>
	<tr><td>rpush</td><td>qpush_back</td></tr>
	<tr><td>lpop</td><td>qpop_front</td></tr>
	<tr><td>rpop</td><td>qpop_back</td></tr>
	<tr><td>lrange</td><td>qslice</td></tr>
	<tr><td>lindex, lget</td><td>qget</td></tr>
	<tr><td>lset</td><td>qset</td></tr>
	<tr><td>keys</td><td>qlist(for queue/list type only)</td></tr>
</tbody>
</table>



## Dashboard
<!--![Alt text](https://s3.amazonaws.com/heroku-devcenter-files/article-images/1480396320-Screenshot-291116-07-08-09.png)-->

The dashboard contains following components:

#### Connection Details 

The variables you should use in order to connect and authenticate against your
SSDB cluster. 
They shown at the top of the dashboard. 

```bash
SSDB_HOST
SSDB_PORT
SSDB_URL
SSDB_PASSWORD
```
 
#### Plan Usage

There are a couple of important things to know about the Plan Usage 
representation:

1. It indicates how much space on the disk is used by your database.
2. It's calculated approximately, so you might notice minor variations in it's value.
3. It represents the size of compressed data. The actual size of data might be by 1.5 - 4 times bigger.

It's important to not exceed the Plan Usage limit, provided by your plan.
If this happens, we will send you an email and post the notice at your Dashboard.

#### Backup Button
 
Backup Button downloads SSDB dump file to your local computer. 
Keep in mind that the size of the dump file might be by 1.5-4 times bigger 
then it's indicated by Plan Usage.

#### Flush All Button

You can remove all data from the database by clicking "Flush All" button.
This will schedule `flushdb` action to all SSDB nodes in your cluster. 
It may take awhile to process this action, before the changes will be shown on
the dashboard.

## Troubleshooting

In case of connection problems, please make sure you are using right 
connection details.

If you are getting an alert message on the top of the dashboard view, 
saying that Plan Usage limit is exceeded, 
you should consider to decrease the amount of data in your database, 
or update your plan to get a higher limit.

You may see the message, saying that your account has been suspended, 
which means the Plan Usage limit was extensively outreached
and your SSDB nodes were killed by the system. 
All the data stored on them is lost in such a case.

Given the above, it's a good practice to make regular backups. 
All our plans provide possibility to download SSDB dump file to your 
local machine.

Our Premium Plans provides automatic backups, managed by us and 
available for download for period of one week.

## Migrating between plans

* Application owners should carefully manage the migration timing to ensure 
proper application function during the migration process. 

Use the `heroku addons:upgrade` command to migrate to a new plan.

```bash
$ heroku addons:upgrade ssdb:newplan
-----> Upgrading ssdb:newplan to sharp-mountain-4005... done, v18 ($49/mo)
       Your plan has been updated to: ssdb:newplan
```

## Removing the add-on

SSDBHub plugin can be removed via the CLI.

* **Warning!** This will destroy all associated data. The process is final and cannot be undone!

```bash
$ heroku addons:destroy ssdb
-----> Removing ssdb from sharp-mountain-4005... done, v20 (free)
```

## Support

All SSDBHub support and runtime issues should be submitted via one of the [Heroku Support channels](https://devcenter.heroku.com/articles/support-channels). 
Any non-support related issues or product feedback are welcome at feedback@ssdbhub.com
