# Chapter 7 Databases

## Exercise 7.1 Create a New Option Group
### Up
1. Set option group name variable and create option group for... Hmmm, MySQL... And the current latest version at time of writing is 5.7
    ```
    OPTION_GROUP_NAME="test-option-group-$(openssl rand -hex 18)"

    aws rds create-option-group --option-group-name $OPTION_GROUP_NAME --engine-name mysql --major-engine-version 5.7 --option-group-description $(openssl rand -hex 64)
    ```

### Down
1. Delete option group
    ```
    aws rds delete-option-group --option-group-name $OPTION_GROUP_NAME
    ```

## Exercise 7.2 Create a DynamoDB Table
### Up
### Down

## Exercise 7.3 Add Items to DynamoDB Table
### Up
### Down

## Exercise 7.4 Create MySQL RDS DB Instance
### Up
### Down

