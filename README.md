# AWS Serverless-API
High-level overview and design

![image](https://github.com/user-attachments/assets/10a3538a-2e2c-4e76-b410-b0bd71593742)

An Amazon API Gateway serves as a bridge between clients and backend services, enabling users to expose AWS Lambda functions, HTTP endpoints, and other AWS services through RESTful and WebSocket APIs. Amazon API Gateway consists of a set of resources and corresponding methods that enable the creation, management, and execution of APIs.

In this setup, an API called DynamoDBManager is created with a POST method. This method is powered by a Lambda function named LambdaFunctionOverHttps, which is responsible for inserting data into a DynamoDB table called lambda-apigateway.

The POST method in the DynamoDBManager API supports the following DynamoDB operations:

✔ Create an item 📝

✔ Read an item 📖

✔ List all items 📋

Additionally, it includes built-in logging and monitoring through Amazon CloudWatch, ensuring better visibility and performance tracking.

The JSON request sent in the POST request specifies the desired operation and includes the relevant data needed for execution.

✅ Suggestion: Use Postman, a third-party application, to send HTTP requests and interact with the DynamoDBManager API efficiently.

# SETUP
**Creating an IAM Role for Lambda Execution**

To allow your Lambda function to access AWS resources, you need to create an IAM execution role with the necessary permissions.

Steps to Create an Execution Role:

1️⃣ Open the IAM console and navigate to the Roles page.

2️⃣ Click on Create role.

3️⃣ Configure the role with the following properties:

**Trusted entity**: AWS Lambda

**Role name**: lambda-apigateway-role

**Permissions**: Attach a **custom policy** that grants access to **DynamoDB** for data operations and **CloudWatch Logs** for monitoring.

This role ensures that the Lambda function can securely write data to **DynamoDB** and log events to **CloudWatch**.

Below is a JSON for IAM policy:

    {
    "Version": "2012-10-17",
    "Statement": [
    {
    "Sid": "Stmt1428341300017",
    "Action": [  
    "dynamodb:DeleteItem",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateItem"
    ],
    "Effect": "Allow",
    "Resource": "*"
    },
    {
    "Sid": "",
    "Resource": "*", 
    "Action": [ 
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
    ], 
    "Effect": "Allow"
    }
    ]
    }

 **Create Lambda Function**

 Creating a New Lambda Function on AWS

Follow these steps to create a Lambda function from scratch:

1️⃣ Navigate to AWS Lambda

Open the AWS Console and go to the Lambda service.

2️⃣ Create a New Function

Click on "Create Function".

![image](https://github.com/user-attachments/assets/4a039793-6b53-4a56-9f87-b7e87d68f934)

3️⃣ Choose Function Settings

Select "Author from Scratch".

Enter the function name as "LambdaFunctionOverHttps".

Set the runtime to Python 3.13.

Choose architecture as x86_64.

4️⃣ Finalize and Create

Click on "Create Function" to complete the setup.

✅ Your Lambda function is now created and ready for further configuration! 🚀

![image](https://github.com/user-attachments/assets/5f07dec9-e44e-490a-a4fb-c097927b50a6)

Click on the Lambda function and replace the boilerplate coding with below code

      from __future__ import print_function
      
      import boto3
      import json
      
      print('Loading function')
      
      
      def lambda_handler(event, context):
          '''Provide an event that contains the following keys:

          - operation: one of the operations in the operations dict below
          - tableName: required for operations that interact with DynamoDB
          - payload: a parameter to pass to the operation being performed
        '''
        #print("Received event: " + json.dumps(event, indent=2))
    
        operation = event['operation']
    
        if 'tableName' in event:
            dynamo = boto3.resource('dynamodb').Table(event['tableName'])
    
        operations = {
            'create': lambda x: dynamo.put_item(**x),
            'read': lambda x: dynamo.get_item(**x),
            'update': lambda x: dynamo.update_item(**x),
            'delete': lambda x: dynamo.delete_item(**x),
            'list': lambda x: dynamo.scan(**x),
            'echo': lambda x: x,
            'ping': lambda x: 'pong'
        }
    
        if operation in operations:
            return operations[operation](event.get('payload'))
        else:
            raise ValueError('Unrecognized operation "{}"'.format(operation))

**Testing Your Lambda Function (Echo Test)**

Before deploying, let's run a quick echo test to ensure the function is working correctly. Since we haven't set up API Gateway or DynamoDB yet, the function should simply return whatever input we provide.

Steps to Test the Lambda Function:

1️⃣ Open your Lambda function in the AWS Console.

2️⃣ Click on the "Test" tab.

3️⃣ Create a new test event by clicking "Create a new test event".

4️⃣ Paste the provided JSON into the event input field.

5️⃣ Click "Test" to execute the function.

    {
        "operation": "echo",
        "payload": {
            "somekey1": "somevalue1",
            "somekey2": "somevalue2"
        }
    }

Click "Save"

![image](https://github.com/user-attachments/assets/906d431e-c523-4391-93d3-52235730c75a)

Click "Test", and it will execute the test event. You should see the output in the console

Once you are done with the testing, click on "Deploy"

We're all set to create DynamoDB table and an API using our lambda as backend!

**Create DynamoDB Table**

Follow these steps to set up a DynamoDB table that your Lambda function will use:

1️⃣ Open the DynamoDB Console

Navigate to the AWS Management Console.

Search for DynamoDB and open the service.

2️⃣ Create a New Table

Click on "Create Table".

![image](https://github.com/user-attachments/assets/b3487a26-cca4-4479-a899-fc6c4c5a8f5b)


3️⃣ Configure Table Settings

Table Name: lambda-apigateway

Primary Key: id (Data type: String)

4️⃣ Finalize the Table Creation

Click "Create" to generate the table.

✅ Your DynamoDB table is now set up and ready to store data from the Lambda function! 🚀

![image](https://github.com/user-attachments/assets/04e45a11-044c-4dca-b5e6-488c90295f2a)

**Create API**

**Creating the DynamoDBOperations API in API Gateway**

Follow these steps to set up the DynamoDBOperations API in AWS API Gateway:

1️⃣ Open API Gateway Console

Navigate to the AWS Console and open the API Gateway service.

2️⃣ Create a New API

Click on "Create API".

![image](https://github.com/user-attachments/assets/5e019585-f545-47c0-8452-91e9e70bf98d)

3️⃣ Choose API Type

Select "REST API".

Choose "New API".

4️⃣ Configure API Settings

API Name: DynamoDBOperations

Description (optional): API to perform CRUD operations on DynamoDB.

![image](https://github.com/user-attachments/assets/90678c7a-476b-429e-9b84-be4940bb3fcc)

API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. 

let's add a resource next. click on "Create resource"

![image](https://github.com/user-attachments/assets/a18aac82-e043-4f86-8ceb-f927b9a77e8b)

Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create resource"

![image](https://github.com/user-attachments/assets/49e479ff-93be-49a3-a2f1-abb1a7c45dcc)

Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".

![image](https://github.com/user-attachments/assets/b3f9abbf-86b5-46e8-9fe8-d17d3d23abb1)

Adding a POST Method to the API Resource
Now that we have created a resource (e.g., /items) in API Gateway, let's add a POST method to integrate it with our Lambda function.

Steps to Add a POST Method:

1️⃣ Open the DynamoDBOperations API in the API Gateway Console.

2️⃣ In the left panel, select the newly created resource (e.g., /items).

3️⃣ Click "Create Method", and from the dropdown, select POST.

4️⃣ Configure the method:

Integration type: Select Lambda Function.

Lambda function name: Enter LambdaFunctionOverHttps (the function we created earlier).

5️⃣ Click "Create Method" to save the configuration.

✅ POST method successfully created! This setup allows API Gateway to invoke the Lambda function, which will handle requests and interact with DynamoDB. 🚀

![image](https://github.com/user-attachments/assets/fe457309-85d1-4cb2-9a2e-5337c4b6a018)

Deploying the API to a Stage

Now that we have configured our DynamoDBOperations API, it’s time to deploy it to make it accessible.

Steps to Deploy the API:

1️⃣ Open the API Gateway Console and select your DynamoDBOperations API.

2️⃣ Click on the "Actions" dropdown.

3️⃣ Select "Deploy API".

![image](https://github.com/user-attachments/assets/c137971f-ba7b-4424-8d21-04fbf16b5994)

4️⃣ In the Deployment Stage section:

Choose or Create a Stage: Select "New Stage".

Stage Name: Enter prod (short for production).

5️⃣ Click "Deploy" and copy the "Invoke URL" from screen

**Running our solution using Postman application**

The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON:

    {
        "operation": "create",
        "tableName": "lambda-apigateway",
        "payload": {
            "Item": {
                "id": "1234ABCD",
                "number": 5
            }
        }
    }

Postman is a powerful tool for API development and testing, making it easier to send requests, analyze responses, and manage API workflows. Follow these steps to test your deployed API using Postman.

1️⃣ Create a New Collection

Open Postman.

Click "New" → "Collection".

Name it something meaningful, e.g., DynamoDBOperations API.

2️⃣ Add a New Request to the Collection

Click the three dots (...) next to the collection name.

Select "Add Request".

Name the request "Create Item" (or another relevant name).

3️⃣ Configure the API Request

In the request editor, select "POST" from the method dropdown.

Copy the "Invoke URL" from the PROD stage of API Gateway.

Paste the URL in the request URL field.

✅ Your Postman request is now set up! 🚀

![image](https://github.com/user-attachments/assets/2428b361-3eef-4dea-a8f4-472eae6e3ab0)

To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Items" tab, and the newly inserted item should be displayed.

![image](https://github.com/user-attachments/assets/0029c838-c095-414c-b010-5ae7644d52a1)

To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

    {
        "operation": "list",
        "tableName": "lambda-apigateway",
        "payload": {
        }
    }


**Load Tesing & Measuring Performance**

Postman enables you to simulate user traffic and observe how your API behaved under load. It also helps to identify any issues or bottlenecks that affect performance.

**Configuration**

TImeout: 5 sec to 3 sec

Limit instances: 10

Memeory: 128MB to 1024MB

Reserved Concurrency: 10

Performance Results

![image](https://github.com/user-attachments/assets/ad6447bb-b2c5-4dba-9111-f0a6d88e002a)


![image](https://github.com/user-attachments/assets/f7eabcc7-5b67-4686-bcb8-3407660fc4b1)

Observed significant performance boost after adjusting Lambda configurations. This enabled average response time improved drastically from 630ms to 147ms! 







