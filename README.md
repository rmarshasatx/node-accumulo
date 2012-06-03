node-accumulo
=============

This project uses node.js to ingest into Apache Accumulo via RabbitMQ and a Java application. The intent is to store records of a client visiting a website, much like a Google Analytics. This example is meant to be contrived and the intent is to show the potential of using node to (indirectly) ingest data into Apache Accumulo. 

### Brief Summary

[node.js][] runs an HTTP webserver which accepts incoming requests from an HTTP client. Upon receipt, it strips the intended information off the query string of the request URL and fires a JSON string over RabbitMQ. Meanwhile, a Java process is running in the background, pulling data off of a queue that the node server is writing to. Upon receipt of a message, the Java process converts the JSON string into an object, creates, and then inserts mutations into Accumulo corresponding to the data contained in the JSON string.

### Prerequisites

I'll make the (bold) assumption that you already have [Apache Hadoop][], [Apache Zookeeper][], and [Apache Accumulo][] installed on your machine. There is good documentation elsewhere for this.

#### RabbitMQ and node.js

These should be relatively straightfoward, something along the lines of:

    # emerge net-misc/rabbitmq-server net-libs/nodejs

Otherwise, you can manually install both [RabbitMQ][] and [node.js][]. Don't forget to start RabbitMQ.

    # /etc/init.d/rabbitmq start

#### node-amqp-module

Things get a little tricky with the [node-amqp-module][]. The version of the module pulled down using npm was old so I had to pull down the code manually, build the module, and tell node about the path

    $ git clone https://github.com/postwait/node-amqp.git node-amqp.git
    $ cd node-amqp.git && make
    $ export NODE_PATH=$NODE_PATH:/path/to/node-amqp.git

#### Running

Make sure you have Hadoop, Zookeeper, and Accumulo started, then start the node process

    $ node node/server.js

Build and run the AmqpWebAnalytics class

    $ cd java/webanalytics
    $ mvn package
    $ bin/run.sh

Then, fire up curl and request the URL

    $ curl "http://localhost:12345/post?host=10.0.0.1&visitor=10.0.0.2"

At this point, you should have a print statement from both the node and Java process acknowledging that they both received the message, and, you should also see a new entry in a new Accumulo table with the information you provided, similar to:

    root@accumulo analytics> scan
    10.0.0.1 10.0.0.2:1335928232348 []

#### Having fun

If you really want to test out the server, try running siege to flood the process :D

    # emerge -av app-benchmarks/siege
    $ siege.config
    $ siege -b -c 800 "http://localhost:12345/post?host=10.0.0.1&visitor=10.0.0.2"

[RabbitMQ]: http://www.rabbitmq.com/ "RabbitMQ"
[node.js]: http://nodejs.org/       "node.js"
[node-amqp-module]: https://github.com/postwait/node-amqp "node-amqp module"
[Apache Hadoop]: http://hadoop.apache.org/common/docs/r0.20.2/quickstart.html#PseudoDistributed "Apache Hadoop"
[Apache Zookeeper]: http://zookeeper.apache.org/doc/r3.3.1/zookeeperStarted.html "Apache Zookeeper"
[Apache Accumulo]: http://accumulo.apache.org/1.4/user_manual/Administration.html "Apache Accumulo"