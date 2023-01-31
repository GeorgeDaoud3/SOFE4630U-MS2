# SOFE4630U-MS2
# Milestone 2: Data Processing Service (Dataflow)

##  Repository: 
[https://github.com/GeorgeDaoud3/SOFE4630U-MS2.git](https://github.com/GeorgeDaoud3/SOFE4630U-MS2.git)

## Objective:
1. Get familiar with Dataflow.
2. Understand MapReduce patterns.
3. Run batch and Stream Processing examples over MNIST dataset.

## Dataflow and a Simple Map Reduce Example:
In this section, you will learn about Dataflow, MapReduce pattern, and a word count as a MapReduce example. 
1. Watch this video about [https://www.youtube.com/watch?v=KalJ0VuEM7s](Google Cloud Dataflow).
2. Watch this video about [https://www.youtube.com/watch?v=JZiM-NsdiJo](MapReduce concepts).
3. Read this article about [https://www.analyticsvidhya.com/blog/2022/05/an-introduction-to-mapreduce-with-a-word-count-example/](implementing a word count example using the MapReduce patterns).

## Configure Dataflow
1. Open the GCP site.
2. Search for **Dataflow API**.
    
    ![](images/df1.jpg)
3. Then click **Enable** and Wait until the GCP get the service enabled for you.
    
    ![](images/df2.jpg)
4. To grant privileges for you project to use Dataflow, search for **Service Accounts**.
    
    ![](images/df3.jpg)
5. Create a new service account.
    
    ![](images/df4.jpg)
6. As the service name is a global identifier, it’s a good practice to use the project id as a prefix as **ProjectID-DFSA**, the project ID can be copied from the console.
    
    ![](images/df5.jpg)
7. Add **Compute Engine Service Agent**  and **Pub/Sub Admin** as roles to the service account.

    ![](images/df6.jpg)
8. Now, it’s time to install the python library of DataFlow
    ``` cmd
    pip install pip --upgrade
    pip install 'apache-beam[gcp]'
    ```

## Running the wordcount Example
1.	There already a set of examples exist within the python library folder. The following command will search for the file containg wordcount example within any subdirectory of the home directory (**~**) and print it.

    ``` cmd
    find ~ -name 'wordcount.py'
    ```
2.	The following command will copy the file to the home directory (Replace path with the **path** you got from the previous step).

    ``` cmd
    cp path  ~/wordcount.py
    ```
    
    ![](images/df7.jpg)
3. Open the file using the text editor. Now, let’s try to understand the python code. The user can send arguments to customize the processing. The first step is to parse those arguments. Lines 69 to 73 define the first argument which will be set using the option **--input** . It’s an optional argument and if not given, it will have the default value given in line 72. The second argument is set using **--output** option. It’s required (not optional) and thus, no default value is needed. After describing the arguments, line 79 will parse the arguments and return a dictionary (**known_args**) with two keys named as the **dest** parameter of the parsed arguments (**input** and **output**)   

    ![](images/df8.jpg)
4.	The pipeline is described from line 87 to 106. It’s instructions that will be given to a worker to execute. 
* Line 87 defines the root of the processing as a python variable **p**. 
* The first stage inserted after **p** ( using **|** operator) is given in line 90. It’s called **Read** which run a built-in function to read a text file. Note, when executed,  the output will be a list of lines (strings).  The pipeline till the first stage is saved into a python variable named **line**. 
* The statements from line 92 to 96 add three consequent stages. 
    * The first is called **Split** that splits each line into list of words using a custom class implemented in lines 50:63. 
    * The second is called **PairWithOne** that converts each word into a key/value pair which the key is the word and the value is 1. This stage is used a inline function that takes a word **x** and return the tuple **(x,1)**. 
    * The first two stages are Map operations which takes a single input and produce a single or multiple outputs.
    * The third stage is a Reduce stage that will combine tuples having the same key (word) and them apply the **sum** function over the values to generate a new tuple of a word as a key and the count as the value. 
* Line 102 append a new stage to the pipeline. The stage implements another Map operation that executes a customized function defined in lines 99 and 100 to convert each tuple to a string.
* A final stage is used to save the output (list of strings) to a text file.

    ![](images/df9.jpg)
5.	To check the code, we can execute it locally (at the GCP console) first be running the following command. As no input option is given, it will use the default value while the output will be saved in the home directory (current directory) with a prefix **outputs**
    
    ``` cmd
    python wordcount.py --output outputs
    ```
    
    you can display the output file(s) using
    
    ``` cmd
    ls outputs*
    ```
    
    you can open the output file(s) with the text editor or using the Linux command
    
    ``` cmd
    cat outputs* | more
    ```
6.	Now, let’s run it as a GCP service. First step is to get the project ID using and save it to an environment variable (**$PROJECT**). (Note: the variable is temporally and has to be created if the console/session is terminated)
    
    ``` cmd
    PROJECT=$(gcloud config list project --format "value(core.project)")
    echo $PROJECT
    ```
7.	As the input and output pathes should be globally accessed files, a folder created in Google Cloud Storage is needed to be accessed by the Dataflow service. Google Cloud Storage that acts as a File System is called Bucket. The following steps will lead you to create a Bucket.
    
    a) Search for **Buckets**

    ![](images/df10.jpg)
    
    b) Click **create**.

    ![](images/df11.jpg)
    
    c) As the name of the bucket is a unique global identifier, let’s use the project ID as a prefix as **ProjectID-bucket**. Then click **create**.

    ![](images/df12.jpg)
    
    d)	As only the service from our project will access the bucket, enable **public access prevention**.

    ![](images/df13.jpg)
    
    e)	Let’s got the console and create another environment variable for the bucket.
    
    ``` cmd
    BUCKET=gs://$PROJECT-bucket
    echo $BUCKET
    ```

    ![](images/df14.jpg)
