# Dataflow
Load data from GCS to BigQuery using Dataflow

## Tech Stack
1. Cloud Composer
2. Google Cloud Storage
3. BigQuery
4. Dataflow

## Setup
### 1. Composer Environment
Composer is a managed Airflow environment on GCP. With Composer, you don't have to spend more time managing Airflow; You can focus on writing DAGs. Note: to convert data from GCS to BigQuery, you don't need to use Composer, but it's useful if you want to use a DAG, and I'll write down the steps to create an environment in Composer.

Steps to activate the environment:
1. Activate Composer API if you haven't. You can refer to this link on how to: https://cloud.google.com/endpoints/docs/openapi/enable-api#console
2. Open https://console.cloud.google.com/
3. On Navigation Menu, go to Big Data > Composer. Wait until Composer page shown
4. Click CREATE button to create new Composer environment and choose Composer 1.
<img width="553" alt="Screenshot 2023-03-08 at 16 29 20" src="https://user-images.githubusercontent.com/113230789/223675432-299755de-db63-4484-bf50-8b8850dc89a2.png">
5. Fill some required fields there. This projects use this machine config:

 - Name: cloud-flower
 - Location: us-central1
 - Image Version: composer-1.20.7-airflow-2.2.5
 - Node count: 3
 - Zone: us-central-a
 - Machine Type: n1-standard-1
 - Disk size (GB): 30 GB
 - Service account: choose your Compute Engine service account.
 - Cloud SQL machine type: db-n1-standard-2
 - Web server machine type: composer-n1-webserver2

6. Fill the Environment Variable with
 - ENVIRONMENT: production
7. Then, click the **CREATE** button
8. You will need to wait a few minutes (15-25 minutes) for the environment to be successfully created. After the environment has been created successfully you will see like this:

<img width="1440" alt="Screenshot 2023-03-08 at 16 09 03" src="https://user-images.githubusercontent.com/113230789/223671573-11cbdb1e-a748-4301-b54e-7a91023e019e.png">
notes: errors may occur, you can try to change the location, I suggest choosing the closest location (eg: asia-south1) so that the costs incurred are not too large.

## 2. Google Cloud Storage
When you create an environment on Composer, it will simultaneously generate a bucket so you can use the bucket directly as I did, but if not, you can create your bucket by:
1. Back to your GCP console, choose Cloud Storage. You can find it on More Products > Storage > Cloud Storage, or simply search it at the search bar.
2. Click **CREATE BUCKET** button. Then fill some fields such as:
  - Name your bucket (example: df9_bkt)
  - Choose where to store your data
  - Leave default for the rest of fields.
  - Click **CREATE**
  - Your bucket will be created and showed on GCS Browser
 <img width="1376" alt="Screenshot 2023-03-08 at 16 33 54" src="https://user-images.githubusercontent.com/113230789/223676393-3de91164-a5f9-46ad-99f1-6650a474c0b1.png">
 
 3. Upload files to bucket:
  - javascript file and json that you can find in the dataflow-function folder.
  - citizen.txt that you can find in the data folder.
 
## 3. BigQuery
1. On your GCP console, go to BigQuery. You can find it on More Products > Analytics > BigQuery, or simply search it at the search bar.
2. Create dataset
 <img width="412" alt="Screenshot 2023-03-08 at 16 37 10" src="https://user-images.githubusercontent.com/113230789/223677165-ca9007d6-bd4b-4474-b21b-ee0b87277a77.png">
3. Fill the Dataset field such as:

 - Data set ID (example: dataflow_citizen)
 - Data location. Note: make sure the dataset location is the same as the bucket location!
<img width="577" alt="Screenshot 2023-03-08 at 16 40 20" src="https://user-images.githubusercontent.com/113230789/223677866-1867427b-480a-41d7-8d22-fdf2f09877bd.png">

4. Click **CREATE DATASET**
5. Ensure that your dataset has been created.
<img width="395" alt="Screenshot 2023-03-08 at 16 41 35" src="https://user-images.githubusercontent.com/113230789/223678142-d84e2405-7d97-4c4b-ac73-50e8fb029f95.png">
6. Create table
<img width="411" alt="Screenshot 2023-03-08 at 16 42 35" src="https://user-images.githubusercontent.com/113230789/223678358-48497b17-3ffe-4870-a94a-11d2b7729150.png">

7. Fill the Table field such as:
 - table name (eg: citizen)
 - schema (can be skipped)

## 4. Dataflow
1. On your GCP console, go to Dataflow. You can find it on More Products > Analytics > Dataflow, or simply search it at the search bar.
2. Make sure that you have enabled API.
3. Click **CREATE JOB FROM TEMPLATE**
4. Fill the field such as:
  - Jobname (eg: citizen_to_bq)
  - Regional endpoint: us-central1
  - Dataflow template: Process Data in Bulk (batch) > Text Files on Cloud Storage to BigQuery
  - Javascript path: open your javascript file in the bucket, and copy the gsutil URL.
 <img width="715" alt="Screenshot 2023-03-08 at 17 16 38" src="https://user-images.githubusercontent.com/113230789/223686321-801e2561-afd8-4c27-8974-405bea6ba6e2.png">
 
  - Json path: browse your json file.
  - Javascript UDF name: the function to call from your JavaScript file (in this project, the function to call javascript is "transform")
  - Bigquery output table (browse your output table, in this project the table is "citizen")
  - Temporary Bigquery directory (create your temp folder on bucket: format gss://(your bucket)/(folder name)/ )
  - Temporary location (create your temp folder on bucket: format gss://(your bucket)/(folder name)/ )
 
   <img width="492" alt="Screenshot 2023-03-08 at 17 22 53" src="https://user-images.githubusercontent.com/113230789/223687760-62898d4c-a73a-43a9-9f0a-f4bd425c4be0.png">
   
5. Click **RUN JOB**
6. Here's what you can see when the job runs successfully:
<img width="1375" alt="Screenshot 2023-03-08 at 17 02 01" src="https://user-images.githubusercontent.com/113230789/223683419-d68dc681-4377-4ce9-92fc-4f7728e7bb54.png">
7. Check the table output in BigQuery to ensure the data has been loaded.
<img width="1107" alt="Screenshot 2023-03-08 at 17 11 22" src="https://user-images.githubusercontent.com/113230789/223685119-a23a6d56-eba4-4235-a07b-7e80b7a198fc.png">
