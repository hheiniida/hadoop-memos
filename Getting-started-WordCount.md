## WordCount instructions

### 1. Install hadoop

### 2. Install Java

### 3. Set env variables

Check current env varibles. Mine look like this:

```
jpm@jpm-VirtualBox:~/WordCount$ echo $JAVA_HOME 
/usr/lib/jvm/java-8-openjdk-amd64
jpm@jpm-VirtualBox:~/WordCount$ echo $PATH
/usr/lib/jvm/java-8-openjdk-amd64/bin:/usr/lib/jvm/java-8-openjdk-amd64//bin:/home/jpm/bin:/home/jpm/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
jpm@jpm-VirtualBox:~/WordCount$ echo $HADOOP_CLASSPATH
/usr/lib/jvm/java-8-openjdk-amd64/lib/tools.jar
```

Contents of my `$JAVA_HOME` look like this:

```
jpm@jpm-VirtualBox:~/WordCount$ ls /usr/lib/jvm/java-8-openjdk-amd64
ASSEMBLY_EXCEPTION  THIRD_PARTY_README  bin  docs  include  jre  lib  man  src.zip
```

You can set them temporalily with `EXPORT`:

```
export JAVA_HOME=/usr/java/default
export PATH=${JAVA_HOME}/bin:${PATH}
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```

### 4. Verify java and hadoop installations

Verify java by checking the version

```
jpm@jpm-VirtualBox:~/WordCount$ java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-0ubuntu0.16.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```
 
Verify hadoop by executing ´hadoop´ from binaries. Locations depends on installation.

```
jpm@jpm-VirtualBox:~$ ~/hadoop-2.7.7/bin/hadoop/
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  CLASSNAME            run the class named CLASSNAME
 or
  where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  jar <jar>            run a jar file
  ...
```

If you have hadoop binaries is `$PATH` you can just call `hadoop` instead of full path

```
jpm@jpm-VirtualBox:~$ export PATH=$PATH:~/hadoop-2.7.7/bin/
jpm@jpm-VirtualBox:~$ hadoop
Usage: hadoop [--config confdir] [COMMAND | CLASSNAME]
  CLASSNAME            run the class named CLASSNAME
 or
  where COMMAND is one of:
  fs                   run a generic filesystem user client
  version              print the version
  ...
```

### 5. Create WordCount.java
* Get source from here https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0
* Copy-paste to `~/WordCount/WordCount.java`

### 6. Compile:

There are two alternative methods

Method 1:

```
jpm@jpm-VirtualBox:~/WordCount2$ ls
WordCount.java
jpm@jpm-VirtualBox:~/WordCount2$ hadoop com.sun.tools.javac.Main WordCount.java
jpm@jpm-VirtualBox:~/WordCount2$ ls
WordCount$IntSumReducer.class  WordCount$TokenizerMapper.class  WordCount.class  WordCount.java
```

Three *.class files = success!

Method 2:

First check content of $(hadoop classpath)
```
jpm@jpm-VirtualBox:~/WordCount2$ echo $(hadoop classpath)
/home/jpm/hadoop-2.7.7/etc/hadoop:/home/jpm/hadoop-2.7.7/share/hadoop/common/lib/*:/home/jpm/hadoop-2.7.7/share/hadoop/common/*:/home/jpm/hadoop-2.7.7/share/hadoop/hdfs:/home/jpm/hadoop-2.7.7/share/hadoop/hdfs/lib/*:/home/jpm/hadoop-2.7.7/share/hadoop/hdfs/*:/home/jpm/hadoop-2.7.7/share/hadoop/yarn/lib/*:/home/jpm/hadoop-2.7.7/share/hadoop/yarn/*:/home/jpm/hadoop-2.7.7/share/hadoop/mapreduce/lib/*:/home/jpm/hadoop-2.7.7/share/hadoop/mapreduce/*:/usr/lib/jvm/java-8-openjdk-amd64/lib/tools.jar:/contrib/capacity-scheduler/*.jar
```

Then compile

```
javac WordCount.java -cp $(hadoop classpath)
```

### 7. Create jar-file

