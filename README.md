# CITS5503 Project Submission

Following on from our proposal to our client, the decision has been agreed upon that the best course of action to future proof the business is to shift the central database from their head office and into the cloud. This has the following benefits:
- Ensures that the server is secure, free from tampering and always available;
- Infrastructure is not required for the server, it does not take up any floor space and the server hardware does not need to be managed; and
- Is easily scalable;

## Google Cloud Platform
The [Google Cloud Platform][GCP] has been decided upon as the best solution for the client. 

From the suite of products, the following items will be deployed:
- [**Google Cloud Storage**][constorage]: Similar to AWS S3 and Azure Blob Storage, a bucket will be created to hold any output analytics and provide a staging space for any compute processes.
- [**Google Cloud Dataproc**][condataproc]: Managed Hadoop and Spark service that will be run periodicly for small periods of time when analytics are required on the database. It will use storage buckets for staging and an initilization script to also install Jupyter, miniconda and associated analytical python packages and configure Jupyter to make use of pySpark.
- [**Google SQL**][consql]: MySQL database instance that simulates the retail database

These instances will be primarily driven by use of the command line commands `gcloud` and `gsutil`. The initiation of this project has been based off but heavily expanded from the tutorial [How to install and run a Jupyter notebook in a Cloud Dataproc cluster][tutorial]

### Getting Started
The following steps need to be taken in setting up the project:
- Register a trial account at https://cloud.google.com. This gives $300 US of free credit to use over 60 days. A credit card is required but is not automatically charged once the trial period is finished or if you exceed the limit.
- Take note of the `project_id` that is automatically created when you create an account
- Install the [Cloud SDK][cloudsdk] which provisions the use of `gcloud` and `gsutil` on the command line locally. Once installed, run `gcloud init` to setup the account priviledges on your computer. 

### Creating a Google Cloud Storage Bucket
- A bucket can be created by following the prompts at https://console.cloud.google.com/storage
- Although not critical, it is suggested that the bucket should be created with the following options specified while using the trial period:
    - **Bucket Name**: `<bucket-name>`
    - [**Storage Class**][stclass]: Multi-Regional
    - [**Multi-Regional Location**][mrlocation]: Asia
- The bucket can be created by issuing the following [`gsutil mb`][gsutilmb] command:
```
gsutil mb -c multi_regional -l asia -p <project_id> gs://<bucket-name>
```

### Creating a Google Cloud Dataproc instance
- The Cloud DataProc dashboard can be accessed from https://console.cloud.google.com/dataproc/
- However, rather than than using the browser we will instead use the `gcloud dataproc clusters` command to manage the Dataproc instances
- An initialization script will also be used based on the [`jupyter.sh`][jupyter] script from the [`GoogleCloudPlatform/dataproc-initialization-actions`][dpinit] GitHub repository. Basic features of this script are as follows:
    - Clone the [`GoogleCloudPlatform/dataproc-initialization-actions`][dpinit] repository on to each master and worker cluster node.
    - Download miniconda using the [`bootstrap-conda.sh`][scrconda] script
    - Install additional pip packages: (plotly, cufflinks, numpy) and conda packages: (pandas, scikit-learn) using the [`install-conda-env.sh`][scrcondaenv] script. This is an additional feature added to the original script. Adding these additional packages allows use of these additional packages in Spark RDD transformations. Had we not installed these packages at this step, we would have had to SSH into both the master and worker VM instances and manually install these packages.
    - Install jupyter notebook on the master node (only) and provision it for use with the pySpark kernel
- In order to use the initialization script stored in this repository [`configuration1.sh`][scrconfig] when we initiate the Dataproc cluster, we need to upload it into the Google Storage bucket we created in the previous section. We should then be able to access it from the link `gs://<some-unique-name>/configuration1.sh`
- Once this script has been loaded into our Google Storage Bucket, we can provision the dataproc cluster by running the following command:
```
gcloud dataproc clusters create <cluster-name> \ 
--project <project_id> \
--bucket <bucket-name> \
--initialization-actions gs://<some-unique-name>/configuration1.sh \
--master-machine-type n1-standard-2 \
--worker-machine-type n1-standard-2
```
- In this command: 
    - `n1-standard-2` is a [`machine-type`][mtypes] that consists of a standard 2 CPU machine type with 2 virtual CPU's and 7.5 GB of memory
    - One master machine will be created with the name `<cluster-name>-m`
    - Two worker machines will be created with the names `<cluster-name>-w-0` and `<cluster-name>-w-1`
- Also note that we did not specify which Zone we wanted in this command. However it is likely that an instance will be provisioned at a location close to the Zone we specified for the staging bucket. In my case, a `n1-standard-2` machine was available at the zone `asia-east-1` and was hence created at this location.
    
 ### Loading a Jupyter Notebook
