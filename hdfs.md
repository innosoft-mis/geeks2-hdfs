# Hands-on Lab 1: HDFS 

## Objective:
- Transfer IN/OUT files between local file-system and HDFS
-	Files Management on HDFS
-	Manipulate file on HDFS

## Login to Virtual Machine
```
Username: testuser
Password: testuser
```
และทำการ Download ไฟล์ hospcode.csv จาก URL ที่วิทยากรแจ้ง
```
cd ~
wget https://github.com/innosoft-mis/de-moph/raw/main/hospcode.csv
```

Open the new terminal and follow the below instruction. (See Hadoop Cheat sheet for the Hadoop command detail.)

## 0) start HDFS Service
```
$ start-dfs.sh
$ start-yarn.sh
```

## 1) Upload dataset to HDFS

```
$ hadoop fs -put ~/Downloads/hospcode.csv
```

## 2)	List file on HDFS, explain the difference result between -ls and -lsr

```
$ hadoop fs –ls
$ hadoop fs -lsr 
```

## 3)	Create directory on HDFS, and move file into created directory.

```
$ hadoop fs -mkdir hospcode_dataset
$ hadoop fs -mv /user/testuser/hospcode.csv /user/testuser/hospcode_dataset
```

Rename the hospcode file
```
$ hadoop fs -mv /user/testuser/hospcode_dataset/hospcode.csv /user/testuser/hospcode_dataset/data.csv
```

## 4)	Display contents of hospcode.csv file

Display All Content in the terminal
```
$ hadoop fs -cat /user/testuser/hospcode_dataset/data.csv
```

Display only few head line
```
$ hadoop fs -cat /user/testuser/hospcode_dataset/data.csv | head
```

Display only few tail line
```
$ hadoop fs -cat /user/testuser/hospcode_dataset/data.csv | tail
```

Display last 1k byte of file 
```
$ hadoop fs -tail /user/testuser/hospcode_dataset/data.csv
```

## 5)	To find the size of file, amount of space and disk usage. 

Display the file size
```
$ hadoop fs -du /user/testuser/hospcode_dataset/data.csv
$ hadoop fs -du -h /user/testuser/hospcode_dataset/data.csv
```

Display the amount of space
```
$ hadoop fs -du -h /user/testuser/
$ hadoop fs -du -h -s /user/testuser/
```

Show the capacity, free and used space of the filesystem
```
$ hadoop fs -df -h
```

## 6)	Backup the HDFS data using HDFS Snapshot. 

Create the demo directory
```
$ hadoop fs -mkdir /user/testuser/snapshot_demo
$ hadoop fs -touchz /user/testuser/snapshot_demo/text1.txt
$ echo "Hello World for Snapshot" > text2.txt
$ hadoop fs -put text2.txt /user/testuser/snapshot_demo/
$ hadoop fs -ls /user/testuser/snapshot_demo
```

Enable snapshot directory
```
$ hdfs dfsadmin -allowSnapshot /user/testuser/snapshot_demo/
$ hdfs lsSnapshottableDir
```

Create snapshot on demo directory
```
$ hdfs dfs -createSnapshot /user/testuser/snapshot_demo
```

## 7)	Restore data from snapshot

Check the snapshot
```
$ hadoop fs -ls /user/testuser/snapshot_demo/.snapshot
```

Remove the file
```
$ hadoop fs -rm /user/testuser/snapshot_demo/text2.txt
$ hadoop fs -ls /user/testuser/snapshot_demo/
```

Restore from snapshot
```
$ hadoop fs -cp -ptopax /user/testuser/snapshot_demo/.snapshot/sXXXXXXXX-XXXXXX.XXX/text2.txt /user/testuser/snapshot_demo/
```

Note:
```
XXXXXXXX-XXXXXX.XXX depend on your drive
-ptopax is restored file timestamp to original
```

Check the restored files
```
$ hadoop fs -ls /user/testuser/snapshot_demo/
$ hadoop fs -cat /user/testuser/snapshot_demo/text2.txt
```

## 8)	Delete and Disable snapshot

Delete the snapshot 
```
$ hadoop fs -deleteSnapshot /user/testuser/snapshot_demo sXXXXXXXX-XXXXXX.XXX
```

Disable snapshot
```
$ hdfs dfsadmin -disallowSnapshot /user/testuser/snapshot_demo/
$ hdfs lsSnapshottableDir
```

## 9) Stop HDFS and Yarn
```
$ stop-dfs.sh
$ stop-yarn.sh
```

## HDFS commands
```
https://github.com/san089/Cloudera_Material/blob/master/hadoop-hdfs-commands-cheatsheet.pdf
```
