# LAB: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives:

In this lab, you learn how to perform the following tasks:

  - Create and use buckets

  - Set access control lists to restrict access

  - Use your own encryption keys

  - Implement version controls

  - Use directory synchronization

  - Share a bucket across projects using IAM

  ## Steps:

1. Preparation:

   1. Create a Cloud Storage bucket:

             gsutil mb gs://$DEVSHELL_PROJECT_ID

   2. Download a sample file using CURL and make two copies:

      - Store [BUCKET_NAME_1] in an environment variable:

             export BUCKET_NAME_1=$DEVSHELL_PROJECT_ID

      - Verify it with echo:

             echo $BUCKET_NAME_1

      - Download a sample file (this sample file is a publicly available Hadoop documentation HTML file):

             curl http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html > setup.html

      - Make copies of the file, run the following commands:

             cp setup.html setup2.html

             cp setup.html setup3.html

2. Set access control lists to restrict access:

   1. Copy the file to the bucket and configure the access control list:

      - Copy the first file to the bucket:

             gsutil cp setup.html gs://$BUCKET_NAME_1/

      - Get the default access list that's been assigned to setup.html:

             gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl.txt

             cat acl.txt

      - Set the access list to private and verify the results:

             gsutil acl set private gs://$BUCKET_NAME_1/setup.
             
             gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl2.txt

             cat acl2.txt

      - Update the access list to make the file publicly readable:

             gsutil acl ch -u AllUsers:R gs://$BUCKET_NAME_1/setup.html

             gsutil acl get gs://$BUCKET_NAME_1/setup.html  > acl3.txt

             cat acl3.txt

   2. Examine the file in the Cloud Console:

             gsutil ls -r gs://$DEVSHELL_PROJECT_ID

   3. Delete the local file and copy back from Cloud Storage:

      - Delete the setup file:

             rm setup.html

      - Verify that the file has been deleted:

             ls

      - Copy the file from the bucket again:

             gsutil cp gs://$BUCKET_NAME_1/setup.html setup.html

3. Customer-supplied encryption keys (CSEK):

   1. Generate a CSEK key:

             python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'

   2. Modify the boto file:

             ls -al

             nano .boto

   3. Upload the remaining setup files (encrypted) and verify in the Cloud Console:

             gsutil cp setup2.html gs://$BUCKET_NAME_1/

             gsutil cp setup3.html gs://$BUCKET_NAME_1/

   4. Delete local files, copy new files, and verify encryption:

      - Delete your local files:

             rm setup*

      - Copy the files from the bucket again:

             gsutil cp gs://$BUCKET_NAME_1/setup* ./

      - Cat the encrypted files to see whether they made it back:

             cat setup.html

             cat setup2.html

             cat setup3.html

4. Rotate CSEK keys:

   1. Move the current CSEK encrypt key to decrypt key:

             nano .boto

   2. Generate another CSEK key and add to the boto file:

             python3 -c 'import base64; import os; print(base64.encodebytes(os.urandom(32)))'

             nano .boto

   3. Rewrite the key for file 1 and comment out the old decrypt key:

             gsutil rewrite -k gs://$BUCKET_NAME_1/setup2.html

             nano .boto

   4. Download setup 2 and setup3:

             gsutil cp  gs://$BUCKET_NAME_1/setup2.html recover2.html

             gsutil cp  gs://$BUCKET_NAME_1/setup3.html recover3.html

5.Enable lifecycle management

   1. View the current lifecycle policy for the bucket:

             gsutil lifecycle get gs://$BUCKET_NAME_1

   2. Create a JSON lifecycle policy file:

             nano life.json

   3. Paste the code into the created json file: 

             {
                "rule":
                [
                  {
                    "action": {"type": "Delete"},
                    "condition": {"age": 31}
                  }
                ]
             }

   4. Set the policy and verify:

             gsutil lifecycle set life.json gs://$BUCKET_NAME_1

             gsutil lifecycle get gs://$BUCKET_NAME_1

6. Enable versioning:

   1. View the versioning status for the bucket and enable versioning:

      - View the current versioning status for the bucket:

             gsutil versioning get gs://$BUCKET_NAME_1

      - Enable versioning:

             gsutil versioning set on gs://$BUCKET_NAME_1

      - Verify that versioning was enabled:

             gsutil versioning get gs://$BUCKET_NAME_1

   2. Create several versions of the sample file in the bucket:

      - Check the size of the sample file:

             ls -al setup.html

      - Open the setup.html file:

             nano setup.html

      - Delete any 5 lines from setup.html to change the size of the file.

      - Press Ctrl+O, ENTER to save the file, and then press Ctrl+X to exit nano.

      - Copy the file to the bucket with the -v versioning option:

             gsutil cp -v setup.html gs://$BUCKET_NAME_1

      - Open the setup.html file:

             nano setup.html

      - Delete another 5 lines from setup.html to change the size of the file.

      - Press Ctrl+O, ENTER to save the file, and then press Ctrl+X to exit nano.

      - Copy the file to the bucket with the -v versioning option:

             gsutil cp -v setup.html gs://$BUCKET_NAME_1

   3. List all versions of the file:

             gsutil ls -a gs://$BUCKET_NAME_1/setup.html

             export VERSION_NAME=gs://qwiklabs-gcp-04-23bf2354a280/setup.html#1584457872853517

             echo $VERSION_NAME

   4. Download the oldest, original version of the file and verify recovery:

             gsutil cp $VERSION_NAME recovered.txt

             ls -al setup.html

             ls -al recovered.txt

7. Synchronize a directory to a bucket

   1. Make a nested directory and sync with a bucket:

             mkdir firstlevel
             
             mkdir ./firstlevel/secondlevel
             cp setup.html firstlevel
             cp setup.html firstlevel/secondlevel

             gsutil rsync -r ./firstlevel gs://$BUCKET_NAME_1/firstlevel

   2. Examine the results:

             gsutil ls -r gs://$BUCKET_NAME_1/firstlevel

             exit

8. Cross-project sharing

   1. Switch to the second project:

   2. Prepare the bucket:

             gcloud mb gs://cain-bucket-1

   3. Upload a text file to the bucket

   4. Create an IAM Service Account

             gcloud iam service-accounts create cross-project-storage

             gcloud iam roles create role-id --permissions=storage.object.viewer --stage=launch-stage --service-account cross-project-storage

   5. Create a VM

             gcloud compute instances create crossproject --zone europe-west1-d --machine-type n1-standard-1 --boot-disk "Debian GNU/Linux 10 (buster)"

   6. SSH to the VM:

             gcloud compute ssh crossproject

             export BUCKET_NAME_2=cain-bucket-1

             echo $BUCKET_NAME_2

             export FILE_NAME=sample.txt

             echo $FILE_NAME

             gsutil ls gs://$BUCKET_NAME_2/

   7. Authorize the VM

      - Upload and verify that the json file has been uploaded:

             ls

      - Authorize the VM to use the Google Cloud API:

             gcloud auth activate-service-account --key-file credentials.json

   8. Verify access:

      - Retry this command:

             gsutil ls gs://$BUCKET_NAME_2/

      - Retry this command:

             gsutil cat gs://$BUCKET_NAME_2/$FILE_NAME

      - Try to copy the credentials file to the bucket:

             gsutil cp credentials.json gs://$BUCKET_NAME_2/

   9. Modify role:

             gcloud iam service-accounts update cross-project-storage --role cloud-storage --permissions storage.object.admin

   10. Verify changed access:

      - Return to VM

             gcloud compute ssh crossproject

      - Copy the credentials file to the bucket:

             gsutil cp credentials.json gs://$BUCKET_NAME_2/

             