```
jpm@jpm-VirtualBox:~/WordCount2$ ls
WordCount$IntSumReducer.class  WordCount$TokenizerMapper.class  WordCount.class  WordCount.java  wc.jar
```

### 8. Prepare input-files

```
jpm@jpm-VirtualBox:~/WordCount2$ mkdir input
jpm@jpm-VirtualBox:~/WordCount2$ touch input/file01
jpm@jpm-VirtualBox:~/WordCount2$ touch input/file02
jpm@jpm-VirtualBox:~/WordCount2$ echo "Hello World Bye World" > input/file01
jpm@jpm-VirtualBox:~/WordCount2$ echo "Hello Hadoop Goodbye Hadoop" > input/file02
jpm@jpm-VirtualBox:~/WordCount2$ cat input/file0*
Hello World Bye World
Hello Hadoop Goodbye Hadoop
```

### 9. Check that hadoop can read your files

Both of these should show the contents of your root dir:

```
jpm@jpm-VirtualBox:~/WordCount2$ hadoop fs -ls /
jpm@jpm-VirtualBox:~/WordCount2$ ls -la /
```

And here you should see the files you just created:

```
jpm@jpm-VirtualBox:~/WordCount2$ hadoop fs -ls /home/jpm/WordCount/input
Found 2 items
-rw-rw-r--   1 jpm jpm         22 2018-09-11 17:37 /home/jpm/WordCount/input/file01
-rw-rw-r--   1 jpm jpm         29 2018-09-11 17:37 /home/jpm/WordCount/input/file02
```

### 10. Run the program!

```
jpm@jpm-VirtualBox:~/WordCount2$ ls
WordCount$IntSumReducer.class  WordCount$TokenizerMapper.class  WordCount.class  WordCount.java  input  wc.jar
```

```
jpm@jpm-VirtualBox:~/WordCount2$ hadoop jar wc.jar WordCount /home/jpm/WordCount2/input /home/jpm/WordCount2/output
```

(Execution log attached)

Syntax:
```
hadoop = program to run
jar = option to run a jar-file
wc.jar = path to jar-file
WordCount = java class name to run
/home/jpm/WordCount2/input = input file directory
/home/jpm/WordCount2/output = output file directory (will be created by program)
```

### 11. Check the results

```
jpm@jpm-VirtualBox:~/WordCount2$ ls output
_SUCCESS  part-r-00000
jpm@jpm-VirtualBox:~/WordCount2$ cat output/part-r-00000 
Bye	1
Goodbye	1
Hadoop	2
Hello	2
World	2
```

### Wordcount execution log

Execution log:

