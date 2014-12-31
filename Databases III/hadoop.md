# Hadoop
========

## Introduction

Volume, Variety and Velocity are 3 problems that relational databases have trouble with. Hadoop tries to solve this with HDFS, in which it uses distributed data, redundancy and namenodes to guarantee uptime and fast computation

## HDFS

### Intro

1 Namenode and multiple datanodes.

Data is distributed in blocks of (default) 64MB.

Data is duplicated on the drives to reduce problems on hardware failure.

Namenode has duplicates too.

### Commands

View contents of HDFS

    hadoop fs -ls

Put file in HDFS

    hadoop fs -put file

A few bash commands work in the HDFS:

    hadoop fs -command
    tail
    cat
    cp
    mv
    rm
    mkdir
    du

To check the health and duplication status of the HDFS

    hdfs dfsadmin -report

To modify block sizes

    src/hadoop-common-project/hadoop-common/src/test/resources/core-site.xml

## MapReduce

Mapreduce is an algorithm designed for distributed file systems, as it is better to do calculations at the same time on multiple drives (map) and then after that combine them to return it (reduce)

### Example

A group of 6 people (6 blocks) each hold 5 cards (blocksize 5), they want to know how many cards of suit hearts there are.

They go through their own cards and write on notes the useful information of every card (only the suit); let's say they note only the suit and the number 1 (as this is a key/value pair and will be clear when we program this). Everyone now has 5 notes with a list of suits in their cards.

So a person holding a 6 of diamonds, a 2 of clubs, a 9 of hearts, a queen of hearts and a jack of spades will have the following notes:

    diamonds 1
    clubs 1
    hearts 1
    hearts 1
    spades 1

Now they all split their notes on the key (the suit) and give the stack of notes to the appropriate guy. So there will be 4 guys (reducers), each receiving all notes of the same suit.

The 4 guys (reducers) now count the values and sum these and announce these. Now the MapReduce is finished.

To put it in a diagram: 
#### Map
![Map](images/map.png)
#### Reduce
![Reduce](images/reduce.png)
#### Result
![Result](images/result.png)

### In Hadoop

Now we will make custom mapper and reducer classes. We will do this through eclipse. 

Don't forget to add the required libraries:

    /usr/lib/hadoop/hadoop-common-2.0.0-cdh4.1.1.jar
    /usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core-2.0.0-cdh4.1.1.jar
    /usr/lib/hadoop-mapreduce/hadoop-mapreduce-jobclient-2.0.0-cdh4.1.1.jar
    /usr/lib/hadoop/lib/commons-cli-1.2.jar
    /usr/lib/hadoop/lib/commons-logging-1.1.1.jar

Now we can create a mapper class by extending `org.apache.hadoop.mapreduce.Mapper`.  
Example mapper class:

    public class MyMapper extends Mapper<Object, Text, Text, IntWritable> {
        @Override
        public void map(Object key, Text value, Context output) throws IOException, InterruptedException {
            String line = value.toString(); // Getting the line as a String
            String[] words = line.split(" "); // Splitting the line on a space, in an array
            output.write(new Text(words[0]), new IntWritable(1)); // Output the required results
        }
    }

The Mapper class has 4 type parameters: input key type, input value type, output key type, output value type.

We can create a reducer class similarly by extending `org.apache.hadoop.mapreduce.Reducer`.  
Example reducer class:

    public class MyReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        @Override
        public void reduce(Text key, Iterable<IntWritable> values, Context output) throws IOException, InterruptedException {
            int count = 0;
            for(IntWritable value : values){
                count += value.get(); // Sum all values
            }
            output.write(key, new IntWritable(count)); 
        }
    }

The Reducer class has the same type parameters as the Mapper class.

Now we only need to make an Application class by extending `org.apache.hadoop.conf.Configured` and implementing `org.apache.hadoop.util.Tool`.  
Example Application class:

    public class MyApplication extends Configured implements Tool{ 
        public static void main(String[] args) throws Exception {
            int res = ToolRunner.run(new Configuration(), new MyApplication(), args);
            System.exit(res);
        }

        @Override
        public int run(String[] args) throws Exception {
            if (args.length != 2) {
                System.out.println("usage: [input] [output]");
                System.exit(-1);
            }
            Job job = Job.getInstance(new Configuration());

Set the outputtypes  

            job.setOutputKeyClass(Text.class); 
            job.setOutputValueClass(IntWritable.class);

Set the classes  

            job.setMapperClass(MyMapper.class); 
            job.setReducerClass(MyReducer.class);

Set the formats  

            job.setInputFormatClass(TextInputFormat.class); 
            job.setOutputFormatClass(TextOutputFormat.class);

Set the files  

            FileInputFormat.setInputPaths(job, new Path(args[0])); 
            FileOutputFormat.setOutputPath(job, new Path(args[1]));

Run the actual job  

            job.setJarByClass(MyApplication.class);
            job.submit();
            return 0; 
        }
    }


