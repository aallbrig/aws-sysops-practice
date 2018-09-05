# Chapter 5 Networking

## Exercise 5.1 Create an Elastic IP
### Up
1. Allocate Elastic IP address

    ```
    aws ec2 allocate-address
    ```
1. Make note of IP address

    ```
    aws ec2 describe-addresses | jq '.Addresses[].PublicIp'
    ```

### Down
1. Disassociate Elastic IP address

    ```
    # Identify allocation ID
    aws ec2 describe-addresses
    # Disassociate address
    aws ec2 disassociate-address
    ```
1. Release Elastic IP address
    ```
    # Identify allocation ID
    aws ec2 describe-addresses
    # Release address
    aws ec2 release-address --allocation-id
    ```

## Exercise 5.2a Create VPC from management console
### Up
1. Start the VPC wizard from VPC section in AWS management console
1. Select "VPC with public and private subnets"
1. Name VPC "VPC-1" and cidr block of 10.0.0.16
1. Assign public cidr block of 10.0.0.0/24 and no preference on availability zone
1. Assign private cidr block of 10.0.1.0/24 and no preference on availability zone
1. Assign NAT gateway and use the elastic IP address from Exercise 5.1
1. Assign an S3 endpoint and leave default DNS and tenancy options

### Down
1. In "Your VPCs" section, find your VPC, select it,  and execute "Actions>Delete VPC"

## Exercise 5.2b Create VPC from AWS CLI
### Up
1. Create VPC and assign ID to variable

    ```
    VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
    ```
1. Create public subnet
    1. Create a subnet

        ```
        PUBLIC_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.0.0/24 --query 'Subnet.SubnetId' --output text)
        ```
    1. Create internet gateway

        ```
        INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
        ```
    1. Attach internet gateway to VPC

        ```
        aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID 
        ```
    1. Create custom route table for VPC
        
        ```
        ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
        ```
    1. Create route that points all traffic to internet gateway

        ```
        aws ec2 create-route \
        --route-table-id $ROUTE_TABLE_ID \
        --gateway-id $INTERNET_GATEWAY_ID \
        --destination-cidr-block 0.0.0.0/0
        ```

        Confirm the desired route exists by running...

        ```
        aws ec2 describe-route-tables --route-table-id $ROUTE_TABLE_ID
        ```
    1. Associate subnet with route table

        ```
        aws ec2 associate-route-table --subnet-id $PUBLIC_SUBNET_ID --route-table-id $ROUTE_TABLE_ID
        ```
    1. Map public IP on launch when launching into public subnet
        
        ```
        aws ec2 modify-subnet-attribute --subnet-id $PUBLIC_SUBNET_ID --map-public-ip-on-launch
        ```
    1. (Test) Create EC2 instance in newly created VPC and verify internet access
        1. Create keypair, if needed

            ```
            KEY_NAME=TestTangoKP

            aws ec2 create-key-pair --key-name $KEY_NAME --query 'KeyMaterial' --output text > "${KEY_NAME}.pem"

            chmod 400 "${KEY_NAME}.pem"
            ```
        1. Create security group and SSH ingress rule, if needed

            ```
            SECURITY_GROUP_ID=$(aws ec2 create-security-group --vpc-id $VPC_ID --group-name TestTangoSG --description "SSH access for TestTango instance." --query 'GroupId' --output text)

            aws ec2 authorize-security-group-ingress --group-id $SECURITY_GROUP_ID --protocol tcp --port 22 --cidr 0.0.0.0/0
            ```
        1. Launch instance into public subnet

            ```
            INSTANCE_ID=$(aws ec2 run-instances --image-id ami-04169656fea786776  --count 1 --instance-type t2.micro --key-name TestTangoKP --security-group-ids $SECURITY_GROUP_ID --subnet-id $PUBLIC_SUBNET_ID --query 'Instances[0].InstanceId' --output text)
            ```
        1. SSH into instance and verify internet access

1. Create private subnet
    ```
    PRIVATE_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --query 'Subnet.SubnetId' --output text)
    ```
### Down
1. Terminate test EC2 instance, if applicable

    ```
    aws ec2 terminate-instances --instance-ids $INSTANCE_ID
    ```
1. Delete security group

    ```
    aws ec2 delete-security-group --group-id $SECURITY_GROUP_ID
    ```
1. Delete key pair

    ```
    aws ec2 delete-key-pair --key-name $KEY_NAME
    ```

    If located in same directory, remove the key pair's ".pem" file too

    ```
    rm "${KEY_NAME}.pem"
    ```
1. Delete subnets (public, private)

    ```
    aws ec2 delete-subnet --subnet-id $PRIVATE_SUBNET_ID

    aws ec2 delete-subnet --subnet-id $PUBLIC_SUBNET_ID
    ```
1. Delete custom route table

    ```
    aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID
    ```
1. Detach internet gateway from VPC

    ```
    aws ec2 detach-internet-gateway --internet-gateway-id $INTERNET_GATEWAY_ID --vpc-id $VPC_ID
    ```
1. Delete internet gateway

    ```
    aws ec2 delete-internet-gateway --internet-gateway-id $INTERNET_GATEWAY_ID
    ```
1. Delete VPC

    ```
    aws ec2 delete-vpc --vpc-id $VPC_ID
    ```

## Exercise 5.3 Tag Your VPC and Subnets
### Up
1. Spin up a new VPC, with private & public subnet from exercise 5.2b
1. Through AWS management console (because piecing together ARNs is a beast), find your VPC and tag it
1. Through AWS management console, find your VPC's subnets and tag them

### Down
1. Delete tags from your VPC
1. Spin down your VPC, private & public subnet

## Exercise 5.4, 5.5, 5.6, 5.7 Create an Elastic Network Interface (ENI), associate EIP with ENI, test ENI, delete VPC
### Up
1. Spin up a new VPC, with private subnet, public subnet, and EC2 instance from exercise 5.2b
1. Create Elastic IP address

    ```
    ALLOCATION_ID=$(aws ec2 allocate-address --query AllocationId --output text)
    ```
1. Create a network interface; use your VPC's public subnet and use a security group (default or new)

    ```
    NETWORK_INTERFACE_ID=$(aws ec2 create-network-interface --groups $SECURITY_GROUP_ID --subnet-id $PUBLIC_SUBNET_ID --query 'NetworkInterface.NetworkInterfaceId' --output text)
    ```
1. Tag your ENI via AWS management console
1. Associate EIP to ENI

    ```
    ASSOCIATION_ID=$(aws ec2 associate-address --allocation-id $ALLOCATION_ID --network-interface-id $NETWORK_INTERFACE_ID --query 'AssociationId' --output text)
    ```
1. Attach network interface to EC2 instance

    ```
    ATTACHMENT_ID=$(aws ec2 attach-network-interface --network-interface-id $NETWORK_INTERFACE_ID --instance-id $INSTANCE_ID --device-index 1 --query 'AttachmentId' --output text)
    ```
### Down
1. Delete your tag
1. Delete your ENI
1. Release your Elastic IP
1. Spin down VPC following 5.2b Down instructions

