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
    1. Run...
        ```sh
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
    ```sh
    IMAGE_AMI= \
    KEY_NAME= \
    SECURITY_GROUP_ID= \
    SUBNET_ID=
    ```
1. Run the command...
    ```sh
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
    ```sh
    aws ec2 describe-instances \
    | jq ".Reservations[].Instances[].InstanceId" -r \
    | xargs -n1 -I instance_id aws ec2 terminate-instances --instance-ids instance_id
    ```
1. If needed, delete security group and key pair if one was dedicated to this exercise

## Exercise 4.4 Create a Windows Instance via AWS CLI
### Up

1. Set variables
    ```sh
    IMAGE_AMI="ami-0b7b74ba8473ec232" \
    VPC_ID=$( \
      aws ec2 describe-vpcs | jq -c ".Vpcs | map(select(.IsDefault | contains(true)))[].VpcId" -r \
    ) \
    SG_NAME=AlphaBravoSG \
    SG_DESCRIPTION="Security Group for Alpha Bravo"
    KEY_NAME=AlphaBravoKP \
    SUBNET_ID=$(
      aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" | jq ".Subnets[] | .SubnetId" -r | gshuf | head -n 1 \
    ) \
    SECURITY_GROUP_ID=$( \
      aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[] .GroupId" -r \
    )
    ```
1. Identify if you already have relevant security group
    ```sh
    aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[]"
    ```

    Above will return an object or nothing. If nothing, the security group does not exist.
    1. If no security group exists, run...
        ```sh
        aws ec2 create-security-group --group-name $SG_NAME \
        --description $SG_DESCRIPTION \
        --vpc-id $VPC_ID
        ```
    1. Assign security group ID to variable
        ```sh
        SECURITY_GROUP_ID=$(aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[] .GroupId" -r)
        ```
    1. Create ingress rule for RDP from anywhere (0.0.0.0/0)
        ```sh
        aws ec2 authorize-security-group-ingress \
        --group-id $SECURITY_GROUP_ID \
        --protocol tcp \
        --port 3389 \
        --cidr 0.0.0.0/0
        ```
1. Identify if you already have relevant key pair
    ```sh
    aws ec2 describe-key-pairs --key-name $KEY_NAME
    ```
    1. If you do not, run
        ```sh
        aws ec2 create-key-pair --key-name $KEY_NAME  --query 'KeyMaterial' --output text > "${KEY_NAME}.pem"
        ```
1. Run the command...
    ```sh
    aws ec2 run-instances \
    --image-id $IMAGE_AMI \
    --count 1 \
    --instance-type t2.micro \
    --key-name $KEY_NAME \
    --security-group-ids $SECURITY_GROUP_ID \
    --subnet-id $SUBNET_ID
    ```
1. RDP into instance, poke around a bit

### Down
1. You may need to run `aws ec2 describe-instances` to get instance IDs

    `aws ec2 describe-instances | jq ".Reservations[].Instances[].InstanceId" -r` can be used if `jq` is installed
1. Once instance ID has been identified, run `aws ec2 terminate-instances --instance-ids ` and fill in instance ID value
1. Handy one liner, to terminate all currently running EC2 instances
    ```sh
    aws ec2 describe-instances \
    | jq ".Reservations[].Instances[].InstanceId" -r \
    | xargs -n1 -I instance_id aws ec2 terminate-instances --instance-ids instance_id
    ```
1. If needed, delete security group and key pair if one was dedicated to this exercise
    ```sh
    aws ec2 delete-key-pair --key-name $KEY_NAME
    aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID
    ```

## Exercise 4.5 Inspect AWS Service Health Dashboards
### Up
1. Go to http://status.aws.amazon.com and poke around a bit
1. Go to http://phd.aws.amazon.com and poke around a bit
### Down
1. Log out of console

## Exercise 4.6 Use Elastic IP Address
### Up
1. Use existing or spin up new EC2 instance (Ubuntu Server)
    1. Assign variables
        ```sh
        IMAGE_AMI="ami-04169656fea786776" \
        VPC_ID=$( \
          aws ec2 describe-vpcs | jq -c ".Vpcs | map(select(.IsDefault | contains(true)))[].VpcId" -r \
        ) \
        SG_NAME=YankeeZuluSG \
        SG_DESCRIPTION="Security Group for Yankee Zulu"
        KEY_NAME=YankeeZuluKP \
        SUBNET_ID=$(
          aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" | jq ".Subnets[] | .SubnetId" -r | gshuf | head -n 1 \
        ) \
        SECURITY_GROUP_ID=$( \
          aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[] .GroupId" -r \
        )
        ```
    1. Identify or create security group
        1. If none already exist

            ```sh
            aws ec2 create-security-group --group-name $SG_NAME \
            --description $SG_DESCRIPTION \
            --vpc-id $VPC_ID
            ```
        1. Assign newly created security group ID to variable
            ```sh
            SECURITY_GROUP_ID=$(aws ec2 describe-security-groups | jq ".SecurityGroups | map(select(.GroupName | contains(\"$SG_NAME\")))[] .GroupId" -r)
            ```
        1. Create ingress rule for SSH from anywhere (0.0.0.0/0)
            ```sh
            aws ec2 authorize-security-group-ingress \
            --group-id $SECURITY_GROUP_ID \
            --protocol tcp \
            --port 22 \
            --cidr 0.0.0.0/0
            ```
    1. Identify or create key pair
        1. If none already exist

            ```sh
            aws ec2 create-key-pair --key-name $KEY_NAME  --query 'KeyMaterial' --output text > "${KEY_NAME}.pem"
            ```
        1. Ensure proper permissions exist on newly created .pem file
            ```sh
            chmod 400 "${KEY_NAME}.pem"
            ```
    1. Spin up EC2 instance
        ```sh
        aws ec2 run-instances \
        --image-id $IMAGE_AMI \
        --count 1 \
        --instance-type t2.micro \
        --key-name $KEY_NAME \
        --security-group-ids $SECURITY_GROUP_ID \
        --subnet-id $SUBNET_ID
        ```
    1. Assign instance ID to variable
        ```sh
        INSTANCE_ID=$(aws ec2 describe-instances | jq '.Reservations[].Instances[].InstanceId' -r)
        ```
        TODO: Above two commands could be combined
1. Make note of it's public IP address. This will be changed in later instructions
    1. Run this command to identify EC2 instance public IP address
1. Allocate new Elastic IP address, if none exist
    ```sh
    aws ec2 allocate-address --domain vpc
    ```

    ```sh
    ALLOCATION_ID=$(aws ec2 describe-addresses | jq '.Addresses[].AllocationId' -r)
    ```

    TODO: Above steps could probably be combined into one command. One can take the output of the create command and pipe into a variable.
1. Associate it to your EC2 instance
    ```sh
    aws ec2 associate-address --instance-id $INSTANCE_ID --allocation-id $ALLOCATION_ID
    ```
1. SSH into your EC2 instance from the elastic IP address (this will be different than previous "identify default public address of EC2 instance" step)

### Down
1. Disassociate elastic IP address
1. Release elastic IP address
1. Terminate EC2 instance
1. Remove security group and key pair

## Exercise 4.7 Work with Metadata
### Up
1. Launch EC2 instance
1. Run command `curl http://169.254.169.254/latest/meta-data` to see meta data
1. Add onto above curl command to see more information e.g.
    ```sh
    curl http://169.254.169.254/latest/meta-data/security-groups
    curl http://169.254.169.254/latest/meta-data/public-ipv4
    ```

### Down
1. Close down EC2 instance and any new security groups or keypairs for this exercise

