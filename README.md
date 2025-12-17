# AWS Project – Built a Full End-to-End Serverless Web Application Using 7 AWS services

## TL;DR

I built a **fully serverless web application** for a unicorn ride-sharing service called **Wild Rydes**, based on the original [AWS Serverless Wild Rydes Workshop](https://aws.amazon.com/serverless-workshops/).
The application integrates **AWS Amplify, IAM, Amazon Cognito, AWS Lambda, Amazon API Gateway, and Amazon DynamoDB**, with source code hosted on GitHub and deployed using a CI/CD pipeline via AWS Amplify.

Users can create an account, log in securely, and request a ride by clicking on a map powered by ArcGIS.
This project was implemented as a **hands-on learning exercise** to understand real-world serverless architectures on AWS.

---

## Acknowledgement

This project was developed by following AWS learning material and tutorials by **Amber Israelsen**.

YouTube Playlist:
[https://youtube.com/playlist?list=PLwyXYwu8kL0wMalR9iXJIPfiMYWNFWQzx](https://youtube.com/playlist?list=PLwyXYwu8kL0wMalR9iXJIPfiMYWNFWQzx)

---

## Architecture Overview

The application follows a serverless, event-driven architecture:

* Frontend hosted using AWS Amplify
* Authentication handled by Amazon Cognito
* REST API managed through Amazon API Gateway
* Backend logic executed via AWS Lambda
* Ride request data stored in Amazon DynamoDB
* Access controlled using AWS IAM roles and policies

---

## AWS Services Used

* AWS Amplify (Frontend hosting and CI/CD)
* Amazon Cognito (User authentication)
* AWS Lambda (Backend logic)
* Amazon API Gateway (API routing)
* Amazon DynamoDB (NoSQL database)
* AWS IAM (Access control and permissions)
* GitHub (Source code management)

---

## Cost

All services used are eligible for the **AWS Free Tier**.

Outside of the Free Tier, minimal charges (typically less than $1 USD) may occur during development and testing.
To avoid recurring charges, the deployed application and AWS resources were **terminated after successful testing**.

---

## The Application Code

All frontend and backend source code used in this project is available in this repository.

---

## Implementation Summary

### Frontend

* Deployed using AWS Amplify
* Integrated with GitHub for continuous deployment
* Configuration updated to connect with Cognito and API Gateway

### Authentication

* Amazon Cognito User Pool created
* User Pool ID and Client ID configured in frontend
* Secure sign-up and sign-in flow verified

### Database

* Amazon DynamoDB table created with `RideId` as the partition key
* Used to store ride request details from Lambda

### Backend

* AWS Lambda function implemented using Node.js 20.x
* IAM execution role configured with least-privilege permissions
* Lambda invoked through API Gateway with Cognito authorizer

---

## The Lambda Function Code

The Lambda function logic is adapted from the official AWS workshop and updated for **Node.js 20.x**:

```node
import { randomBytes } from 'crypto';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const ddb = DynamoDBDocumentClient.from(client);

const fleet = [
    { Name: 'Angel', Color: 'White', Gender: 'Female' },
    { Name: 'Gil', Color: 'White', Gender: 'Male' },
    { Name: 'Rocinante', Color: 'Yellow', Gender: 'Female' },
];

export const handler = async (event, context) => {
    if (!event.requestContext.authorizer) {
        return errorResponse('Authorization not configured', context.awsRequestId);
    }

    const rideId = toUrlString(randomBytes(16));
    console.log('Received event (', rideId, '): ', event);

    const username = event.requestContext.authorizer.claims['cognito:username'];
    const requestBody = JSON.parse(event.body);
    const pickupLocation = requestBody.PickupLocation;

    const unicorn = findUnicorn(pickupLocation);

    try {
        await recordRide(rideId, username, unicorn);
        return {
            statusCode: 201,
            body: JSON.stringify({
                RideId: rideId,
                Unicorn: unicorn,
                Eta: '30 seconds',
                Rider: username,
            }),
            headers: {
                'Access-Control-Allow-Origin': '*',
            },
        };
    } catch (err) {
        console.error(err);
        return errorResponse(err.message, context.awsRequestId);
    }
};

function findUnicorn(pickupLocation) {
    console.log(
        'Finding unicorn for ',
        pickupLocation.Latitude,
        ', ',
        pickupLocation.Longitude
    );
    return fleet[Math.floor(Math.random() * fleet.length)];
}

async function recordRide(rideId, username, unicorn) {
    const params = {
        TableName: 'Rides',
        Item: {
            RideId: rideId,
            User: username,
            Unicorn: unicorn,
            RequestTime: new Date().toISOString(),
        },
    };
    await ddb.send(new PutCommand(params));
}

function toUrlString(buffer) {
    return buffer
        .toString('base64')
        .replace(/\+/g, '-')
        .replace(/\//g, '_')
        .replace(/=/g, '');
}

function errorResponse(errorMessage, awsRequestId) {
    return {
        statusCode: 500,
        body: JSON.stringify({
            Error: errorMessage,
            Reference: awsRequestId,
        }),
        headers: {
            'Access-Control-Allow-Origin': '*',
        },
    };
}
```

---

## The Lambda Function Test Event

The following test event was used to validate the Lambda function:

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

---

## Final Outcome

A fully functional **serverless ride-request web application** demonstrating:

* Secure user authentication with Amazon Cognito
* API-driven backend using API Gateway and Lambda
* Scalable NoSQL data storage with DynamoDB
* Continuous deployment using AWS Amplify and GitHub
* Proper IAM-based access control

---
