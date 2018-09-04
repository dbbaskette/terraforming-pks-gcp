# Terraforming PKS GCP

This Repo is based on  [https://github.com/pivotal-cf/terraforming-gcp](https://github.com/pivotal-cf/terraforming-gcp). and the work at (https://github.com/making/terraforming-pks-gcp).  The Concourse Pipline work has been completed and modified to keep from having to download a 3GB file locally to then just send it back up.

### Prerequisites

The prerequisites can all installed on your local system via brew, so if you don't have brew....go get it.   Your system needs the `gcloud` cli, fly-cli (concourse) as well as `gettext`, and of course, `Docker.  Here are the OS-X installation commands:

```bash
brew update
brew install Caskroom/cask/google-cloud-sdk
brew install gettext
wget https://github.com/concourse/concourse/releases/download/v4.1.0/fly_darwin_amd64
mv fly_darwin_amd64 /usr/local/bin/fly
chmod +x fly
```

### GCP Account

##### MAKE SURE YOU HAVE ENOUGH __RESOURCE QUOTA IN THE REGION OF DEPLOYMENT__ TO CREATE THE INFRASTRUCTURE AND A PKS CLUSTER.   (CPU/RAM/STORAGE,etc)
1) If this is the first time you have used the account on your system, you will have to run these commands:
```
gcloud auth login
gcloud config set project ${PROJECT_ID}
```

You will need to enable the following Google Cloud APIs:
* Compute Engine API
* Identity and Access Management
* Cloud Resource Manager

### Service Account

You will need a key file for your service account to allow terraform to deploy resources. If you don't have one, you can create a service account and a key for it:

```
export PROJECT_ID=XXXXXXXX
export ACCOUNT_NAME=YYYYYYYY
gcloud iam service-accounts create ${ACCOUNT_NAME} --display-name "PKS Account"
gcloud iam service-accounts keys create "terraform.key.json" --iam-account "${ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member "serviceAccount:${ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" --role 'roles/owner'
```


### Var File

I have tried to limit the number of variables you are required to provide to get going.  Just fill in the variables in the text file called  `variables.txt` that's located in the `ci` subdirectory of the project.   

```
### Var Details
PIVNET_API_TOKEN: This is the new style UAA API Token (Refresh token) from your Pivnet Profile
TERRAFORM_BUCKET: This is the name of a versioned bucket.  You can pre-create it, or the system will create it for you.
GCP_PROJECT_ID: Your GCP Project name
GCP_SERVICE_ACCOUNT_KEY_PATH: The fully qualified path to a JSON Key with Owner privileges for your GCP account.  
PKS_CLI_USERNAME: A username for use in PKS CLI. (No underscores)
PKS_CLI_PASSWORD: Password for the CLI User
PKS_INITIAL_CLUSTER_NAME=""
PKS_INITIAL_CLUSTER_SIZE="small / medium / large"
GCP_REGION="Can be changed to local region"
GCP_ZONE_1="Can be changed to zones within local region"
GCP_ZONE_2="Can be changed to zones within local region"
GCP_ZONE_3="Can be changed to zones within local region"
```
### Running

Note: please make sure you have populated the `variables.txt` file above as mentioned.

### Standing up environment

From the `./ci` subdirectory:
```
./run-pipeline.sh
```

### Monitoring Progress

The script will launch multiple containers and bring up a Concourse instance and the load and start the pipeline.   After a few seconds, the script will complete and show a URL to access the pipeline within Concourse.

[http://127.0.0.1:8080/teams/main/pipelines/deploy-pks](http://127.0.0.1:8080/teams/main/pipelines/deploy-pks)

Username:  concourse
Password:  password

The Last Step of the Pipeline, SHOW-NEXT-INSTRUCTIONS, has a task that displays the procedures for creating a K8S cluster.   You should be able to cut and past from that screen.

These is also an Optional Task available that will create a cluster for you.  

### Tearing down environment

To tear down the installation, simply trigger the wipe-env task.   There might be a few cluster specific things to clean up.  If the task fails, you can clean those things up manually and then trigger it again. If ops-man was not created poroperly, wipe-env will not work in it's current state.   You will have to delete ALL object manually.  Sometimes objects are easy to miss, so it's also recommended to change the prefix of the next deployment and the bucket name.    This will ensure you don't have any conflicts.