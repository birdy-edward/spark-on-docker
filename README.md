# Pyspark and Selenium on Docker
---------

*I updated docker-compose for these two processes. Run **<code>docker-compose up</code>**, all of data will be collected and loaded into Azure SQL Server.*

---------

**1. Selenium on Docker**

<img title="a title" alt="Alt text" src="/images/Selenium_Folder.png">

This folder contains code to run a crawler that get data from Soundcloud.com and save it into data folder. The crawler runs on Docker and data generated in a built container is mounted from container to local desktop with the docker-compose.yml .

There are steps to run app:
- <code>docker build -t crawler-img .</code>
- <code>docker run crawler-img</code>

When the contain stopped and the screen display "Crawling Successful", the data folder including Soundcloud_User.csv will exist. The file after crawling is mounted to <code>../Pyspark ETL/data/</code>

**2. Pyspark on Docker**

<img title="a title" alt="Alt text" src="/images/Spark_Folder.png">

When having had the Soundcloud_User.csv, I will do an ETL process to transform and load data to SoundCloudUser table in Azure SQL server. The origin folder has a (.env) folder containing credential infomation of SQL server (password, username,connection_string), so I have to hide it from repository.

Steps to run this app:
- Configure the essential infomation in .env file
- <code>docker build -t etl-img .</code>
- <code>docker run etl-img</code>

When all of data in Soundcloud_User.csv tranformed and loaded to database, the notification "Successful" will be displayed.

<img title="a title" alt="Alt text" src="/images/DataFile.png">
<p>
        <em style="text-align: center;">Data in SoundCloud_User.csv</em>
</p>



<img title="a title" alt="Alt text" src="/images/Data in database.png">

<em>Data in  after ETL</em>


**3. Execute ETL piple with Shell Script**

I have added new shell script `spark_submit.sh` to combine all commands needed to build up Spark Master&Worker and submit ETL pipeline to master node

There are 2 Spark deploying modes: **Spark Client** and **Spark Cluster**

**Spark Client:**
- Driver runs on a dedicated server (Master node) inside a dedicated process
- Drive opens up a dedicated Netty HTTP server and distrubutes operational JAR files across all Worker nodes
- Because the Master's node is in your own, it do not need to spend any resource in Spark cluster for Driver program
- Suits for live debugging and live notebook

**Spark Cluster:**
- Driver runs on one of the cluster's Worker nodes. The worker is chosen by the Master leader
- Driver runs as a dedicated, standalone process inside the Worker.
- Driver programs takes up at least 1 core and a dedicated amount of memory from one of the workers (this can be configured).
- Driver program can be monitored from the Master node using the --supervise flag and be reset in case it dies.
- When working in Cluster mode, all JARs related to the execution of your application need to be publicly available to all the workers. This means you can either manually place them in a shared place or in a folder for each of the workers.
- It benefits especially for long-lasting jobs with heavy workload which keeps running even when the drive is fallen because the drive would be reinstalled by then on available resources

In this projecs, I build up Spark Cluster in Client mode with specific Spark master after running `docker compose up -f ./docker/docker-compose-spark.yml -d`

Finally, submitting my ETL job to Spark Master, `spark_submit.sh` kind of is about to create a live session to Spark cluster, the job remains executing whenever this connection alive. In constrast, Cluster Mode is turned of if you destroy the connection

Steps:

- Open the project
- Run `chmod +x spark_submit.sh`
- <code>source ./spark_submit.sh</code>

