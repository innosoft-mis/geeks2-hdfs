# Lab 2: Get Log Data with Flume

- [Upload multiple files to HDFS using FLUME](#upload-multiple-files-to-hdfs-using-flume)
- [Stream Log data from Syslog to HDFS](#stream-log-data-from-syslog-to-hdfs)
- [Read data from single file to HDFS](#read-data-from-single-file-to-hdfs)

## Upload multiple files to HDFS using FLUME

#### 1) Review file content and prepare a directory on HDFS
Dataset Explanations: Multiple CSV format files are on a local path. The data in the files is banking transactions.  

Investigate files on local directory
```
$ cd ~/lab2
$ head -n 10 flume_data/001.csv
$ head -n 10 flume_data/002.csv
$ head -n 10 flume_data/003.csv
```
Create output directory on HDFS
```
$ hdfs dfs -mkdir /user/cloudera/flume_output
```
#### 2) Create a FLUME configuration file to spool a local directory
Investigate files on local directory
```
$ cd ~/lab2
$ gedit flume.conf &
```
#### 3) Flume Configuration File (flume.conf)
```
cloudera_agent.sources = src1
cloudera_agent.channels = ch1
cloudera_agent.sinks = sink1

cloudera_agent.sources.src1.type = spooldir
cloudera_agent.sources.src1.channels = ch1
cloudera_agent.sources.src1.spoolDir = /home/cloudera/lab2/flume_data
cloudera_agent.sources.src1.batchSize = 1000

cloudera_agent.channels.ch1.type = memory
cloudera_agent.channels.ch1.capacity = 100000000
cloudera_agent.channels.ch1.transactionCapacity = 5000

cloudera_agent.sinks.sink1.type = hdfs
cloudera_agent.sinks.sink1.channel = ch1
cloudera_agent.sinks.sink1.hdfs.batchSize = 5000
cloudera_agent.sinks.sink1.hdfs.path = /user/cloudera/flume_output
cloudera_agent.sinks.sink1.hdfs.filePrefix = log-output
cloudera_agent.sinks.sink1.hdfs.rollSize = 5120000
cloudera_agent.sinks.sink1.hdfs.rollInterval = 10
cloudera_agent.sinks.sink1.hdfs.rollCount = 0
cloudera_agent.sinks.sink1.hdfs.fileType = DataStream
cloudera_agent.sinks.sink1.hdfs.writeFormat = Text
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
