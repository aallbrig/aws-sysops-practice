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
    VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 | jq '.Vpc.VpcId' -r)
    ```
1. Create public subnet
    1. Create a subnet

        ```
        PUBLIC_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.0.0/24 | jq '.Subnet.SubnetId' -r)
        ```
    1. Create internet gateway

        ```
        INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway | jq '.InternetGateway.InternetGatewayId' -r)
        ```
    1. Attach internet gateway to VPC

        ```
        aws ec2 attach-internet-gateway --vpc-id $VPC_ID --internet-gateway-id $INTERNET_GATEWAY_ID 
        ```
    1. Create custom route table for VPC
        
        ```
        ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID | jq '.RouteTable.RouteTableId' -r)
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

1. Create private subnet
    ```
    PRIVATE_SUBNET_ID=$(aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 | jq '.Subnet.SubnetId' -r)
    ```
### Down

