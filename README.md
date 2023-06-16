# WordCount to UrlCount

Taking WordCount (an existing Hadoop application that is extensively described in the [Hadoop tutorial](https://hadoop.apache.org/docs/r3.0.3/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)) we can modify it into UrlCount. I have used the [Hadoop streaming API](https://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/) to implement the solution in Python.

Step 1: Created the URLCount tool using a Coding environment (IDE) - used since that's easier (and cheaper) environment to debug my code.
Step 2: Run my URLCount implementation using Google's `dataproc`.

### Technologies: 
1) Git
2) Hadoop (v3.3.4 on local IDE and v3.2.3 on gcloud)
3) Python3

### Commands
- `make prepare`: Rule to copy the wikipedia articles to HDFS in the `input` directory. When running on the Coding environment, this will use files in local directory; when using the `dataproc` environment, this will use files on the Hadoop filesystem (HDFS)
- `make`: Compiles and creates the output `WordCount1.jar` file.
- `make prepare`: Copies two files from Wikipedia that are used as reference input. The files are placed in directory `input`.
- `make run`: Executes the `WordCount1` application, leaving the output in directory `output`.
- `make stream`: Run the program using Hadoop Streaming API
- `make filesystem`: Creates an HDFS entry for your user id (required for dataproc environment, prior to copying the WIkipedia articles on dataproc).

### Observations Using Hadoop Streaming API
#### Brief description of my solution:
I first executed and observed the output for the starter code that was provided to us. I referred to https://www.geeksforgeeks.org/python-check-url-string/ to make changes to my Mapper that contains the regular expression `href=\".*\"`. Next, I made changes to my reducer to incude the condition where the program will produce output only when count of the URL's would be > 5. Once the changes were done, I tested this code on csel.io and pushed the final code to GitHub. I had to make some changes to Makefile for the Hadoop version and jar file path - this was to ensure that the code runs successfully on gcloud. I took the QwikLab tutorial on Dataproc to understand how to create clusters and modify worker nodes. Then I opened gcloud again to run my files, so I started with generating the ssh keys and cloning the GitHub repo. I ran the code first using 2 worker nodes, and then again on 4 worker nodes. I was following a sequence of commands: 1) `make filesystem` to get the HDFS filesystem ready, 2) `make prepare` to get the input files, 3) I went through a phase of debugging my code so I was also using `make stream` to get output, `hdfs dfs -cat stream-output/*` to view output, and `hdfs dfs -rm -r stream-output` to clear the previous output data, 4) `time make stream` to get output and the time taken to produce the output, 5) `gcloud dataproc clusters update cluster-lab2 --num-workers 4` to update the number of worker nodes from 2 to 4.

#### The Java WordCount implementation used a Combiner to improve efficiency, but that may cause problems for this application and produce a different output. An explaination of why this would be the case.

A Combiner generally helps in reducing the data load from Mapper to the Reducer. Here, the Combiner works like the Reducer, and takes into account the condition `count of the URL's > 5`. Now, if we consider an example like the url `"/wiki/MapReduce"` which has a total 6 occureneces, it can get combined into one mapper with a count of 2 and get combined in another mapper with a count of 4. Before it reaches the reducer, it's checking if the count of the URL > 5 and it fails because the count is less on that specific node, even though the total count for that URL is 6. Hence, the count of the URL may get affected if we use a Combiner.

#### Comparison of the 2-node and 4-node execution times.
The execution time when using 1 master and 2 worker nodes was as in the following screenshot:
![image](https://user-images.githubusercontent.com/94497779/189245701-6f44f20f-b900-456e-8267-b3d8aa667e01.png)

The execution time when using 1 master and 4 worker nodes was as in the following screenshot:
![image](https://user-images.githubusercontent.com/94497779/189245731-ababa070-b9a3-463f-bed2-9764672f0ec4.png)

From the above execuation times, we observe that increasing the number of worker nodes decreased the overall time of execution.

Following is the output that I recieved when I executed the program using 2 worker nodes as well as using 4 worker nodes:
![image](https://user-images.githubusercontent.com/94497779/189245996-c5280f2e-36eb-44df-b076-a04878bf76cd.png)





