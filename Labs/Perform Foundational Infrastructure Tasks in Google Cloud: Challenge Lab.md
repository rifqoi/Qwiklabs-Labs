# Create Bucket
1. Open Navigation menu > Cloud Storage > Browser
2. Click Create Bucket
Create all resources in the us-east1 region and us-east1-b zone, unless otherwise directed.
Bucket Name provided by Qwiklabs

# Pubsub
## Create Pubsub
gcloud pubsub topics create [TOPIC_NAME]
## See if pubsub topic already created (optional)
gcloud pubsub topics list 


# Cloud Functions
## Deploy Function
gcloud functions deploy [FUNCTIONS_NAME] \
--entry-point thumbnail \
--region us-east1 \
--trigger-resource [BUCKET_NAME] \
--trigger-event google.storage.object.finalize \
--runtime nodejs14

## Upload image to the bucket previously created 
curl https://storage.googleapis.com/cloud-training/gsp315/map.jpg --output map.jpg
gsutil cp map.jpg gs://[BUCKET_NAME]

# Remove access of username2 from IAM
1. jNavigation menu > IAM & Admin > IAM
2. Search for the "Username 2" > Edit > Delete Role
