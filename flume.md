# Lab XX: Get Log Data with Flume

- [Upload multiple files to HDFS using FLUME](#upload-multiple-files-to-hdfs-using-flume)
- [Stream Log data from Syslog to HDFS](#stream-log-data-from-syslog-to-hdfs)
- [Read data from single file to HDFS](#read-data-from-single-file-to-hdfs)

## Upload multiple files to HDFS using FLUME

#### 1) Review file content and prepare a directory on HDFS
Dataset Explanations: Multiple CSV format files are on a local path. The data in the files is banking transactions.  

Investigate files on local directory
```sh
cd ~/flume_data
head -n 10 flume_data/001.csv
head -n 10 flume_data/002.csv
head -n 10 flume_data/003.csv
```
Create output directory on HDFS
```sh
hadoop fs -mkdir /user/student/flume_output
```
#### 2) Create a FLUME configuration file to spool a local directory
Investigate files on local directory
```sh
cd ~
gedit flume.conf &
```
#### 3) Flume Configuration File (flume.conf)
```sh
# Name of the agent
agent1.sources = source1
agent1.sinks = sink1
agent1.channels = channel1

# Configuring the source
agent1.sources.source1.type = spooldir
agent1.sources.source1.spoolDir = /home/student/flume_data
agent1.sources.source1.batchSize = 1000
agent1.sources.source1.channels = channel1

# Configuring the channel
agent1.channels.channel1.type = memory
agent1.channels.channel1.capacity = 10000000
agent1.channels.channel1.transactionCapacity = 5000


# Configuring the sink
agent1.sinks.sink1.type = avro
agent1.sinks.sink1.hostname = localhost
agent1.sinks.sink1.port = 55051
agent1.sinks.sink1.batch-size = 5000
agent1.sinks.sink1.channel = channel1

# Name of the agent
agent2.sources = source2
agent2.sinks = sink2
agent2.channels = channel2

# Configuring the source
agent2.sources.source2.type = avro
agent2.sources.source2.bind = 0.0.0.0
agent2.sources.source2.port = 55051
agent2.sources.source2.channels = channel2

# Configuring the channel
agent2.channels.channel2.type = memory
agent2.channels.channel2.capacity = 1000000
agent2.channels.channel2.transactionCapacity = 5000

# Configuring the sink
agent2.sinks.sink2.type = hdfs
agent2.sinks.sink2.batchSize = 5000
agent2.sinks.sink2.hdfs.path = /user/student/flume_output
agent2.sinks.sink2.hdfs.fileType = DataStream
agent2.sinks.sink2.hdfs.writeFormat = Text
agent2.sinks.sink2.hdfs.rollInterval = 10
agent2.sinks.sink2.hdfs.rollSize = 0
agent2.sinks.sink2.hdfs.rollCount = 100
agent2.sinks.sink2.channel = channel2
```
#### 4)	Start a FLUME agent to ingest files into HDFS
```
$ flume-ng agent -n cloudera_agent -f flume.conf -Xms1024m -Xmx2048m
```
NOTE: -Xms and –Xmx is memory configurable for Flume Java Heap
#### 5) View the result on HDFS
View the result on HDFS  
```
[Open new terminal]

$ hdfs dfs -cat flume_output/*
```

## Stream Log data from Syslog to HDFS

#### 1)	Import data from Syslog to HDFS using Flume
Create output directory on HDFS
```
$ hdfs dfs -mkdir /user/cloudera/syslog
```

#### 2)	Create a FLUME configuration file to listen syslog port
Investigate files on local directory
```
$ cd ~/lab2
$ gedit rsyslog.conf &
```

#### 3)	Flume Configuration File (rsyslog.conf)
```
# Name the components on this agent
a1.sources = src1
a1.sinks = sink1
a1.channels = ch1

# I'll be using TCP based Syslog source
a1.sources.src1.type = syslogtcp
# the port that Flume Syslog source will listen on
a1.sources.src1.port = 5140
# the hostname that Flume Syslog source will be running on
a1.sources.src1.host = localhost
a1.sources.src1.channels = ch1

# Describe the sink
a1.sinks.sink1.type = hdfs
a1.sinks.sink1.channel = ch1
a1.sinks.sink1.hdfs.path = /user/cloudera/syslog
a1.sinks.sink1.hdfs.batchSize = 10000
a1.sinks.sink1.hdfs.rollSize = 512000
a1.sinks.sink1.hdfs.fileType = DataStream
a1.sinks.sink1.hdfs.writeFormat = Text

# Use a channel which buffers events in memory
a1.channels.ch1.type = memory
a1.channels.ch1.capacity = 10000000
a1.channels.ch1.transactionCapacity = 5000
```

#### 4)	Start a rsyslog service
Modified the /etc/rsyslog.conf
```
$ sudo gedit /etc/rsyslog.conf &
```
Provides TCP syslog reception
```
# Provides TCP syslog reception
$ModLoad imtcp
$InputTCPServerRun 514
*.*	@@localhost:5140
```
Start a rsyslog service
```
$ sudo service rsyslog restart
```

#### 5)	Start a FLUME agent to ingest files into HDFS
```
$ flume-ng agent -n a1 -f rsyslog.conf -Xms1024m -Xmx2048m
```
NOTE: -Xms and –Xmx is memory configurable for Flume Java Heap

#### 6)	Test log forward to HDFS
```
[Open new terminal]
```
Generate log data
```
$ logger -t test 'testing flume with syslog'
$ sudo cat /var/log/messages
```
View the result on HDFS 
```
$ hdfs dfs -cat syslog/*
```

## Read data from single file to HDFS

#### 1)	Read data from log file to flume
Investigate files on local directory
```
$ cd ~/lab2
$ gedit readlog.conf &
```

#### 2)	Flume Configuration File (readlog.conf)
```
a1.sources = src1
a1.sinks = sink1
a1.channels = ch1

#Source
a1.sources.src1.type = exec
a1.sources.src1.command = tail -f /var/log/dmesg
a1.sources.src1.channels = ch1

# Describe the sink
a1.sinks.sink1.type = hdfs
a1.sinks.sink1.channel = ch1
a1.sinks.sink1.hdfs.path = /user/cloudera/dmessages
a1.sinks.sink1.hdfs.batchSize = 10000
a1.sinks.sink1.hdfs.rollSize = 512000
a1.sinks.sink1.hdfs.fileType = DataStream
a1.sinks.sink1.hdfs.writeFormat = Text

# Use a channel which buffers events in memory
a1.channels.ch1.type = memory
a1.channels.ch1.capacity = 10000000
a1.channels.ch1.transactionCapacity = 5000
```

#### 3)	Start a FLUME agent to ingest files into HDFS
```
$ flume-ng agent -n a1 -f readlog.conf -Xms1024m -Xmx2048m
```
NOTE: -Xms and –Xmx is memory configurable for Flume Java Heap

#### 4)	Test log forward to HDFS
```
[Open new terminal]
```
Generate log data
```
$ sudo su -c 'echo HELLO > /dev/kmsg'
$ dmesg
$ sudo su -c "dmesg > /var/log/dmesg"
```
View the result on HDFS 
```
$ hdfs dfs -cat dmessages/*
```
