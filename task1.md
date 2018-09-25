# Intro to big data management

Content
* Results 1-3
* WordCount code
* Cluster

# Problem 1:

I did the punctuation cleaning with simple regex:
`String row = value.toString().replaceAll("[^a-zA-Z ]", "").toLowerCase();`

The trailing and leading spaces are removed by `StringTokenizer` used in the original `WordCount.java

## Result 1.1:

TOP10 without “the”, “am”, “is”, and “are”?

```
and 911
a 810
to 795
of 603
you 574
it 570
she 525
i 510
said 470
in 464
```

## Result 1.2:

TOP5 after "the"

```
a 810
to 795
of 603
you 574
it 57
```

> Note: I had the whole txt file in processing for problems 1 & 2

## Result 1.3:

For the third task I edited the input file so that:
* Each sentence on one row
* Removed all punctuation

```
of the 101
said the 99
in a 97
in the 90
as she 82
you know 74
a little 72
said alice 67
the queen 67
and the 67
to the 66
the red 65
she said 64
it was 62
went on 59
```

## WordCount

WordCount.java

```
import java.io.BufferedReader;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.util.*;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

    public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private String word = "";
        private String nextWord = "";

        public void map(Object key, Text value, Context context
        ) throws IOException, InterruptedException {
            String row = value.toString().replaceAll("[^a-zA-Z ]", "").toLowerCase();
            //System.out.println("Content: " + row);
            StringTokenizer itr = new StringTokenizer(row);
            while (itr.hasMoreTokens()) {
                if (nextWord.length() > 0) {
                    word = nextWord;
                } else {
                    word = itr.nextToken();
                }
                if (itr.hasMoreTokens()) {
                    nextWord = itr.nextToken();
                    context.write(new Text(word + " " + nextWord), one);
                } else {
                    //out of pairs, moving on
                }
            }
        }
    }

    public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                           Context context
        ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static class Sorter {
        public static void printTop15(Path path) {

            String fileName = "/Users/juhapekkam/Documents/Studies/IntroBigData/Task1/" + path.toString() +
                    "/part-r-00000";
            System.out.println("filename: " + fileName);
            String line = null;
            HashMap<String, Integer> results = new HashMap<>();

            try {
                FileReader fileReader = new FileReader(fileName);
                BufferedReader bufferedReader = new BufferedReader(fileReader);

                while((line = bufferedReader.readLine()) != null) {
                    String[] split = line.split("\t");
                    results.put(split[0], Integer.parseInt(split[1]));
                }

                TreeMap<String, Integer> tree = SortMapByValue.sortMapByValue(results);
                Iterator<String> iterator = tree.keySet().iterator();

                for (int i=0; i<15; i++) {
                    String word = iterator.next();
                    System.out.println(word + " " + results.get(word));
                }

                bufferedReader.close();
            }
            catch(FileNotFoundException ex) {
                System.out.println(
                        "Unable to open file '" +
                                fileName + "'");
            }
            catch(IOException ex) {
                System.out.println(
                        "Error reading file '"
                                + fileName + "'");
            }
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        Path path = new Path(args[1] + (int) Math.floor(Math.random() * 1000));
        FileOutputFormat.setOutputPath(job, path);
        boolean jobSuccessfull = job.waitForCompletion(true);
        if (jobSuccessfull) {
            Sorter.printTop15(path);
        }
        System.exit(jobSuccessfull ? 0 : 1);
    }
}
```

SortMapByValue.java

```
import java.util.Comparator;
import java.util.HashMap;
import java.util.TreeMap;

public class SortMapByValue {

    public static TreeMap<String, Integer> sortMapByValue(HashMap<String, Integer> map){
        Comparator<String> comparator = new ValueComparator(map);
        //TreeMap is a map sorted by its keys.
        //The comparator is used to sort the TreeMap by keys.
        TreeMap<String, Integer> result = new TreeMap<String, Integer>(comparator);
        result.putAll(map);
        return result;
    }
}