- In order to load a Jupyter notebook on your local machine that is connected to this newly created dataproc instance, we require the use of two terminal windows. These:
    - Create an SSH tunnel from the cluster's master node to your localhost machine; and
    - Load a browser that connects to the SSH tunnel using the SOCKS protocol
- The following command creates a SSH tunnel from port 10000 on your local machine. Also note, that the zone may change depending on the location in which the dataproc instance was created.
```
gcloud compute ssh --zone=asia-east1-a \
--ssh-flag="-D" --ssh-flag="10000" --ssh-flag="-N" "<cluster-name>"
```
- The following command will load a browser that connects to this SSH tunnel:
```
/usr/bin/chromium-browser "http://<cluster-name>-m:8123" \
--proxy-server="socks5://localhost:10000" \
--host-resolver-rules="MAP * 0.0.0.0 , EXCLUDE localhost" \ 
--user-data-dir=/tmp/
```
- Note that in the previous command, I used a chromium browser from my Linux machine. The following are some other commands you can use if loading Chrome on your own OS:
    - Linux: `/usr/bin/google-chrome`
    - Windows: `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`
    - Mac OS X: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome`
    
### Using the Jupyter Notebook
- A browser should have been loaded with the address `http://<cluster-name>-m:8123/tree`
- In this browser we can:
    - Upload files using the Upload button from the File tab. Files may include:
        - IPython Notebooks `.ipynb` for running Python or pySpark programs
        - Supporting files for these notebooks
    - Create new files, folders or terminal processes using the New button from the File tab
        - Try creating a terminal process and run the following lines of code to test the pySpark functionality. You can terminate this process by using the command `Ctrl+D` twice (Once to exit pySpark, once more to end the terminal process)
```
pyspark
rdd = sc.parallelize([1, 2])
sorted(rdd.cartesian(rdd).collect())
```
- Two example pySpark Notebooks are provided in this repository under [`examples`][flexamples]. Upload these notebooks to the running dataproc instance to test its functionality. These are described as follows:
    - [`PortfolioPredictor.ipynb`][ipynbPP]: Adopted from the Google Cloud example on the [`Monte Carlo Method`][exMCM]
    - [`MoviePredictor.ipynb`][exMP]: Adopted from the Machine Learning Lab from the [`spark-mooc/mooc-setup`][smms] repository. Uses the Alternating Least Squares (ALS) module from the `pyspark.mllib.recommendation` module. Note that the [`movies.dat`][mdat] and [`ratings.dat`][rdat] datasets need to be manually inserted into a local storage bucket.
    
[gcp]: https://cloud.google.com
[constorage]: https://console.cloud.google.com/storage
[condataproc]: https://console.cloud.google.com/dataproc/
[consql]: https://console.cloud.google.com/sql/
[tutorial]: https://cloud.google.com/dataproc/docs/tutorials/jupyter-notebook
[cloudsdk]: https://cloud.google.com/sdk/downloads
[stclass]: https://cloud.google.com/storage/docs/storage-classes
[mrlocation]: https://cloud.google.com/storage/docs/bucket-locations
[gsutilmb]: https://cloud.google.com/storage/docs/gsutil/commands/mb
[jupyter]: https://github.com/GoogleCloudPlatform/dataproc-initialization-actions/blob/master/jupyter/jupyter.sh
[dpinit]: https://github.com/GoogleCloudPlatform/dataproc-initialization-actions
[scrconda]: https://github.com/GoogleCloudPlatform/dataproc-initialization-actions/blob/master/conda/bootstrap-conda.sh
[scrcondaenv]: https://github.com/GoogleCloudPlatform/dataproc-initialization-actions/blob/master/conda/install-conda-env.sh
[scrconfig]: https://github.com/ScottMcCormack/CITS5503/blob/master/configuration1.sh
[mtypes]: https://cloud.google.com/compute/docs/machine-types
[flexamples]: https://github.com/ScottMcCormack/CITS5503/tree/master/examples
[ipynbPP]: https://github.com/ScottMcCormack/CITS5503/blob/master/examples/PortfolioPredictor.ipynb
[exMCM]: https://cloud.google.com/solutions/monte-carlo-methods-with-hadoop-spark
[exMP]: https://github.com/ScottMcCormack/CITS5503/blob/master/examples/MoviePredictor.ipynb
[smms]: https://github.com/spark-mooc/mooc-setup
[mdat]: https://storage.googleapis.com/st-21875529/movies.dat
[rdat]: https://storage.googleapis.com/st-21875529/ratings.dat