```
18/09/12 10:57:25 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
18/09/12 10:57:25 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
18/09/12 10:57:25 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/09/12 10:57:25 INFO input.FileInputFormat: Total input paths to process : 2
18/09/12 10:57:25 INFO mapreduce.JobSubmitter: number of splits:2
18/09/12 10:57:25 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local206998102_0001
18/09/12 10:57:25 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
18/09/12 10:57:25 INFO mapred.LocalJobRunner: OutputCommitter set in config null
18/09/12 10:57:25 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
18/09/12 10:57:25 INFO mapreduce.Job: Running job: job_local206998102_0001
18/09/12 10:57:25 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
18/09/12 10:57:25 INFO mapred.LocalJobRunner: Waiting for map tasks
18/09/12 10:57:25 INFO mapred.LocalJobRunner: Starting task: attempt_local206998102_0001_m_000000_0
18/09/12 10:57:25 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
18/09/12 10:57:25 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
18/09/12 10:57:25 INFO mapred.MapTask: Processing split: file:/home/jpm/WordCount2/input/file02:0+28
18/09/12 10:57:25 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
18/09/12 10:57:25 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
18/09/12 10:57:25 INFO mapred.MapTask: soft limit at 83886080
18/09/12 10:57:25 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
18/09/12 10:57:25 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
18/09/12 10:57:25 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
18/09/12 10:57:25 INFO mapred.LocalJobRunner: 
18/09/12 10:57:25 INFO mapred.MapTask: Starting flush of map output
18/09/12 10:57:25 INFO mapred.MapTask: Spilling map output
18/09/12 10:57:25 INFO mapred.MapTask: bufstart = 0; bufend = 44; bufvoid = 104857600
18/09/12 10:57:25 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214384(104857536); length = 13/6553600
18/09/12 10:57:25 INFO mapred.MapTask: Finished spill 0
18/09/12 10:57:25 INFO mapred.Task: Task:attempt_local206998102_0001_m_000000_0 is done. And is in the process of committing
18/09/12 10:57:25 INFO mapred.LocalJobRunner: map
18/09/12 10:57:25 INFO mapred.Task: Task 'attempt_local206998102_0001_m_000000_0' done.
18/09/12 10:57:26 INFO mapred.Task: Final Counters for attempt_local206998102_0001_m_000000_0: Counters: 18
	File System Counters
		FILE: Number of bytes read=3376
		FILE: Number of bytes written=290601
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=1
		Map output records=4
		Map output bytes=44
		Map output materialized bytes=45
		Input split bytes=103
		Combine input records=4
		Combine output records=3
		Spilled Records=3
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=23
		Total committed heap usage (bytes)=137433088
	File Input Format Counters 
		Bytes Read=28
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local206998102_0001_m_000000_0
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Starting task: attempt_local206998102_0001_m_000001_0
18/09/12 10:57:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
18/09/12 10:57:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
18/09/12 10:57:26 INFO mapred.MapTask: Processing split: file:/home/jpm/WordCount2/input/file01:0+22
18/09/12 10:57:26 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
18/09/12 10:57:26 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
18/09/12 10:57:26 INFO mapred.MapTask: soft limit at 83886080
18/09/12 10:57:26 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
18/09/12 10:57:26 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
18/09/12 10:57:26 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
18/09/12 10:57:26 INFO mapred.LocalJobRunner: 
18/09/12 10:57:26 INFO mapred.MapTask: Starting flush of map output
18/09/12 10:57:26 INFO mapred.MapTask: Spilling map output
18/09/12 10:57:26 INFO mapred.MapTask: bufstart = 0; bufend = 38; bufvoid = 104857600
18/09/12 10:57:26 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26214384(104857536); length = 13/6553600
18/09/12 10:57:26 INFO mapred.MapTask: Finished spill 0
18/09/12 10:57:26 INFO mapred.Task: Task:attempt_local206998102_0001_m_000001_0 is done. And is in the process of committing
18/09/12 10:57:26 INFO mapred.LocalJobRunner: map
18/09/12 10:57:26 INFO mapred.Task: Task 'attempt_local206998102_0001_m_000001_0' done.
18/09/12 10:57:26 INFO mapred.Task: Final Counters for attempt_local206998102_0001_m_000001_0: Counters: 18
	File System Counters
		FILE: Number of bytes read=3623
		FILE: Number of bytes written=290673
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=1
		Map output records=4
		Map output bytes=38
		Map output materialized bytes=40
		Input split bytes=103
		Combine input records=4
		Combine output records=3
		Spilled Records=3
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=22
		Total committed heap usage (bytes)=184619008
	File Input Format Counters 
		Bytes Read=22
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local206998102_0001_m_000001_0
18/09/12 10:57:26 INFO mapred.LocalJobRunner: map task executor complete.
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Waiting for reduce tasks
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Starting task: attempt_local206998102_0001_r_000000_0
18/09/12 10:57:26 INFO output.FileOutputCommitter: File Output Committer Algorithm version is 1
18/09/12 10:57:26 INFO mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
18/09/12 10:57:26 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@17230b7b
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=363285696, maxSingleShuffleLimit=90821424, mergeThreshold=239768576, ioSortFactor=10, memToMemMergeOutputsThreshold=10
18/09/12 10:57:26 INFO reduce.EventFetcher: attempt_local206998102_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
18/09/12 10:57:26 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local206998102_0001_m_000000_0 decomp: 41 len: 45 to MEMORY
18/09/12 10:57:26 INFO reduce.InMemoryMapOutput: Read 41 bytes from map-output for attempt_local206998102_0001_m_000000_0
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 41, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->41
18/09/12 10:57:26 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local206998102_0001_m_000001_0 decomp: 36 len: 40 to MEMORY
18/09/12 10:57:26 INFO reduce.InMemoryMapOutput: Read 36 bytes from map-output for attempt_local206998102_0001_m_000001_0
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 36, inMemoryMapOutputs.size() -> 2, commitMemory -> 41, usedMemory ->77
18/09/12 10:57:26 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
18/09/12 10:57:26 INFO mapred.LocalJobRunner: 2 / 2 copied.
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: finalMerge called with 2 in-memory map-outputs and 0 on-disk map-outputs
18/09/12 10:57:26 INFO mapred.Merger: Merging 2 sorted segments
18/09/12 10:57:26 WARN io.ReadaheadPool: Failed readahead on ifile
EBADF: Bad file descriptor
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posix_fadvise(Native Method)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX.posixFadviseIfPossible(NativeIO.java:267)
	at org.apache.hadoop.io.nativeio.NativeIO$POSIX$CacheManipulator.posixFadviseIfPossible(NativeIO.java:146)
	at org.apache.hadoop.io.ReadaheadPool$ReadaheadRequestImpl.run(ReadaheadPool.java:206)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
18/09/12 10:57:26 INFO mapred.Merger: Down to the last merge-pass, with 2 segments left of total size: 61 bytes
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: Merged 2 segments, 77 bytes to disk to satisfy reduce memory limit
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: Merging 1 files, 79 bytes from disk
18/09/12 10:57:26 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
18/09/12 10:57:26 INFO mapred.Merger: Merging 1 sorted segments
18/09/12 10:57:26 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 69 bytes
18/09/12 10:57:26 INFO mapred.LocalJobRunner: 2 / 2 copied.
18/09/12 10:57:26 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
18/09/12 10:57:26 INFO mapred.Task: Task:attempt_local206998102_0001_r_000000_0 is done. And is in the process of committing
18/09/12 10:57:26 INFO mapred.LocalJobRunner: 2 / 2 copied.
18/09/12 10:57:26 INFO mapred.Task: Task attempt_local206998102_0001_r_000000_0 is allowed to commit now
18/09/12 10:57:26 INFO output.FileOutputCommitter: Saved output of task 'attempt_local206998102_0001_r_000000_0' to file:/home/jpm/WordCount2/output/_temporary/0/task_local206998102_0001_r_000000
18/09/12 10:57:26 INFO mapred.LocalJobRunner: reduce > reduce
18/09/12 10:57:26 INFO mapred.Task: Task 'attempt_local206998102_0001_r_000000_0' done.
18/09/12 10:57:26 INFO mapred.Task: Final Counters for attempt_local206998102_0001_r_000000_0: Counters: 24
	File System Counters
		FILE: Number of bytes read=3851
		FILE: Number of bytes written=290805
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Combine input records=0
		Combine output records=0
		Reduce input groups=5
		Reduce shuffle bytes=85
		Reduce input records=6
		Reduce output records=5
		Spilled Records=6
		Shuffled Maps =2
		Failed Shuffles=0
		Merged Map outputs=2
		GC time elapsed (ms)=0
		Total committed heap usage (bytes)=184619008
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Output Format Counters 
		Bytes Written=53
18/09/12 10:57:26 INFO mapred.LocalJobRunner: Finishing task: attempt_local206998102_0001_r_000000_0
18/09/12 10:57:26 INFO mapred.LocalJobRunner: reduce task executor complete.
18/09/12 10:57:26 INFO mapreduce.Job: Job job_local206998102_0001 running in uber mode : false
18/09/12 10:57:26 INFO mapreduce.Job:  map 100% reduce 100%
18/09/12 10:57:26 INFO mapreduce.Job: Job job_local206998102_0001 completed successfully
18/09/12 10:57:26 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=10850
		FILE: Number of bytes written=872079
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=2
		Map output records=8
		Map output bytes=82
		Map output materialized bytes=85
		Input split bytes=206
		Combine input records=8
		Combine output records=6
		Reduce input groups=5
		Reduce shuffle bytes=85
		Reduce input records=6
		Reduce output records=5
		Spilled Records=12
		Shuffled Maps =2
		Failed Shuffles=0
		Merged Map outputs=2
		GC time elapsed (ms)=45
		Total committed heap usage (bytes)=506671104
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=50
	File Output Format Counters 
		Bytes Written=53
```