// a comparator that compares Strings
class ValueComparator implements Comparator<String>{

    HashMap<String, Integer> map = new HashMap<String, Integer>();

    public ValueComparator(HashMap<String, Integer> map){
        this.map.putAll(map);
    }

    @Override
    public int compare(String s1, String s2) {
        if(map.get(s1) >= map.get(s2)){
            return -1;
        }else{
            return 1;
        }
    }
}
```

## Cluster

I managed to get the cluster up and running but the WordCount program fails if slave nodes are started.
WordCount works on the master node if slaves are stopped. I think I had it working once but I didn't save the logs.
The next day I could not get it working anymore.
I did not get the slaves working despite extensive debugging.
Here are some logs:

Successfull program on master:

```
jpm@jpm-VirtualBox:~/Projects$ hadoop jar wc.jar WordCount /input /outputSingle
18/09/25 20:00:43 INFO client.RMProxy: Connecting to ResourceManager at /192.168.56.101:8032
18/09/25 20:00:44 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/09/25 20:00:45 INFO input.FileInputFormat: Total input paths to process : 1
18/09/25 20:00:45 WARN hdfs.DFSClient: Caught exception
java.lang.InterruptedException
	at java.lang.Object.wait(Native Method)
	at java.lang.Thread.join(Thread.java:1252)
	at java.lang.Thread.join(Thread.java:1326)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.closeResponder(DFSOutputStream.java:716)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.endBlock(DFSOutputStream.java:476)
	at org.apache.hadoop.hdfs.DFSOutputStream$DataStreamer.run(DFSOutputStream.java:652)
18/09/25 20:00:45 INFO mapreduce.JobSubmitter: number of splits:1
18/09/25 20:00:45 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1537894742897_0001
18/09/25 20:00:45 INFO impl.YarnClientImpl: Submitted application application_1537894742897_0001
18/09/25 20:00:45 INFO mapreduce.Job: The url to track the job: http://192.168.56.101:8088/proxy/application_1537894742897_0001/
18/09/25 20:00:45 INFO mapreduce.Job: Running job: job_1537894742897_0001
18/09/25 20:00:56 INFO mapreduce.Job: Job job_1537894742897_0001 running in uber mode : false
18/09/25 20:00:56 INFO mapreduce.Job:  map 0% reduce 0%
18/09/25 20:01:02 INFO mapreduce.Job:  map 100% reduce 0%
18/09/25 20:01:08 INFO mapreduce.Job:  map 100% reduce 100%
18/09/25 20:01:09 INFO mapreduce.Job: Job job_1537894742897_0001 completed successfully
18/09/25 20:01:09 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=95505
		FILE: Number of bytes written=435953
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=189404
		HDFS: Number of bytes written=69183
		HDFS: Number of read operations=6
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=3280
		Total time spent by all reduces in occupied slots (ms)=3610
		Total time spent by all map tasks (ms)=3280
		Total time spent by all reduce tasks (ms)=3610
		Total vcore-milliseconds taken by all map tasks=3280
		Total vcore-milliseconds taken by all reduce tasks=3610
		Total megabyte-milliseconds taken by all map tasks=3358720
		Total megabyte-milliseconds taken by all reduce tasks=3696640
	Map-Reduce Framework
		Map input records=4306
		Map output records=32318
		Map output bytes=314883
		Map output materialized bytes=95505
		Input split bytes=106
		Combine input records=32318
		Combine output records=6695
		Reduce input groups=6695
		Reduce shuffle bytes=95505
		Reduce input records=6695
		Reduce output records=6695
		Spilled Records=13390
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=102
		CPU time spent (ms)=1260
		Physical memory (bytes) snapshot=329965568
		Virtual memory (bytes) snapshot=3910578176
		Total committed heap usage (bytes)=170004480
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=189298
	File Output Format Counters
		Bytes Written=69183
 ```
 
 Failing for unresolved hostname:
 
 ```
 jpm@jpm-VirtualBox:~/Projects$ hadoop jar wc.jar WordCount /input /outputMulti
