# Memcached
If you are on a UNIX system, then using `ps -eaf | grep memcached` command will get you the port Memcached server is running on

For example

```shell
$ ps -eaf | grep memcached
503 55442 55296   0   0:00.14 ttys000   0:00.22 memcached -p 11111 -vv
503 58945 56875   0   0:00.01 ttys003	0:00.01 grep memcached
````

So memcached is running on TCP port 11111 and verbose mode (-vv). If you wanto to run as a daemon process then use -d option in the startup command

## Memcached Telnet Commands
To connect to memcached server with telnet and start a session
```shell
$ telnet localhost 11111
```
To store data in Memcached server with telnet
```shell
set KEY META_DATA EXPIRY_TIME LENGTH_IN_BYTES
````
To retrieve data from Memcached through telnet
```shell
get KEY
````
To overwrite the existing key
```shell
replace KEY META_DATA EXPIRIY_TIME LENGTH_IN_BYTES
````
To delete the key
```shell
delete KEY
```
To get server statistics
```shell
stats
stats items
stats slabs
````
To clear the cache data
```shell
flush_all
```
To quit the telnet session
```shell
quit
````

## Memcached Server Telnet Example
```shell
$ telnet localhost 11111
Trying ::1...
Connected to localhost.
Escape character is '^]'.
set Test 0 100 10
JournalDev
STORED
get Test
VALUE Test 0 10
JournalDev
END
replace Test 0 100 4
Temp
STORED
get Test
VALUE Test 0 4
Temp
END
stats items
STAT items:1:number 1
STAT items:1:age 19
STAT items:1:evicted 0
STAT items:1:evicted_time 0
STAT items:1:outofmemory 0
STAT items:1:tailrepairs 
END
flush all
OK
get Test
END
version
VERSION 1.2.8
quit
Connection closed by foreign host.
$
````

by [JournalDev](http://journaldev.com/16/memcached-telnet-commands-example)