# Building a Serverless Web Application on AWS

This project demonstrates the development of a serverless web application on AWS, utilizing a variety of AWS services such as AWS Lambda, API Gateway, DynamoDB, and Amazon Cognito for authentication. The project is deployed using the AWS Amplify Console, with the code stored in a GitHub repository.

## Project Overview

This serverless application uses the following AWS resources:

1. **AWS Amplify Console:** Used to deploy the web application directly from the GitHub repository.
2. **Amazon Cognito:** Manages user authentication and authorization through a Cognito User Pool.
3. **Amazon DynamoDB:** A DynamoDB table, named `Rides`, stores requests sent by the Lambda function. The table uses `RideId` as the partition key.
4. **IAM Role:** Grants the Lambda function permission to write logs to Amazon CloudWatch Logs and store items in the DynamoDB table.
5. **AWS Lambda:** A Lambda function, `RequestUnicorn`, processes API requests to dispatch a unicorn and interacts with DynamoDB.
6. **API Gateway:** Invokes the Lambda function through a REST API and integrates with Amazon Cognito for secure access.
![Alt text](/Building-a-serverless-webapp-on-aws.png)

## Prerequisites

- AWS Account
- A GitHub repository with the application code
- Proper IAM role configurations to allow Lambda access to DynamoDB and CloudWatch Logs

## Key Components

### AWS Amplify Deployment

The web application is deployed using the AWS Amplify Console, which automatically pulls the source code from the GitHub repository.

### Amazon Cognito

- Created a **Cognito User Pool** to authenticate users.
- Cognito manages user information and generates JSON Web Tokens (JWT) for authentication purposes.

### Amazon DynamoDB

- A DynamoDB table, `Rides`, is used to store ride requests sent by the Lambda function.
- The table has a partition key `RideId` of type `String`.

### AWS Lambda Function

The `RequestUnicorn` Lambda function processes requests sent from the web app. The function selects a unicorn from a predefined list and logs the ride request in DynamoDB.

#### Lambda Function Code

```javascript
const randomBytes = require('crypto').randomBytes;
const AWS = require('aws-sdk');
const ddb = new AWS.DynamoDB.DocumentClient();

const fleet = [
    { Name: 'Taylor', Color: 'White', Gender: 'Female' },
    { Name: 'Jacob', Color: 'Black', Gender: 'Male' },
    { Name: 'Ma', Color: 'Yellow', Gender: 'Female' },
];

exports.handler = (event, context, callback) => {
    if (!event.requestContext.authorizer) {
        errorResponse('Authorization not configured', context.awsRequestId, callback);
        return;
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;
    const unicorn = findUnicorn(pickupLocation);

    recordRide(rideId, username, unicorn).then(() => {
        callback(null, {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: { 'Access-Control-Allow-Origin': '*' },
        });
    }).catch((err) => {
        console.error(err);
        errorResponse(err.message, context.awsRequestId, callback);
    });
};

function findUnicorn(pickupLocation) {
    console.log('Finding unicorn for ', pickupLocation.Latitude, ', ', pickupLocation.Longitude);
    return fleet[Math.floor(Math.random() * fleet.length)];
}

function recordRide(rideId, username, unicorn) {
    return ddb.put({
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    }).promise();
}

function toUrlString(buffer) {
    return buffer.toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId, callback) {
    callback(null, {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: { 'Access-Control-Allow-Origin': '*' },
    });
}
```

### Testing the Lambda Function

To test the Lambda function, use the following test event in the AWS Lambda Console:

```json
{
  "path": "/ride",
  "httpMethod": "POST",
  "headers": {
    "Accept": "*/*",
    "Authorization": "eyJraWQiOiJLTzRVMWZs",
    "content-type": "application/json; charset=UTF-8"
  },
  "queryStringParameters": null,
  "pathParameters": null,
  "requestContext": {
    "authorizer": {
      "claims": {
        "cognito:username": "the_username"
      }
    }
  },
  "body": "{\"PickupLocation\":{\"Latitude\":47.6174755835663,\"Longitude\":-122.28837066650185}}"
}
```

### API Gateway Setup

1. **Create a new REST API** using API Gateway.
2. **Create an Authorizer** to integrate API Gateway with Amazon Cognito for secure access.
3. **Create a Resource and POST Method** in API Gateway that triggers the Lambda function.
4. **Deploy the API** in API Gateway and update the web app configuration to use the new Invoke URL.

## Deployment Instructions

1. Clone the GitHub repository to your local machine or deploy it directly via the **AWS Amplify Console**.
2. Set up the necessary AWS resources (Cognito, DynamoDB, Lambda, API Gateway) following the project architecture.
3. Update the configuration file with the new API Gateway Invoke URL.
4. Test the application end-to-end to ensure functionality.

## Conclusion

This project demonstrates how to build a fully serverless web application using AWS services like Lambda, API Gateway, DynamoDB, and Cognito. By leveraging the power of serverless architecture, the application scales automatically and reduces operational overhead, allowing for high availability and seamless performance.

For more details, visit the [GitHub Repository](https://github.com/Shawnty-z/Serverless-web-app-on-aws).

---

This README should give a clear overview of the project, allowing others to follow along with your implementation and deployment process.
