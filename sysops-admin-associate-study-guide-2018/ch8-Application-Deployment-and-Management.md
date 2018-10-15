# Chapter 8 Application Deployment and Management

## Exercise 8.1 Create an Elastic Beanstalk Environment, Exercise 8.2 Manage Application Versions with Elastic Beanstalk
### Up
1. Set up variables used in later commands
```sh
ENVIRONMENT_NAME="test-eb-env-$(openssl rand -hex 8)"
APPLICATION_NAME="test-eb-app-$(openssl rand -hex 8)"
TEMPLATE_NAME="test-eb-temp-$(openssl rand -hex 8)"
VERSION_LABEL="version-$(openssl rand -hex 16)"
CNAME_PREFIX="test-cname-$(openssl rand -hex 8)"
RANDOM_BUCKET_NAME="test-bucket-$(openssl rand -hex 18)"
BUNDLE_KEY=my-source-bundle.zip
```
1. Create new S3 bucket
```sh
aws s3 mb "s3://${RANDOM_BUCKET_NAME}"
```
1. Create dummy source bundle
    1. Create ch8 directory
    ```sh
    mkdir -p ch8
    ```
    1. Create simple Flask application
        1. Use `ch8` as your current working directory
        ```sh
        cd ch8
        ```
        1. Create and activate virtual environment
        ```sh
        virtualenv virt
        source virt/bin/activate
        ```
        1. Install flask and freeze requirements
        ```sh
        pip install flask==1.0.2
        pip freeze > requirements.txt
        ```
        1. Create `application.py` and paste following python code into file
        ```python
		from flask import Flask

        # print a nice greeting.
		def say_hello(username = "World"):
			return '<p>Hello %s!</p>\n' % username

        # some bits of text for the page.
		header_text = '''
			<html>\n<head> <title>EB Flask Test</title> </head>\n<body>'''
		instructions = '''
			<p><em>Hint</em>: This is a RESTful web service! Append a username
			to the URL (for example: <code>/Thelonious</code>) to say hello to
			someone specific.</p>\n'''
		home_link = '<p><a href="/">Back</a></p>\n'
		footer_text = '</body>\n</html>'

        # EB looks for an 'application' callable by default.
		application = Flask(__name__)

        # add a rule for the index page.
		application.add_url_rule('/', 'index', (lambda: header_text +
			say_hello() + instructions + footer_text))

        # add a rule when the page is accessed with a name appended to the site
        # URL.
		application.add_url_rule('/<username>', 'hello', (lambda username:
			header_text + say_hello(username) + home_link + footer_text))

        # run the app.
		if __name__ == "__main__":
			# Setting debug to True enables debug output. This line should be
			# removed before deploying a production app.
			application.debug = True
			application.run()
        ```
        1. Test by running `python application.py`
    1. Initialize `.elasticbeanstalk/*` folder using awsebcli
    ```sh
    eb init $APPLICATION_NAME -p python-3.6 --region us-east-1
    ```
    1. Zip up ch8 folder into a bundle
    ```sh
    zip -r $BUNDLE_KEY ch8
    ```
1. Move dummy source bundle into newly created S3 bucket
```sh
aws s3 cp $BUNDLE_KEY "s3://${RANDOM_BUCKET_NAME}/${BUNDLE_KEY}"
```
1. Create an Elastic Beanstalk application
```sh
aws elasticbeanstalk create-application --application-name $APPLICATION_NAME
```
1. Create an Elastic Beanstalk application version
```sh
aws elasticbeanstalk create-application-version --application-name $APPLICATION_NAME --version-label $VERSION_LABEL --source-bundle S3Bucket=$RANDOM_BUCKET_NAME,S3Key=$BUNDLE_KEY
```
1. Create configuration template for application
```sh
aws elasticbeanstalk create-configuration-template --application-name $APPLICATION_NAME --template-name $TEMPLATE_NAME --solution-stack-name "64bit Amazon Linux 2018.03 v2.7.3 running Python 3.6"
```
(Python solution stack selected because why not. Find more solution stacks by running `aws elasticbeanstalk list-available-solution-stacks`)
1. Create environment
```sh
aws elasticbeanstalk create-environment --cname-prefix $CNAME_PREFIX --application-name $APPLICATION_NAME --template-name $TEMPLATE_NAME --version-label $VERSION_LABEL --environment-name $ENVIRONMENT_NAME
```
1. Query every now and again until the environment is green
```sh
aws elasticbeanstalk describe-environments --environment-names $ENVIRONMENT_NAME
```
1. Deploy code
```sh
cd ch8
eb deploy
```
1. Open application in browser
```sh
eb open
```

### Down
1. Terminate environment
```sh
aws elasticbeanstalk terminate-environment --environment-name $ENVIRONMENT_NAME
```
1. Delete configuration template
```sh
aws elasticbeanstalk delete-configuration-template --application-name $APPLICATION_NAME --template-name $TEMPLATE_NAME
```
1. Delete application version
```sh
aws elasticbeanstalk delete-application-version --application-name $APPLICATION_NAME --version-label $VERSION_LABEL
```
1. Delete application
```sh
aws elasticbeanstalk delete-application --application-name $APPLICATION_NAME
```
1. Delete S3 bucket
```sh
aws s3 rm "s3://${RANDOM_BUCKET_NAME}" --recursive
aws s3api delete-bucket --bucket "${RANDOM_BUCKET_NAME}"
```

## Exercise 8.3 Perform a Blue/Green Deployment with Elastic Beanstalk
### Up
1. Follow exercise 8.1/8.2 to spin up EB environment
1. Clone environment
### Down
1. Follow exercise 8.1/8.2 to spin down EB environment

## Exercise 8.4 Create an ECS Cluster
### Up
### Down

## Exercise 8.5 Launch an EC2 Instance Optimized for ECS
### Up
### Down

## Exercise 8.6 Use ECR
### Up
### Down

## Exercise 8.7 Work with ECS Task Definitions
### Up
### Down

## Exercise 8.8 Work with ECS Services
### Up
### Down

## Exercise 8.9 Create an OpsWorks Stack
### Up
### Down

## Exercise 8.10 Make a Layer in OpsWorks Stacks
### Up
### Down

## Exercise 8.11 Add an EC2 Instance to an OpsWorks Stacks Layer
### Up
### Down

## Exercise 8.12 Add an Application to OpsWorks Stacks
### Up
### Down

## Exercise 8.13, 8.14 Create a CloudFormation Stack then Delete it
### Up
### Down

