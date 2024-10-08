# AWS CRUD HTTP API with API Gateway, Lambda, and DynamoDB

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Architecture Overview](#architecture-overview)
- [Step-by-Step Implementation](#step-by-step-implementation)
  - [1. Set Up DynamoDB](#1-set-up-dynamodb)
  - [2. Create Lambda Functions](#2-create-lambda-functions)
  - [3. Configure API Gateway](#3-configure-api-gateway)
  - [4. Test the API](#4-test-the-api)
- [Benefits of Serverless Architecture](#benefits-of-serverless-architecture)
- [Conclusion](#conclusion)

## Overview
This tutorial demonstrates how to create a serverless **CRUD HTTP API** using **AWS API Gateway**, **Lambda**, and **DynamoDB**. By following these steps, you can implement a scalable solution for handling HTTP requests without managing any infrastructure.

The CRUD operations covered include:

- **Create**: Add new items to DynamoDB.
- **Read**: Retrieve items.
- **Update**: Modify existing items.
- **Delete**: Remove items.

## Prerequisites
Before you begin, ensure you have the following:

- An **AWS account** with sufficient privileges.
- Basic knowledge of **Lambda** and **DynamoDB**.
- Installed tools: AWS CLI, AWS SDK, Postman (for API testing).

## Architecture Overview
This project leverages three primary AWS services:

- **API Gateway**: Accepts HTTP requests and routes them to Lambda functions.
- **AWS Lambda**: Processes the logic for each CRUD operation.
- **DynamoDB**: Stores data in a fast, flexible NoSQL database.

### Diagram
```plaintext
   Client
      |
      v
API Gateway ---> Lambda (Node.js/Python) ---> DynamoDB
```

## Step-by-Step Implementation

### 1. Set Up DynamoDB
1. Go to the **DynamoDB Console**.
2. Create a table with a partition key (e.g., `ID`).
3. Define your attributes, such as `Name`, `Age`, etc.

### 2. Create Lambda Functions
You will create separate Lambda functions for each CRUD operation. Here's an example using Node.js to interact with DynamoDB:

#### a. Create Item (POST)
```javascript
const AWS = require('aws-sdk');
const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const params = {
        TableName: 'YourTableName',
        Item: {
            'ID': event.id,
            'Name': event.name
        }
    };
    
    try {
        await dynamo.put(params).promise();
        return { statusCode: 200, body: JSON.stringify('Item Created!') };
    } catch (err) {
        return { statusCode: 500, body: JSON.stringify(err) };
    }
};
```

#### b. Read Item (GET)
```javascript
const AWS = require('aws-sdk');
const dynamo = new AWS.DynamoDB.DocumentClient();

exports.handler = async (event) => {
    const params = {
        TableName: 'YourTableName',
        Key: { 'ID': event.id }
    };
    
    try {
        const data = await dynamo.get(params).promise();
        return { statusCode: 200, body: JSON.stringify(data) };
    } catch (err) {
        return { statusCode: 500, body: JSON.stringify(err) };
    }
};
```

Repeat this for **Update** and **Delete** operations.

### 3. Configure API Gateway
1. Go to the **API Gateway Console**.
2. Create a new HTTP API.
3. Define **resources** (paths) and **methods** (GET, POST, PUT, DELETE) for each Lambda function.
4. Use **Lambda Proxy Integration** to map each HTTP method to its corresponding Lambda.

### 4. Test the API
You can test your API using **Postman** or curl commands. For example, to test the GET method:
```bash
curl -X GET https://your-api-url.com/item/123
```

Use different HTTP methods to verify each operation. Ensure that items are created, read, updated, and deleted in DynamoDB as expected.

## Benefits of Serverless Architecture

- **Scalability**: AWS Lambda scales automatically with demand.
- **Cost-Effective**: You only pay for the compute time you consume.
- **Fully Managed**: No need to manage servers or handle scaling.
- **Seamless Integration**: Easy integration between API Gateway, Lambda, and DynamoDB.

## Conclusion
Using AWS services like API Gateway, Lambda, and DynamoDB to build a serverless CRUD API streamlines development while handling scalability and infrastructure needs behind the scenes. This is ideal for building robust, cost-efficient applications in the cloud.