18/09/25 20:03:09 INFO client.RMProxy: Connecting to ResourceManager at /192.168.56.101:8032
18/09/25 20:03:09 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/09/25 20:03:10 INFO input.FileInputFormat: Total input paths to process : 1
18/09/25 20:03:10 INFO mapreduce.JobSubmitter: number of splits:1
18/09/25 20:03:10 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1537894742897_0002
18/09/25 20:03:10 INFO impl.YarnClientImpl: Submitted application application_1537894742897_0002
18/09/25 20:03:10 INFO mapreduce.Job: The url to track the job: http://192.168.56.101:8088/proxy/application_1537894742897_0002/
18/09/25 20:03:10 INFO mapreduce.Job: Running job: job_1537894742897_0002
18/09/25 20:03:19 INFO mapreduce.Job: Job job_1537894742897_0002 running in uber mode : false
18/09/25 20:03:19 INFO mapreduce.Job:  map 0% reduce 0%
18/09/25 20:03:25 INFO mapreduce.Job:  map 100% reduce 0%
18/09/25 20:03:26 INFO mapreduce.Job: Task Id : attempt_1537894742897_0002_r_000000_0, Status : FAILED
Container launch failed for container_1537894742897_0002_01_000003 : java.lang.IllegalArgumentException: java.net.UnknownHostException: hadoobuntu2
	at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:377)
	at org.apache.hadoop.security.SecurityUtil.setTokenService(SecurityUtil.java:356)
	at org.apache.hadoop.yarn.util.ConverterUtils.convertFromYarn(ConverterUtils.java:238)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.newProxy(ContainerManagementProtocolProxy.java:266)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.<init>(ContainerManagementProtocolProxy.java:244)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy.getProxy(ContainerManagementProtocolProxy.java:129)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl.getCMProxy(ContainerLauncherImpl.java:409)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$Container.launch(ContainerLauncherImpl.java:138)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$EventProcessor.run(ContainerLauncherImpl.java:375)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.UnknownHostException: hadoobuntu2
	... 12 more

18/09/25 20:03:29 INFO mapreduce.Job: Task Id : attempt_1537894742897_0002_r_000000_1, Status : FAILED
Container launch failed for container_1537894742897_0002_01_000004 : java.lang.IllegalArgumentException: java.net.UnknownHostException: hadoobuntu2
	at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:377)
	at org.apache.hadoop.security.SecurityUtil.setTokenService(SecurityUtil.java:356)
	at org.apache.hadoop.yarn.util.ConverterUtils.convertFromYarn(ConverterUtils.java:238)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.newProxy(ContainerManagementProtocolProxy.java:266)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.<init>(ContainerManagementProtocolProxy.java:244)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy.getProxy(ContainerManagementProtocolProxy.java:129)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl.getCMProxy(ContainerLauncherImpl.java:409)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$Container.launch(ContainerLauncherImpl.java:138)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$EventProcessor.run(ContainerLauncherImpl.java:375)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.UnknownHostException: hadoobuntu2
	... 12 more

18/09/25 20:03:32 INFO mapreduce.Job: Task Id : attempt_1537894742897_0002_r_000000_2, Status : FAILED
Container launch failed for container_1537894742897_0002_01_000005 : java.lang.IllegalArgumentException: java.net.UnknownHostException: hadoobuntu2
	at org.apache.hadoop.security.SecurityUtil.buildTokenService(SecurityUtil.java:377)
	at org.apache.hadoop.security.SecurityUtil.setTokenService(SecurityUtil.java:356)
	at org.apache.hadoop.yarn.util.ConverterUtils.convertFromYarn(ConverterUtils.java:238)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.newProxy(ContainerManagementProtocolProxy.java:266)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy$ContainerManagementProtocolProxyData.<init>(ContainerManagementProtocolProxy.java:244)
	at org.apache.hadoop.yarn.client.api.impl.ContainerManagementProtocolProxy.getProxy(ContainerManagementProtocolProxy.java:129)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl.getCMProxy(ContainerLauncherImpl.java:409)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$Container.launch(ContainerLauncherImpl.java:138)
	at org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImpl$EventProcessor.run(ContainerLauncherImpl.java:375)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Caused by: java.net.UnknownHostException: hadoobuntu2
	... 12 more

