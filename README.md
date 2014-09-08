hummingbird - no bullshit HTTP load testing suite

# hstress

`hstress` generates large HTTP workloads.

Options are as follows:

    hstress [-c CONCURRENCY] [-b BUCKETS] [-n COUNT] [-p NUMPROCS] [-r RPC] [-i INTERVAL] <-f URIFILE> [HOST] [PORT]

The default host is `127.0.0.1`, and the default port is `80`.

* `-c` controls concurrency. This is the number of outstanding
  requests at a given time
  
* `-b` sets the "bucket spec".  This determines how we bucket the
  measurement histograms. Set to a comma-separated list of values in
  milliseconds. For example `1,10,100,250,500` will bucket requests
  into the given amount of milliseconds.
  
* `-n` controls the total number of requests to make. Left
  unspecified, `hb` never terminates.
  
* `-r` specifies the number of requests per connection (default is no limit)

* `-p` controls the number of processes to fork (for multiple event
  loops). The default value is `1`.
  
* `-i` specifies the reporting interval in seconds
* `-f` specifies an optional file to read request URI(s) from


`hb` produces output like the following:

	$ hb -n100000 -c20 localhost 8080
	# params: c=20 p=1 n=100000 r=-1
	# ts        errors   timeout  closes     <1        <10       <100     >=100    hz
	1310334247  0        0        220        22393     93        0        0        22351
	1310334248  0        0        220        22637     30        0        0        22689
	1310334249  0        0        226        22566     37        0        0        22625
	1310334250  0        0        230        22439     51        0        0        22490
	1310334250  0        0        115        9752      21        0        0        22727
	# total		100019
	# errors	0
	# timeouts	0
	# closes	1011
	# <1		99787
	# <10		232
	# <100		0
	# >=100		0
	# hz		22542

The first column is the timestamp, and the subsequent columns are
according to the specified bucketing (controlled via `-b`). This
output format is handy for analysis with the standard Unix tools. The
banner is written to `stderr`, so only the data values are emitted to
`stdout`.

# hplay

`hplay` replays http requests at a constant rate. E.g.

	# hplay localhost 8000 100 httpreqs
	
will replay the HTTP requests stored in `httpreqs` to `localhost:8000` at a rate of 100 per second. Request parsing is robust so you can give it packet dumps.

For example, on a server host that receives requests you wish to replay:

	$ tcpdump -n -c500 -i any dst port 10100 -s0 -w capture
	
Then reconstruct it with [tcpflow](http://www.circlemud.org/~jelson/software/tcpflow/):

	$ tcpflow -r capture -c | sed 's/^...\....\....\....\......\-...\....\....\....\......: //g' > reqs
	
And finally, replay these requests onto localhost:8000:

	$ hplay localhost 8000 100 reqs

# hserve

`hserve` is a simple HTTP server that will yield a constant response.

# TODO

* support for constant rate load generation
* should be split into two programs? load generation & http requests?
