---
title: Object lifecycles
pcx_content_type: how-to
---

# Object lifecycles

Object lifecycles determine the retention period of objects uploaded to your bucket and allow you to specify when objects should transition from Standard storage to Infrequent Access storage.

For example, you can create an object lifecycle rule to delete objects after 90 days, or you can set a rule to transition objects to Infrequent Access storage after 30 days.

## Behavior

- Objects will typically be removed from a bucket within 24 hours of the `x-amz-expiration` value.
- When a lifecycle policy is applied that deletes objects, newly uploaded objects' `x-amz-expiration` value immediately reflects the expiration based on the new rules, but existing objects may experience a delay. Most objects will be transitioned within 24 hours but may take longer depending on the number of objects in the bucket. While objects are being migrated, you may see old applied rules from the previous policy.
- An object is no longer billable once it has been deleted.
- Buckets have a default lifecycle policy to expire multipart uploads seven days after initiation.
- When an object is transitioned from Standard storage to Infrequent Access storage, a [Class A operation](/r2/pricing/#class-a-operations) is incurred.

## Configure your bucket's object lifecycle policy

When you create an object lifecycle rule, you can specify which prefix you would like it to apply to.

* Note that object lifecycles currently has a 1000 rule maximum.
* Managing object lifecycles is a bucket-level action, and requires an API token with the [`Workers R2 Storage Write`](/r2/api/s3/tokens/#permission-groups) permission group.

### Dashboard

1. From the Cloudflare dashboard, select **R2**.
2. Locate and select your bucket from the list.
3. From the bucket page, select **Settings**.
4. Under **Object lifecycle rules**, select **Add rule**.
5. Fill out the fields for the new rule.
6. When you are done, select **Add rule**.

### S3 API

Below is an example of configuring a lifecycle policy with different sets of rules for different potential use cases. 

```js
---
header: Configure the S3 client to interact with R2
---
const client = new S3({
  endpoint: "https://4893d737c0b9e484dfc37ec392b5fa8a.r2.cloudflarestorage.com",
  credentials: {
    accessKeyId: "7dc27c125a22ad808cd01df8ec309d41",
    secretAccessKey:
      "1aa5c5b0c43defdb88f567487c071d17e234126133444770a706ae09336c57a4",
  },
  region: "auto",
});
```

```javascript
---
header: Configure the lifecycle policy for a bucket
---
await client
  .putBucketLifecycleConfiguration({
    LifecycleConfiguration: {
      Bucket: "testBucket",
      Rules: [
        // Example: deleting objects on a specific date
        // Delete 2019 documents in 2024
        {
          ID: "Delete 2019 Documents",
          Filter: {
            Prefix: "2019/",
          },
          Expiration: {
            Date: new Date("2024-01-01"),
          },
        },
        // Example: transitioning objects to Infrequent Access storage by age
        // Transition objects older than 30 days to Infrequent Access storage
        {
          ID: 'Transition Objects To Infrequent Access',
          Transitions: [
            {
              Days: 30,
              StorageClass: 'STANDARD_IA',
            },
          ],
        },
        // Example: deleting objects by age
        // Delete logs older than 90 days
        {
          ID: "Delete Old Logs",
          Filter: {
            Prefix: "logs/",
          },
          Expiration: {
            Days: 90,
          },
        },
        // Example: abort all incomplete multipart uploads after a week
        {
          ID: "Abort Incomplete Multipart Uploads",
          AbortIncompleteMultipartUpload: {
            DaysAfterInitiation: 7,
          },
        },
        // Example: abort user multipart uploads after a day
        {
          ID: "Abort User Incomplete Multipart Uploads",
          Filter: {
            Prefix: "useruploads/",
          },
          AbortIncompleteMultipartUpload: {
            // For uploads matching the prefix, this rule will take precedence
            // over the one above due to its earlier expiration.
            Days: 1,
          },
        },
      ],
    },
  })
  .promise();
```

## Get a bucket's lifecycle policy

```js
import S3 from "aws-sdk/clients/s3.js";

// Configure the S3 client to talk to R2.
const client = new S3({
 endpoint: "https://4893d737c0b9e484dfc37ec392b5fa8a.r2.cloudflarestorage.com",
 credentials: {
   accessKeyId: "7dc27c125a22ad808cd01df8ec309d41",
   secretAccessKey:
     "1aa5c5b0c43defdb88f567487c071d17e234126133444770a706ae09336c57a4",
 },
 region: "auto",
});


// Get lifecycle policy for bucket
console.log(
 await client
   .getBucketLifecycleConfiguration({
     Bucket: "bucketName",
   })
   .promise()
);
```

## Delete a bucket's lifecycle policy

### Dashboard

1. From the Cloudflare dashboard, select **R2**.
2. Locate and select your bucket from the list.
3. From the bucket page, select **Settings**.
4. Under **Object lifecycle rules**, select the rules you would like to delete.
5. When you are done, select **Delete rule(s)**.

### S3 API

```js
import S3 from "aws-sdk/clients/s3.js";


// Configure the S3 client to talk to R2.
const client = new S3({
 endpoint: "https://4893d737c0b9e484dfc37ec392b5fa8a.r2.cloudflarestorage.com",
 credentials: {
   accessKeyId: "7dc27c125a22ad808cd01df8ec309d41",
   secretAccessKey:
     "1aa5c5b0c43defdb88f567487c071d17e234126133444770a706ae09336c57a4",
 },
 region: "auto",
});


// Delete lifecycle policy for bucket
await client.deleteBucketLifecycle({
 Bucket: "bucketName"
}).promise()
```
