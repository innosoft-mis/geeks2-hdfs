# Hands-on Lab 1: HDFS 

## Objective:
- Transfer IN/OUT files between local file-system and HDFS
-	Files Management on HDFS
-	Manipulate file on HDFS

## 0) start HDFS Service
```
start-dfs.sh
start-yarn.sh
```

## 1) Upload dataset to HDFS

```
hadoop fs -put chospital.csv
```

## 2)	List file on HDFS, explain the difference result between -ls and -lsr

```
hadoop fs –ls
hadoop fs -lsr 
```

## 3)	Create directory on HDFS, and move file into created directory.

```
hadoop fs -mkdir chospital_dataset
hadoop fs -mv /user/student/chospital.csv /user/student/chospital_dataset
```

Rename the chospital file
```
hadoop fs -mv /user/student/chospital_dataset/chospital.csv /user/student/chospital_dataset/data.csv
```

## 4)	Display contents of chospital.csv file

Display All Content in the terminal
```
hadoop fs -cat /user/student/chospital_dataset/data.csv
```

Display only few head line
```
hadoop fs -head /user/student/chospital_dataset/data.csv
```

Display only few tail line
```
hadoop fs -tail /user/student/chospital_dataset/data.csv
```

## 5)	To find the size of file, amount of space and disk usage. 

Display the file size
```
hadoop fs -du /user/student/chospital_dataset/data.csv
hadoop fs -du -h /user/student/chospital_dataset/data.csv
```

Display the amount of space
```
hadoop fs -du -h /user/student/
hadoop fs -du -h -s /user/student/
```

Show the capacity, free and used space of the filesystem
```
hadoop fs -df -h
```

## 6)	Backup the HDFS data using HDFS Snapshot. 

Create the demo directory
```
hadoop fs -mkdir /user/student/snapshot_demo
hadoop fs -touchz /user/student/snapshot_demo/text1.txt
echo "Hello World for Snapshot" > text2.txt
hadoop fs -put text2.txt /user/student/snapshot_demo/
hadoop fs -ls /user/student/snapshot_demo
```

Enable snapshot directory
```
hdfs dfsadmin -allowSnapshot /user/student/snapshot_demo/
hdfs lsSnapshottableDir
```

Create snapshot on demo directory
```
hdfs dfs -createSnapshot /user/student/snapshot_demo
```

## 7)	Restore data from snapshot

Check the snapshot
```
hadoop fs -ls /user/student/snapshot_demo/.snapshot
```

Remove the file
```
hadoop fs -rm /user/student/snapshot_demo/text2.txt
hadoop fs -ls /user/student/snapshot_demo/
```

Restore from snapshot
```
hadoop fs -cp -ptopax /user/student/snapshot_demo/.snapshot/sXXXXXXXX-XXXXXX.XXX/text2.txt /user/student/snapshot_demo/
```

Note:
```
XXXXXXXX-XXXXXX.XXX depend on your drive
-ptopax is restored file timestamp to original
```

Check the restored files
```
hadoop fs -ls /user/student/snapshot_demo/
hadoop fs -cat /user/student/snapshot_demo/text2.txt
```

## 8)	Delete and Disable snapshot

Delete the snapshot 
```
hadoop fs -deleteSnapshot /user/student/snapshot_demo sXXXXXXXX-XXXXXX.XXX
```


## HDFS commands
```
https://github.com/san089/Cloudera_Material/blob/master/hadoop-hdfs-commands-cheatsheet.pdf
```
