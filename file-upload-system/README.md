# Serverless Project : MAYNISM
This is a file upload system with a serverless API using AWS Lambda, S3, and React for the client interface.

## Architecture
![Serverless Architecture Diagram](https://github.com/MayHyeyeonKim/maynism/blob/main/file-upload-system/images/serverless_architecture.png)


## Components
- **Amazon S3 (Simple Storage Service)**: S3 buckets are used to store and retrieve any amount of data at any time, from anywhere on the web.
- **Amazon API Gateway**: Acts as a fully managed gateway for managing, creating, and securing APIs at scale.
- **AWS Lambda**: A serverless compute service that lets you run code without provisioning or managing servers. Lambda executes your code only when needed and scales automatically.
- **IAM Role**: Identity and Access Management (IAM) roles provide Lambda functions with the necessary permissions to access other AWS services like S3.

## Workflow
1. **User Interaction**: Users interact with the application through a frontend built with React. It sends and receives data from the backend via API Gateway.
2. **S3 Bucket**: The frontend uploads files directly to an S3 bucket or retrieves them as needed, providing a durable and secure object storage.
3. **API Gateway**: It routes incoming API requests to various backend services. In this case, it invokes the corresponding Lambda functions.
4. **Lambda Functions**: These functions execute the backend logic in response to API requests. They can manipulate files in S3 buckets, handle business logic, and interact with other AWS services.
5. **S3 Integration**: Lambda functions can retrieve files from or upload files to S3 buckets, depending on the business needs.

## AWS Free Tier Usage
The project is currently operating within the AWS Free Tier limits, which offers a generous amount of resources without any cost for the first 12 months of a new AWS account.

## Gosh!! My Pain in the buxx (Encountered Errors)

![Error-CloudWatch](https://github.com/MayHyeyeonKim/maynism/blob/main/file-upload-system/images/error_log.png)

I spent approximately 6 hours resolving the error shown in the screenshot above. The root cause was a version incompatibility issue with Node.js. The problem was resolved by switching to a lower version of Node.js. A helpful discussion that guided me to the solution can be found on Stack Overflow:

[Stack Overflow Discussion on AWS Lambda and `aws-sdk` Module Error](https://stackoverflow.com/questions/74792293/aws-lambda-cannot-find-module-aws-sdk-in-build-a-basic-web-application-tutoria)


## Lambda Function Handler (node v16)

Below are the Lambda function handlers for Maynism. The handlers listen for incoming events and invoke the appropriate functions based on the HTTP method and the path.


```javascript
// index.cjs
const { processFile } = require('./file-processing-service.cjs');
const { buildResponse } = require('./util.cjs');

const fileUploadPath = '/file-upload';

exports.handler = async function(event) {
  console.log('Request Event: ', event);
  let response;
  switch(true){
    case event.httpMethod === 'POST' && event.path === fileUploadPath:
      response = await processFile(event.body);
      break;
    default:
      response = buildResponse(404);
  }
  return response;
};
```

```javascript
// file-processing-service.cjs
const { buildResponse } = require('./util.cjs');
const AWS = require('aws-sdk');

const s3 = new AWS.S3();

const bucketName = 'YOUR_BUCKET_NAME';
const s3Subfolder = 'YOUR_SUBFOLDER_NAME';

async function process(requestBody) {
    const fileName = requestBody.split('\r\n')[1].split(';')[2].split('=')[1].replace(/^"|"$/g, '').trim();
    let fileContent = requestBody.split('\r\n')[4].trim();
    fileContent += `\n\nProcess Timestamp: ${new Date().toISOString()}`
    const params = {
        Bucket: bucketName,
        Key: `${s3Subfolder}/${fileName}`,
        Body: fileContent
    };
    await s3.putObject(params).promise();
    return buildResponse(200);
}

module.exports = { process };

```

```javascript
// util.cjs
function buildResponse(statusCode, body){
    return {
        statusCode: statusCode,
        headers: {
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(body)
    };
}

module.exports = { buildResponse };

```


