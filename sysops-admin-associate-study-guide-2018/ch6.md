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

### Down

## Exercise 6.4 Create and use S3 bucket
### Up

### Down

## Exercise 6.5 Enable S3 versioning
### Up

### Down

## Exercise 6.6 Enable cross-region replication
### Up

### Down

## Exercise 6.7 Create a Glacier Vault
### Up

### Down

## Exercise 6.8 Enable Lifecycle Rules
### Up

### Down