18/09/25 20:03:36 INFO mapreduce.Job:  map 100% reduce 100%
18/09/25 20:03:36 INFO mapreduce.Job: Job job_1537894742897_0002 failed with state FAILED due to: Task failed task_1537894742897_0002_r_000000
Job failed as tasks failed. failedMaps:0 failedReduces:1

18/09/25 20:03:36 INFO mapreduce.Job: Counters: 38
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=218003
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=189404
		HDFS: Number of bytes written=0
		HDFS: Number of read operations=3
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=0
	Job Counters
		Failed reduce tasks=4
		Launched map tasks=1
		Launched reduce tasks=4
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=3394
		Total time spent by all reduces in occupied slots (ms)=2
		Total time spent by all map tasks (ms)=3394
		Total time spent by all reduce tasks (ms)=2
		Total vcore-milliseconds taken by all map tasks=3394
		Total vcore-milliseconds taken by all reduce tasks=2
		Total megabyte-milliseconds taken by all map tasks=3475456
		Total megabyte-milliseconds taken by all reduce tasks=2048
	Map-Reduce Framework
		Map input records=4306
		Map output records=32318
		Map output bytes=314883
		Map output materialized bytes=95505
		Input split bytes=106
		Combine input records=32318
		Combine output records=6695
		Spilled Records=6695
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=66
		CPU time spent (ms)=600
		Physical memory (bytes) snapshot=215150592
		Virtual memory (bytes) snapshot=1949552640
		Total committed heap usage (bytes)=137433088
	File Input Format Counters
		Bytes Read=189298
 ```
 
 Ping is working thou:
 
```
jpm@jpm-VirtualBox:~/Projects$ ping hadoobuntu2
PING hadoobuntu2. (192.168.56.102) 56(84) bytes of data.
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=1 ttl=64 time=0.253 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=2 ttl=64 time=0.517 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=3 ttl=64 time=0.665 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=4 ttl=64 time=0.343 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=5 ttl=64 time=0.369 ms
^C
--- hadoobuntu2. ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4090ms
rtt min/avg/max/mdev = 0.253/0.429/0.665/0.146 ms
jpm@jpm-VirtualBox:~/Projects$ ping hadoobuntu2.
PING hadoobuntu2. (192.168.56.102) 56(84) bytes of data.
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=1 ttl=64 time=0.416 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=2 ttl=64 time=0.428 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=3 ttl=64 time=0.385 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=4 ttl=64 time=0.676 ms
64 bytes from hadoobuntu2. (192.168.56.102): icmp_seq=5 ttl=64 time=0.469 ms
^C
--- hadoobuntu2. ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4059ms
rtt min/avg/max/mdev = 0.385/0.474/0.676/0.107 ms
jpm@jpm-VirtualBox:~/Projects$ ping hadoobuntu3
PING hadoobuntu3. (192.168.56.103) 56(84) bytes of data.
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=1 ttl=64 time=0.296 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=2 ttl=64 time=0.345 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=3 ttl=64 time=0.389 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=4 ttl=64 time=0.676 ms
^C
--- hadoobuntu3. ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3060ms
rtt min/avg/max/mdev = 0.296/0.426/0.676/0.149 ms
jpm@jpm-VirtualBox:~/Projects$ ping hadoobuntu3.
PING hadoobuntu3. (192.168.56.103) 56(84) bytes of data.
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=1 ttl=64 time=0.314 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=2 ttl=64 time=0.670 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=3 ttl=64 time=0.416 ms
64 bytes from hadoobuntu3. (192.168.56.103): icmp_seq=4 ttl=64 time=0.300 ms
^C
--- hadoobuntu3. ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3019ms
rtt min/avg/max/mdev = 0.300/0.425/0.670/0.148 ms
``` 

host are set up also:

```
jpm@jpm-VirtualBox:~/Projects2$ cat /etc/hosts
127.0.0.1	localhost
127.0.1.1	jpm-VirtualBox
192.168.56.102  hadoobuntu2. hadoobuntu2
192.168.56.103  hadoobuntu3. hadoobuntu3

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Failing on something different after just reboot:
 
