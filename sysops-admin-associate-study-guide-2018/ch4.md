# Chapter 4 Compute
## Exercise 4.1 Create a Linux Instance via AWS Management Console
### Up
1. Launch EC2
1. Choose Linux AMI
1. T.2 micro instance type (free tier eligible)
1. Launch into default VPC
1. Assign a public IP address
1. Add tag to instance of `key` = `Yankee`, `value` = `Zulu`
1. Create new security group called `YankeeZuluSG`
1. Add rule to `YankeeZulu` security group that allows SSH access from anywhere (0.0.0.0/0)
1. Launch instance
1. Create new key pair, name it `YankeeZuluKeyPair`, save private portion
1. SSH into instance
  1. You may need to `chmod 400` the newly downloaded keypair
1. Install software updates
  1. Maybe you want to start a simple web server; assuming Ubuntu OS...
  1. Add HTTP inbound rule to `YankeeZuluSG` from anywhere
  1. `mkdir www && cd www`
  1. `touch index.html`
  1. 
```
cat <<HEREDOC > index.html
<html>
  <head><title>Test HTML page</title></head>
  <body><h3>Test HTML page</h3></body>
</html>
HEREDOC
``` 
  1. `sudo python3 -m http.server 80`  
1. Run `ifconfig -a` then make note of IP address
1. Access instance metadata by running `curl http://169.254.169.254/latest/meta-data/public-ipv4`
1. Exist SSH session

### Down
If not using for Exercise 4.6
1. Terminate instance
1. Delete security group called `YankeeZuluSG`
1. Delete keypair `YankeeZuluKeyPair`

## Exercise 4.2 Create a Windows Instance via AWS Management Console
### Up
1. Launch EC2
1. Choose Windows 2016 Base AMI
1. The book wants T2.medium, but that isn't free tier so... t2.micro it is! Let's see how well the t2.micro handles RDP
1. Launch into default VPC
1. Assign public IP address
1. Add tag to instance of `key` = `alpha`, `value` = `bravo`
1. Create new security group called `AlphaBravoSG`
1. Add rule to security group that allows RDP from anywhere (0.0.0.0/0)
1. Launch instance
1. Create new key pair, name it `AlphaBravoKeyPair`
  1. You may need to `chmod 400` the newly downloaded keypair
1. On connect instance UI, decrypt administrator password, then download the RDP file
1. Connect to instance and poke around for a bit

### Down
1. Terminate instance
1. Delete security group called `AlphaBravoSG`
1. Delete key pair called `AlphaBravoKeyPair`

## Exercise 4.3 Create Linux Instance via AWS CLI
### Up
Before starting, take note of AMI ID for the target region.

1. Set variables
```
IMAGE_AMI= \
KEY_NAME= \
SECURITY_GROUP_ID= \
SUBNET_ID= 
```
1. Run the command...
```
aws ec2 run-instances \
--image-id $IMAGE_AMI \
--count 1 \
--instance-type t2.micro \
--key-name $KEY_NAME \
--security-group-ids $SECURITY_GROUP_ID \
--subnet-id $SUBNET_ID
```
1. SSH into instance, poke around a bit

### Down
1. You may need to run `aws ec2 describe-instances` to get instance IDs

`aws ec2 describe-instances | jq ".Reservations[].Instances[].InstanceId" -r` can be used if `jq` is installed
1. Once instance ID has been identified, run `aws ec2 terminate-instances --instance-ids ` and fill in instance ID value
1. Handy one liner, to terminate all currently running EC2 instances
```
aws ec2 describe-instances \
| jq ".Reservations[].Instances[].InstanceId" -r \
| xargs -n1 -I instance_id aws ec2 terminate-instances --instance-ids instance_id
```
1. If needed, delete security group and key pair if one was dedicated to this exercise

## Exercise 4.4 Create a Windows Instance via AWS CLI
### Up
Identify default VPC, assuming jq is installed
```
aws ec2 describe-vpcs \
| jq -c ".Vpcs \
| map(select(.IsDefault \
| contains(true)))[].VpcId" -r
```

1. Create security group if needed
  1. Set variables
  ```
  VPC_ID=$( \
    aws ec2 describe-vpcs | jq -c ".Vpcs | map(select(.IsDefault | contains(true)))[].VpcId" -r \
  ) \
  SG_NAME=AlphaBravoSG \
  SG_DESCRIPTION="Security Group for Alpha Bravo"
  ```
  1. Identify if you already have relevant SG
  ```
  aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[]"
  ```
===== Below to be deleted, up to "### Down" (exclusive)
Before starting, take note of AMI ID for the target region.
1. Set variables
```
IMAGE_AMI= \
KEY_NAME= \
SECURITY_GROUP_ID= \
SUBNET_ID= 
```
1. Run the command...
```
aws ec2 run-instances \
--image-id $IMAGE_AMI \
--count 1 \
--instance-type t2.micro \
--key-name $KEY_NAME \
--security-group-ids $SECURITY_GROUP_ID \
--subnet-id $SUBNET_ID
```
1. Make note of the instance ID (or run `aws ec2 describe-instances`) returned from previous command
1. Use RDP to access this windows EC2 instance

### Down
1. You may need to run `aws ec2 describe-instances` to get instance IDs

`aws ec2 describe-instances | jq ".Reservations[].Instances[].InstanceId" -r` can be used if `jq` is installed
1. Once instance ID has been identified, run `aws ec2 terminate-instances --instance-ids ` and fill in instance ID value
1. Handy one liner, to terminate all currently running EC2 instances
```
aws ec2 describe-instances \
| jq ".Reservations[].Instances[].InstanceId" -r \
| xargs -n1 -I instance_id aws ec2 terminate-instances --instance-ids instance_id
```
1. If needed, delete security group and key pair if one was dedicated to this exercise

