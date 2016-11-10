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
- [**Google SQL**][consql]: MySQL database instance that similates the retail database

These instances will be primarily driven by use of the command line commands `gcloud` and `gsutil`. The initiation of this project has been based off but heavily expanded from the tutorial [How to install and run a Jupyter notebook in a Cloud Dataproc cluster][tutorial]

### Getting Started
The following steps need to be taken in setting up the project:
- Register a trial account at https://cloud.google.com. This gives $300 US of free credit to use over 60 days. A credit card is required but is not automatically charged once the trial period is finished or if you exceed the limit.
- Take note of the `project_id` that is automatically created when you create an account
- Install the [Cloud SDK][cloudsdk] which provisions the use of `gcloud` and `gsutil` on the command line locally. Once installed, run `gcloud init` to setup the account priviledges on your computer. 

### Google Cloud Storage
- A bucket can be created by following the prompts at https://console.cloud.google.com/storage
- Although not critical, it is suggested that the bucket should be created with the following options specified while using the trial period:
    - **Bucket Name**: *unique-name*
    - [**Storage Class**][stclass]: Multi-Regional
    - [**Multi-Regional Location**][mrlocation]: Asia
- The bucket can be created by issuing the following [`gsutil mb`][gsutilmb] command:
```
gsutil mb -c multi_regional -l asia -p <project_id> gs://<some-unique-name>
```

### Google Cloud Dataproc
- The Cloud DataProc dashboard can be accessed from https://console.cloud.google.com/dataproc/
- However, rather than than using the browser we will instead use the `gcloud dataproc clusters` command to manage the Dataproc instances
- An initialization script will also be used based on the [`jupyter.sh`][jupyter] script from the [`GoogleCloudPlatform/dataproc-initialization-actions`][dpinit] GitHub repository. Basic features of this script are as follows:
    - Clone the [`GoogleCloudPlatform/dataproc-initialization-actions`][dpinit] repository on to each master and worker cluster node.
    - Download miniconda using the [`bootstrap-conda.sh`][scrconda] script
    - Install additional pip packages: (plotly, cufflinks, numpy) and conda packages: (pandas, scikit-learn) using the [`install-conda-env.sh`][scrcondaenv] script. This is an additional feature added to the original script
    - Install jupyter notebook on the master node (only) and provision it for use with the pySpark kernel

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