```
jpm@jpm-VirtualBox:~/Projects$ hadoop jar wc.jar WordCount /input /outputMulti2
18/09/25 20:15:54 INFO client.RMProxy: Connecting to ResourceManager at /192.168.56.101:8032
18/09/25 20:15:55 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
18/09/25 20:15:55 INFO input.FileInputFormat: Total input paths to process : 1
18/09/25 20:15:55 INFO mapreduce.JobSubmitter: number of splits:1
18/09/25 20:15:56 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1537894742897_0003
18/09/25 20:15:56 INFO impl.YarnClientImpl: Submitted application application_1537894742897_0003
18/09/25 20:15:56 INFO mapreduce.Job: The url to track the job: http://192.168.56.101:8088/proxy/application_1537894742897_0003/
18/09/25 20:15:56 INFO mapreduce.Job: Running job: job_1537894742897_0003
18/09/25 20:16:04 INFO mapreduce.Job: Job job_1537894742897_0003 running in uber mode : false
18/09/25 20:16:04 INFO mapreduce.Job:  map 0% reduce 0%
18/09/25 20:16:17 INFO mapreduce.Job:  map 100% reduce 0%
18/09/25 20:16:17 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_m_000000_0, Status : FAILED
18/09/25 20:16:18 INFO mapreduce.Job:  map 0% reduce 0%
18/09/25 20:16:32 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_m_000000_1, Status : FAILED
18/09/25 20:16:47 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_m_000000_2, Status : FAILED
18/09/25 20:16:54 INFO mapreduce.Job:  map 100% reduce 0%
18/09/25 20:17:07 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_r_000000_0, Status : FAILED
18/09/25 20:17:22 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_r_000000_1, Status : FAILED
18/09/25 20:17:37 INFO mapreduce.Job: Task Id : attempt_1537894742897_0003_r_000000_2, Status : FAILED
18/09/25 20:17:53 INFO mapreduce.Job:  map 100% reduce 100%
18/09/25 20:17:53 INFO mapreduce.Job: Job job_1537894742897_0003 failed with state FAILED due to: Task failed task_1537894742897_0003_r_000000
Job failed as tasks failed. failedMaps:0 failedReduces:1

18/09/25 20:17:53 INFO mapreduce.Job: Counters: 40
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=218004
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=189404
		HDFS: Number of bytes written=0
		HDFS: Number of read operations=3
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=0
	Job Counters
		Failed map tasks=3
		Failed reduce tasks=4
		Launched map tasks=4
		Launched reduce tasks=4
		Other local map tasks=3
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=41405
		Total time spent by all reduces in occupied slots (ms)=48108
		Total time spent by all map tasks (ms)=41405
		Total time spent by all reduce tasks (ms)=48108
		Total vcore-milliseconds taken by all map tasks=41405
		Total vcore-milliseconds taken by all reduce tasks=48108
		Total megabyte-milliseconds taken by all map tasks=42398720
		Total megabyte-milliseconds taken by all reduce tasks=49262592
	Map-Reduce Framework
		Map input records=4306
		Map output records=32318
		Map output bytes=314883
		Map output materialized bytes=95505
		Input split bytes=106
		Combine input records=32318
		Combine output records=6695
		Spilled Records=6695
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=71
		CPU time spent (ms)=720
		Physical memory (bytes) snapshot=213700608
		Virtual memory (bytes) snapshot=1949552640
		Total committed heap usage (bytes)=137433088
	File Input Format Counters
		Bytes Read=189298
``` 