8.	To run the pipeline using DataFlow, run the following command, 
    
    ``` cmd 
    python wordcount.py \
      --region northamerica-northeast2 \
      --runner DataflowRunner \
      --project $PROJECT \
      --temp_location $BUCKET/tmp/ \
      --input gs://dataflow-samples/shakespeare/winterstale.txt \
      --output $BUCKET/result/outputs \
      --experiment use_unsupported_python_version
    ```
    
    Some of the command arguments are needed by Dataflow to process the pipeline while others as input and output are used by the wordcount code to generate a customized pipeline. It will take minutes for Dataflow to generate worker nodes, configure them, execute the pipeline, and finally it will display **JOB_STATE_DONE**.

    ![](images/df14a.jpg)
9.	To see the details about Dataflow Job, search for **Dataflow Jobs**, then choose the first one in the list (last job). A diagram of the pipeline will be displayed in which each stage is named as instructed in the python file. Note that, the name is unique and can’t be repeated for different stages. Also, the job info and logs are shown which can be used to debug the job. 
    
    ![](images/df15.jpg)
10. Go to the bucket created in step 7 and open the file(s) started by the prefix **outputs** within a folder named **result**. Download the file to read it.
11.	In the GitHub repository, there is an upgrade to the wordcount script. Clone the repository and run the updated script. Try to understand its function.
    ```cmd 
    cd ~
    git clone https://github.com/GeorgeDaoud3/SOFE4630U-MS2.git
    cd ~/SOFE4630U-MS2/wordcount
    python wordcount2.py \
      --region northamerica-northeast2 \
      --runner DataflowRunner \
      --project $PROJECT \
      --temp_location $BUCKET/tmp/ \
      --input gs://dataflow-samples/shakespeare/winterstale.txt \
      --output $BUCKET/result/outputs \
      --output2 $BUCKET/result/outputs2 \
      --experiment use_unsupported_python_version
    ```
## NMIST dataset
