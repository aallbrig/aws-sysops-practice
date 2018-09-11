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
1. Create input JSON for table definition. I named my file `create-dynamo-db-table.json` and put inside `ch7` directory.
    ```
    {
        "AttributeDefinitions": [
            {
                "AttributeName": "Artist",
                "AttributeType": "S"
            },
            {
                "AttributeName": "Title",
                "AttributeType": "S"
            }
        ],
        "TableName": "MusicCollection",
        "KeySchema": [
            {
                "AttributeName": "Artist",
                "KeyType": "HASH"
            },
            {
                "AttributeName": "Title",
                "KeyType": "RANGE"
            }
        ],
        "ProvisionedThroughput": {
            "ReadCapacityUnits": 1,
            "WriteCapacityUnits": 1
        }
    }
    ```
1. Run create command referencing your previously created json file and using `query` to extract the table name.
    ```
    TABLE_NAME=$(aws dynamodb create-table --cli-input-json file://ch7/create-dynamo-db-table.json --query 'TableDescription.TableName' --output text)
    ```
### Down
1. Delete table
    ```
    aws dynamodb delete-table --table-name $TABLE_NAME
    ```

## Exercise 7.3 Add Items to DynamoDB Table
### Up
### Down

## Exercise 7.4 Create MySQL RDS DB Instance
### Up
### Down

