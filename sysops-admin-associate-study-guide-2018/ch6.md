# Chapter 6 Storage Systems

## Exercise 6.1a Create an Encrypted EBS Volume (using management console)
### Up
1. Navigate to EC2 > Elastic Block Store > Volumes
1. Create volume
1. Select default KMS master key. Create one if none exist

### Down
1. Delete volume
1. Delete KMS key, if one was created for exercise

## Exercise 6.1b Create an Encrypted EBS Volume (using AWS CLI)
### Up
1. Create new KMS key
    ```
    KEY_ID=$(aws kms create-key --query 'KeyMetadata.KeyId' --output text)
    ```
1. Create new KMS alias, associate with newly created KMS key
    ```
    ALIAS_NAME="alias/aallbrig/chapter-exercises-ebs"

    aws kms create-alias --alias-name $ALIAS_NAME --target-key-id $KEY_ID
    ```
1. Get a random availability zone (assuming you're on OSX and have installed coreutils, and have `shuf` under `gshuf`)
    ```
    AVAILABILITY_ZONE=$(gshuf -n 1 -e $(aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text))
    ```
1. Create encrypted EBS volume
    ```
    VOLUME_ID=$(aws ec2 create-volume --availability-zone $AVAILABILITY_ZONE --encrypted --kms-key-id $KEY_ID --size 1 --volume-type standard --query 'VolumeId' --output text)
    ```
1. Celebrate. No really, the book's exercise doesn't ask you to do more with the encrypted volume.

### Down
1. Delete volume
    ```
    aws ec2 delete-volume --volume-id $VOLUME_ID
    ```
1. Delete alias
    ```
    aws kms delete-alias --alias-name $ALIAS_NAME
    ```
1. Schedule KMS key to be deleted
    ```
    aws kms schedule-key-deletion --key-id $KEY_ID --pending-window-in-days 7
    ```

## Exercise 6.2 Monitor EBS using CloudWatch
### Up
1. Open CloudWatch service dashboard in management console and navigate to EBS per-volume metrics
    ```
    aws cloudwatch list-metrics --namespace "AWS/EBS" --max-items 10
    ```
1. Poke around, explore
    ```
    aws cloudwatch help
    ```

### Down
N/A because no resources were created, thus no resources need to be destroyed

## Exercise 6.3 Create and attach an EFS volume
### Up
1. Launch EC2 instance (preferably linux ami)
1. Open EFS service dashboard and create file system
1. Log in to console of linux instance, install NFS client on EC2 instance
    ```
    sudo yum install -y nfs-utils
    ```
1. Make directory for newly created EFS
    ```
    mkdir efs
    ```
1. Mount the newly created file system using DNS name
    ```
    sudo mount -t nfs4 -o nfscers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 $EFS_DNS / efs
    ```

### Down
1. Terminate EC2 instance
1. Delete EFS file system

## Exercise 6.4 Create and use S3 bucket
### Up
1. Create globally unique name for S3 bucket
    ```
    RANDOM_BUCKET_NAME="test-bucket-$(openssl rand -hex 18)"
    ```
1. Create S3 bucket
    ```
    aws s3 mb "s3://${RANDOM_BUCKET_NAME}"
    ```
1. Test S3 bucket
    1. Create random file to upload
        ```
        echo "Test S3 bucket upload\!" > test.txt
        ```
    1. Upload file to newly created S3 bucket
        ```
        aws s3 cp test.txt "s3://${RANDOM_BUCKET_NAME}/test.txt" --acl public-read
        ```
    1. Verify upload by listing out contents of bucket
        ```
        aws s3 ls "s3://${RANDOM_BUCKET_NAME}"
        ```
    1. Check public read permission via cURL
        ```
        curl "https://s3.amazonaws.com/$RANDOM_BUCKET_NAME/test.txt"
        ```
        - If you want to verify public access by restricting access, follow these next commands
            ```
            aws s3 cp test.txt "s3://${RANDOM_BUCKET_NAME}/test.txt" --acl private
            curl "https://s3.amazonaws.com/$RANDOM_BUCKET_NAME/test.txt"
            ```

            You will see a generic "access denied" HTML page
    1. Delete local test.txt file
        ```
        rm test.txt
        ```
    1. Download text.txt file from S3 bucket
        ```
        aws s3 cp "s3://${RANDOM_BUCKET_NAME}/test.txt" test.txt
        ```

### Down
1. Remove test file (if created for testing purposes)
    ```
    rm test.txt
    ```
1. Recursively remove all content in bucket
    ```
    aws s3 rm "s3://${RANDOM_BUCKET_NAME}" --recursive
    ```
1. Delete s3 bucket
    ```
    aws s3api delete-bucket --bucket "${RANDOM_BUCKET_NAME}"
    ```

## Exercise 6.5 Enable S3 versioning
### Up
1. If you do not already have a bucket, follow the steps for exercise 6.4
1. Enable S3 bucket versioning
    ```
    aws s3api put-bucket-versioning --bucket $RANDOM_BUCKET_NAME --versioning-configuration Status=Enabled
    ```
    1. Get a baseline on versions
        ```
        aws s3api list-object-versions --bucket $RANDOM_BUCKET_NAME
        ```
        Notice that there is only one version
    1. Write a new version of `test.txt` to test versioning
        ```
        echo "Test S3 bucket upload\! Version 2" > test.txt
        ```
    1. Upload new version of `test.txt` to S3 bucket
        ```
        aws s3 cp test.txt "s3://${RANDOM_BUCKET_NAME}/test.txt"
        ```
    1. Check versions-- you should see a new listing after uploading your new version of `test.txt`
        ```
        aws s3api list-object-versions --bucket $RANDOM_BUCKET_NAME
        ```

### Down
1. Follow steps for 6.4 to spin down bucket

## Exercise 6.6 Enable cross-region replication
### Up

### Down

## Exercise 6.7 Create a Glacier Vault
### Up

### Down

## Exercise 6.8 Enable Lifecycle Rules
### Up

### Down

