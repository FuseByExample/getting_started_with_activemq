Getting Started with ActiveMQ
=============================

This project includes a number of example configurations showing various ActiveMQ High Availability scenarios. These
include: Master-Slave, Network of Brokers, and Network of Master-Slave Brokers. This project also includes a simple JMS
Queue (point to point) message producer and message consumer.

Notes:

* all instructions assume you are executing from the top level directory of this project
* it is assumed you have Apache Maven installed, and that you are familiar with its usage
* assumes you are using ActiveMQ 5.5.1 or later

Note: if you are using Fuse MQ Enterprise, the shell command is `bin/fusemq` versus `bin/activemq`. Everything else
should be the same.

The Master-Slave configuration enables quick activation of a broker instance to continue processing of messages stored
within a message persistence store. It utilizes a shared file system, in this case the same local directory, with both
brokers referencing this same directory. The first broker instance to acquire a file lock is considered the 'master',
and all subsequent broker instances waiting for the file lock are considered 'slave'.

To run this sample,

Start the 'master' broker in a shell:

    shell1> <activemq_home>/bin/activemq console xbean:file:conf/activemq-failover1.xml

Start the 'slave' broker in a different shell:

    shell2> <activemq_home>/bin/activemq console xbean:file:conf/activemq-failover2.xml
    
Alternatively you can start ActiveMQ from Maven:

    shell1> mvn -P broker-failover1
    shell2> mvn -P broker-failover2

Start the message consumer in another shell:

    shell3> mvn -P consumer

Start the message producer in another shell:

    shell4> mvn -P producer

The message producer is coded to send 100 messages. The consumer will log the messages it receives, and will timeout
and exit after 120 seconds of inactivity (no messages received). To test failover, kill (Ctrl+C) the current master
broker while the Producer is sending messages. You will see a log entry about the clients reconnecting. You will
eventually see log entries in the slave broker about it starting up; it may take up to 30 seconds for the slave to
detect that the master has failed and startup.

To test fail back, restart the failed broker, see the log entries about it not acquiring the file lock, kill (Ctrl+C)
the currently active broker, and see the client automatically fail back. The automatic reconnect be the clients assumes,
as is the case with this example, that the clients are using the fail over transport.


The Network of Brokers configuration allows you to scale out the processing of messages across multiple brokers. In
this configuration, messages are forwarded from the broker the producer sends the messages to the networked broker that
the consumer is connected to. The clients have been configured to connect to different brokers.

To run this sample,

Start the first broker in a shell:

    shell1> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob1.xml

Start the second broker in another shell:

    shell2> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob2.xml

Alternatively you can start ActiveMQ from Maven:

    shell1> mvn -P broker-nwob1
    shell2> mvn -P broker-nwob2

Start the message consumer in another shell:

    shell3> mvn -P consumer-nwob

Start the message producer in another shell:

    shell4> mvn -P producer-nwob

You can test the brokers forwarding messages by randomly killing and restarting broker instances. You'll see messages
stop when the broker is stopped, and you'll see any sent messages forwarded when the brokers are restarted and the
network connection is re-established. This shows how ActiveMQ is saving persistent messages in its store for delivery
when the broker re-starts. You can especially see this if you kill the broker the consumer is connected to
(activemq-nwob1.xml) while the producer continues sending messages to the other broker.


The last scenario is the Network of Master-Slave brokers. This combines both the scale out capabilities of the Network
of Brokers with quick access to persistent messages of Master-Slave.

To run this sample,

Start the first 'master' broker in a shell:

    shell1> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob-ms1a.xml

Start the first 'slave' broker in another shell:

    shell2> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob-ms1b.xml

Start the second 'master' broker in another shell:

     shell3> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob-ms2a.xml

Start the second 'slave' broker in another shell:

     shell4> <activemq_home>/bin/activemq console xbean:file:conf/activemq-nwob-ms2b.xml

Alternatively you can start ActiveMQ from Maven:

    shell1> mvn -P broker-nwob-ms1a
    shell2> mvn -P broker-nwob-ms1b
    shell3> mvn -P broker-nwob-ms2a
    shell4> mvn -P broker-nwob-ms2b

Start the message consumer

    shell5> mvn -P consumer-nwob-ms

Start the message producer

    shell6> mvn -P producer-nwob-ms

You can test the configuration by killing and restarting broker instances.  Both fail-over and fail-back should be
observed, as in the first example, as well as message-forwarding across the broker network in the second example.