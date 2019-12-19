# SignalFx Node Lambda Wrapper

## Overview

You can use this document to add a SignalFx wrapper to your AWS Lambda for Node. 

The SignalFx Node Lambda Wrapper wraps around an AWS Lambda Node function handler, which allows metrics and traces to be sent to SignalFx.

At a high-level, to add a SignalFx Node Lambda wrapper, you can package the code yourself, or you can use a Lambda layer containing the wrapper and then attach the layer to a Lambda function.

To learn more about Lambda Layers, please visit the AWS documentation site and see [AWS Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html).

## Step 1: Add the Lambda wrapper in AWS

To add the SignalFx wrapper, you have the following options:
   
   * Option 1: In AWS, create a Lambda function, then attach a SignalFx-hosted layer with a wrapper.
      * If you are already using Lambda layers, then SignalFx recommends that you follow this option. 
      * In this option, you will use a Lambda layer created and hosted by SignalFx.
   * Option 2: In AWS, create a Lambda function, then create and attach a layer based on a SignalFx SAM (Serverless Application Model) template.
      * If you are already using Lambda layers, then SignalFx also recommends that you follow this option. 
      * In this option, you will choose a SignalFx template, and then deploy a copy of the layer.
   * Option 3: Use the wrapper as a regular dependency, and then create a Lambda function based on your artifact containing both code and dependencies.   
   
### Option 1: Create a Lambda function, then attach the SignalFx-hosted Lambda layer

In this option, you will use a Lambda layer created and hosted by SignalFx.

