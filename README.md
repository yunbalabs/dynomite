<!--
# Dynomite
-->
**Dynomite**, inspired by [Dynamo whitepaper](http://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf), is a thin, distributed dynamo layer for different storage engines and protocols. Currently these include [Redis](http://redis.io) and [Memcached](http://www.memcached.org/).  Dynomite supports multi-datacenter replication and is designed for high availability.
<center>![dynomite logo](images/dynomite-logo.png?raw=true =150x150)</center>

The ultimate goal with Dynomite is to be able to implement high availability and cross-datacenter replication on storage engines that do not inherently provide that functionality. The implementation is efficient, not complex (few moving parts), and highly performant.

## Build

To build Dynomite from source with _debug logs enabled_ and _assertions disabled_:

    $ git clone git@github.com:Netflix/dynomite.git
    $ cd dynomite
    $ autoreconf -fvi
    $ ./configure --enable-debug=log
    $ make
    $ src/dynomite -h

To build Dynomite in _debug mode_:

    $ git clone git@github.com:Netflix/dynomite.git
    $ cd dynomite
    $ autoreconf -fvi
    $ CFLAGS="-ggdb3 -O0" ./configure --enable-debug=full
    $ make
    $ sudo make install
    
To build on Mac, use feature/mac-port branch:

    $ git checkout feature/mac-port
    
## Help

    Usage: dynomite [-?hVdDt] [-v verbosity level] [-o output file]
                      [-c conf file] [-s stats port] [-a stats addr]
                      [-i stats interval] [-p pid file] [-m mbuf size]
                      [-M max alloc messages]

    Options:
      -h, --help             : this help
      -V, --version          : show version and exit
      -t, --test-conf        : test configuration for syntax errors and exit
      -d, --daemonize        : run as a daemon
      -D, --describe-stats   : print stats description and exit
      -v, --verbosity=N      : set logging level (default: 5, min: 0, max: 11)
      -o, --output=S         : set logging file (default: stderr)
      -c, --conf-file=S      : set configuration file (default: conf/dynomite.yml)
      -s, --stats-port=N     : set stats monitoring port (default: 22222)
      -a, --stats-addr=S     : set stats monitoring ip (default: 0.0.0.0)
      -i, --stats-interval=N : set stats aggregation interval in msec (default: 30000 msec)
      -p, --pid-file=S       : set pid file (default: off)
      -m, --mbuf-size=N      : set size of mbuf chunk in bytes (default: 16384 bytes)
      -M, --max-msgs=N       : set max number of messages to allocate (default: 2000000)


## Configuration

Dynomite can be configured through a YAML file specified by the -c or --conf-file command-line argument on process start. The configuration files parses and understands the following keys:

+ **env**: Specify environment of a node.  Currently supports aws and network (for physical datacenter).
+ **datacenter**: The name of the datacenter.  Please refer to [architecture document](https://github.com/Netflix/dynomite/wiki/Architecture).
+ **rack**: The name of the rack.  Please refer to [architecture document](https://github.com/Netflix/dynomite/wiki/Architecture).
+ **dyn_listen**: The port that dynomite nodes use to inter-communicate and gossip.
+ **gos_interval**: The sleeping time in milliseconds at the end of a gossip round.
+ **tokens**: The token(s) owned by a node.  Currently, we don't support vnode yet so this only works with one token for the time being.
+ **dyn_seed_provider**: A seed provider implementation to provide a list of seed nodes.
+ **dyn_seeds**: A list of seed nodes in the format: address:port:rack:dc:tokens (node that vnode is not supported yet)
+ **listen**: The listening address and port (name:port or ip:port) for this server pool.
+ **timeout**: The timeout value in msec that we wait for to establish a connection to the server or receive a response from a server. By default, we wait indefinitely.
+ **preconnect**: A boolean value that controls if dynomite should preconnect to all the servers in this pool on process start. Defaults to false.
+ **data_store**: An integer value that controls if a server pool speaks redis (0) or memcached (1) or other protocol. Defaults to redis (0).
+ **server_connections**: The maximum number of connections that can be opened to each server. By default, we open at most 1 server connection.
+ **auto_eject_hosts**: A boolean value that controls if server should be ejected temporarily when it fails consecutively server_failure_limit times. See [liveness recommendations](notes/recommendation.md#liveness) for information. Defaults to false.
+ **server_retry_timeout**: The timeout value in msec to wait for before retrying on a temporarily ejected server, when auto_eject_host is set to true. Defaults to 30000 msec.
+ **server_failure_limit**: The number of consecutive failures on a server that would lead to it being temporarily ejected when auto_eject_host is set to true. Defaults to 2.
+ **servers**: A list of local server address, port and weight (name:port:weight or ip:port:weight) for this server pool. Usually there is just one.

For example, the configuration file in [conf/dynomite.yml](conf/dynomite.yml)

Finally, to make writing syntactically correct configuration files easier, dynomite provides a command-line argument -t or --test-conf that can be used to test the YAML configuration file for any syntax error.



## License

Licensed under the Apache License, Version 2.0: http://www.apache.org/licenses/LICENSE-2.0

## benchmark
### redis
localhost:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 6379
PING_INLINE: 109529.02 requests per second
PING_BULK: 111856.82 requests per second
SET: 111982.08 requests per second
GET: 112107.62 requests per second
INCR: 107411.38 requests per second
SADD: 111856.82 requests per second
```

remote:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 6379 -h 192.168.2.120
PING_INLINE: 35149.38 requests per second
PING_BULK: 35473.57 requests per second
SET: 33726.81 requests per second
GET: 34164.67 requests per second
INCR: 32626.43 requests per second
SADD: 34059.95 requests per second
```

### dynomite (single)
localhost:
```
redis-benchmark -t ping,set,get,incr,sadd -n 100000 -q -p 8102
PING_INLINE: 61162.08 requests per second
PING_BULK: 57504.31 requests per second
SET: 58275.06 requests per second
GET: 50100.20 requests per second
INCR: 51733.06 requests per second
SADD: 57306.59 requests per second
```

remote:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 8102 -h 192.168.2.120
PING_INLINE: 24330.90 requests per second
PING_BULK: 24026.91 requests per second
SET: 22956.84 requests per second
GET: 23507.29 requests per second
INCR: 24667.00 requests per second
SADD: 23185.72 requests per second
```

### dynomite (two rack)
localhost:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 8102
PING_INLINE: 56625.14 requests per second
PING_BULK: 55803.57 requests per second
SET: 28042.62 requests per second
GET: 55555.56 requests per second
INCR: 28192.84 requests per second
SADD: 28860.03 requests per second
```

remote:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 8102 -h 192.168.2.120
PING_INLINE: 21181.95 requests per second
PING_BULK: 20449.90 requests per second
SET: 18218.26 requests per second
GET: 20345.88 requests per second
INCR: 18754.69 requests per second
SADD: 18765.25 requests per second
```

### dynomite (two rack + write_consistency)
localhost:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 8102
PING_INLINE: 63171.20 requests per second
PING_BULK: 56148.23 requests per second
SET: 24987.51 requests per second
GET: 56116.72 requests per second
INCR: 25284.45 requests per second
SADD: 25195.26 requests per second
```

remote:
```
redis-benchmark -t ping,set,get,incr,sadd -q -n 100000 -p 8102 -h 192.168.2.120
PING_INLINE: 23568.23 requests per second
PING_BULK: 23724.79 requests per second
SET: 17624.25 requests per second
GET: 22716.95 requests per second
INCR: 17689.72 requests per second
SADD: 17930.79 requests per second
```
