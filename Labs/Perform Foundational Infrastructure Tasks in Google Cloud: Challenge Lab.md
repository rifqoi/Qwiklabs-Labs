# Create Bucket
- Create all resources in the **us-east1** region or **us-east1-b** zone, unless otherwise directed.
- **BUCKET_NAME** provided by Qwiklabs

```bash 
gsutil mb -p $GOOGLE_CLOUD_PROJECT 
-c standard 
-l us-east1 
gs://[BUCKET_NAME]
```

# Pubsub
- Create Pubsub
- **TOPIC_NAME** provided by Qwiklabs\
`gcloud pubsub topics create [TOPIC_NAME]`
- See if pubsub topic already created (optional)\
`gcloud pubsub topics list`

# Cloud Functions
- Deploy Function
```
mkdir gcf_thumbnail
cd gcf_thumbnail
nano index.js
```
- Copy code below to index.js
- **!! Replace line 15 "REPLACE_WITH_YOUR_TOPIC ID" with "TOPIC_NAME"**

```js
/* globals exports, require */
//jshint strict: false
//jshint esversion: 6
"use strict";
const crc32 = require("fast-crc32c");
const { Storage } = require('@google-cloud/storage');
const gcs = new Storage();
const { PubSub } = require('@google-cloud/pubsub');
const imagemagick = require("imagemagick-stream");
exports.thumbnail = (event, context) => {
  const fileName = event.name;
  const bucketName = event.bucket;
  const size = "64x64"
  const bucket = gcs.bucket(bucketName);
  const topicName = "REPLACE_WITH_YOUR_TOPIC ID";
  const pubsub = new PubSub();
  if ( fileName.search("64x64_thumbnail") == -1 ){
    // doesn't have a thumbnail, get the filename extension
    var filename_split = fileName.split('.');
    var filename_ext = filename_split[filename_split.length - 1];
    var filename_without_ext = fileName.substring(0, fileName.length - filename_ext.length );
    if (filename_ext.toLowerCase() == 'png' || filename_ext.toLowerCase() == 'jpg'){
      // only support png and jpg at this point
      console.log(`Processing Original: gs://${bucketName}/${fileName}`);
      const gcsObject = bucket.file(fileName);
      let newFilename = filename_without_ext + size + '_thumbnail.' + filename_ext;
      let gcsNewObject = bucket.file(newFilename);
      let srcStream = gcsObject.createReadStream();
      let dstStream = gcsNewObject.createWriteStream();
      let resize = imagemagick().resize(size).quality(90);
      srcStream.pipe(resize).pipe(dstStream);
      return new Promise((resolve, reject) => {
        dstStream
          .on("error", (err) => {
            console.log(`Error: ${err}`);
            reject(err);
          })
          .on("finish", () => {
            console.log(`Success: ${fileName} → ${newFilename}`);
              // set the content-type
              gcsNewObject.setMetadata(
              {
                contentType: 'image/'+ filename_ext.toLowerCase()
              }, function(err, apiResponse) {});
              pubsub
                .topic(topicName)
                .publisher()
                .publish(Buffer.from(newFilename))
                .then(messageId => {
                  console.log(`Message ${messageId} published.`);
                })
                .catch(err => {
                  console.error('ERROR:', err);
                });
          });
      });
    }
    else {
      console.log(`gs://${bucketName}/${fileName} is not an image I can handle`);
    }
  }
  else {
    console.log(`gs://${bucketName}/${fileName} already has a thumbnail`);
  }
};
```
- Create package.json for index.js\
`nano package.json`

```json
{
  "name": "thumbnails",
  "version": "1.0.0",
  "description": "Create Thumbnail of uploaded image",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "@google-cloud/pubsub": "^2.0.0",
    "@google-cloud/storage": "^5.0.0",
    "fast-crc32c": "1.0.4",
    "imagemagick-stream": "4.1.1"
  },
  "devDependencies": {},
  "engines": {
    "node": ">=4.3.2"
  }
}
```
```bash
gcloud functions deploy [FUNCTIONS_NAME] \
  --entry-point thumbnail \
  --region us-east1 \
  --trigger-resource [BUCKET_NAME] \
  --trigger-event google.storage.object.finalize \
  --runtime nodejs14 
```
  

- Upload image to the bucket previously created 

```
curl https://storage.googleapis.com/cloud-training/gsp315/map.jpg --output map.jpg
gsutil cp map.jpg gs://[BUCKET_NAME]
```

# Remove access of Username 2 from IAM
### Using Google Shell
```
gcloud projects remove-iam-policy-binding  \
$GOOGLE_CLOUD_PROJECT \
--member='user:[USERNAME 2 EMAIL]' \
--role='roles/viewer'
```
### Using Google Console
1. Navigation menu > IAM & Admin > IAM
2. Search for the "Username 2" > Edit > Delete Role