1. To verify compatibility, review the list of supported regions. See [Lambda Layer Versions](https://github.com/signalfx/lambda-layer-versions/blob/master/python/PYTHON.md).
2. Open your AWS console. 
3. In the landing page, under **Compute**, click **Lambda**.
4. Click **Create function** to create a layer with SignalFx's capabilities.
5. Click **Author from scratch**.
6. In **Function name**, enter a descriptive name for the wrapper. 
7. In **Runtime**, select the desired language.
8. Click **Create function**. 
9. Click on **Layers**, then add a layer.
10. Mark **Provide a layer version**.
11. Enter an ARN number. 
   * To locate the ARN number, see [Lambda Layer Versions](https://github.com/signalfx/lambda-layer-versions/blob/master/python/PYTHON.md).

### Option 2: Create a Lambda function, then create and attach a layer based on a SignalFx template

In this option, you will choose a SignalFx template, and then deploy a copy of the layer.

1. Open your AWS console. 
2. In the landing page, under **Compute**, click **Lambda**.
3. Click **Create function** to create a layer with SignalFx's capabilities.
4. Click **Browse serverless app repository**.
5. Click **Public applications**.
6. In the search field, enter and select **signalfx-lambda-python-wrapper**.
7. Review the template, permissions, licenses, and then click **Deploy**.
    * A copy of the layer will now be deployed into your account.
8. Return to the previous screen to add a layer to the function, select from list of runtime compatible layers, and then select the name of the copy. 

### Option 3: Install the wrapper package 

1. Use the hosted package:

```
{
  "name": "my-module",
  "dependencies": {
    "signalfx-lambda": "^0.0.13"
  }
}
```

## Step 2: Locate and set the ingest endpoint

By default, this function wrapper will send data to the us0 realm. As a result, if you are not in us0 realm and you want to use the ingest endpoint directly, then you must explicitly set your realm. To set your realm, use a subdomain, such as ingest.us1.signalfx.com or ingest.eu0.signalfx.com.

To locate your realm:

1. Open SignalFx and in the top, right corner, click your profile icon.
2. Click **My Profile**.
3. Next to **Organizations**, review the listed realm.


## Step 3: Set environment variables

1. Set SIGNALFX_ACCESS_TOKEN with your correct access token. Review the following example. 

.. code:: bash

    SIGNALFX_ACCESS_TOKEN=access token


2. If you use POPS, Smart Gateway, or want to ingest directly from a realm other than us0, then you must set at least one endpoint variable. (For environment variables, SignalFx defaults to the us0 realm. As a result, if you are not in the us0 realm, you may need to set your environment variables.) You can update one of the variables below. Review the following examples.  

.. code:: bash

    SIGNALFX_ENDPOINT_URL=http://<my_gateway>:8080
    SIGNALFX_METRICS_URL=ingest endpoint [ default: https://pops.signalfx.com ]
    
To learn more, see: 
  * [SignalFx Point of Presence Service (POPS)](https://docs.signalfx.com/en/latest/integrations/integrations-reference/integrations.signalfx.point.of.presence.service.(pops).html)
  * [Deploying the SignalFx Smart Gateway](https://docs.signalfx.com/en/latest/apm/apm-deployment/smart-gateway.html)
        
    
3. (Optional) Set additional environment variable. Review the following examples.  

.. code:: bash

    SIGNALFX_SEND_TIMEOUT=timeout in seconds for sending datapoint [ default: 0.3 ]
    SIGNALFX_TRACING_URL=tracing endpoint [ default: https://ingest.signalfx.com/v1/trace ]
    
## Step 4: Wrap a function

1. Wrap your function handler:

```
'use strict';

const signalFxLambda = require('signalfx-lambda');

exports.handler = signalFxLambda.wrapper((event, context, callback) => {
  ...
});
```

2. Use async/await:

```
'use strict';

const signalFxLambda = require('signalfx-lambda');

exports.handler = signalFxLambda.asyncWrapper(async (event, context) => {
  ...
});
```

## Step 5: Deploy the function 

1. Run `npm pack` to package the module with the configuration in `package.json`.

## Step 6: Test the function 

1. Install node-lambda via `npm install -g node-lambda` (globally) or `npm install node-lambda` (locally).

## Step 7: Test locally 

1. Create deploy.env to submit data to SignalFx, containing the required and optional environment variables mentioned above:

2. Run `node-lambda run -f deploy.env`.

## Step 8: Test from AWS 

1. Run `node-lambda deploy -f deploy.env` to deploy to AWS. It will use any environment variables configured in `.env`. Example:

```
AWS_ENVIRONMENT=
AWS_PROFILE=
AWS_SESSION_TOKEN=
AWS_HANDLER=index.handler
AWS_MEMORY_SIZE=128
AWS_TIMEOUT=3
AWS_DESCRIPTION=
AWS_RUNTIME=nodejs6.10 # Use nodejs8.10 when using async/await
AWS_VPC_SUBNETS=
AWS_VPC_SECURITY_GROUPS=
AWS_TRACING_CONFIG=
AWS_REGION=us-east-2
AWS_FUNCTION_NAME=my-function
AWS_ROLE_ARN=arn:aws:iam::someAccountId:role/someRole
PACKAGE_DIRECTORY=build
```

## Additional information 

### Metrics sent by the metrics wrapper

The Lambda wrapper sends the following metrics to SignalFx:

| Metric Name  | Type | Description |
| ------------- | ------------- | ---|
| function.invocations  | Counter  | Count number of Lambda invocations|
| function.cold_starts  | Counter  | Count number of cold starts|
| function.errors  | Counter  | Count number of errors from underlying Lambda handler|
| function.duration  | Gauge  | Milliseconds in execution time of underlying Lambda handler|

The Lambda wrapper adds the following dimensions to all data points sent to SignalFx:

| Dimension | Description |
| ------------- | ---|
| lambda_arn  | ARN of the Lambda function instance |
| aws_region  | AWS Region  |
| aws_account_id | AWS Account ID  |
| aws_function_name  | AWS Function Name |
| aws_function_version  | AWS Function Version |
| aws_function_qualifier  | AWS Function Version Qualifier (version or version alias if it is not an event source mapping Lambda invocation) |
| event_source_mappings  | AWS Function Name (if it is an event source mapping Lambda invocation) |
| aws_execution_env  | AWS execution environment (e.g. AWS_Lambda_nodejs6.10) |
| function_wrapper_version  | SignalFx function wrapper qualifier (e.g. signalfx-lambda-0.0.9) |
| metric_source | The literal value of 'lambda_wrapper' |


### License

Apache Software License v2. Copyright © 2014-2017 SignalFx
