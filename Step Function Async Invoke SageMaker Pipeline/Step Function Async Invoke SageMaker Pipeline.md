# Step Function Async Invoke SageMaker Pipeline

We can use Step Functions to orchestrate the execution of SageMaker pipeline. 

In the step function, we can use a lambda function to start a SageMaker pipeline execution. It passes a task token into the Lambda Function, which subsequently passed to the pipeline execution as a pipeline parameter. 

In last step of the SageMaker pipeline, create a LambdaStep to callback to Step Functions. https://docs.aws.amazon.com/sagemaker/latest/dg/build-and-manage-steps.html#step-type-callback  

![image-20220720213000125](https://raw.githubusercontent.com/qinjie/picgo-images/main/image-20220720213000125.png)

## Step Functions

Definition

```json
{
  "Comment": "An example of the Amazon States Language for starting a task and waiting for a callback.",
  "StartAt": "Invoke Lambda And Wait For Callback",
  "States": {
    "Invoke Lambda And Wait For Callback": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke.waitForTaskToken",
      "Parameters": {
        "FunctionName": "test-execute-sagemaker-transform-pipleline",
        "Payload": {
            "message": "Hello World",
            "comment.$": "$.Comment",
            "TaskToken.$": "$$.Task.Token"
         }
      },
      "ResultPath": "$.lambdaResult",
      "Next": "Succeed",
      "Catch": [
      {
        "ErrorEquals": [ "States.ALL" ],
        "Next": "Fail"
      }
      ]
    },
    "Fail": {
      "Type": "Fail"
    },
    "Succeed": {
      "Type": "Succeed"
    }
  }
}
```

IAM Role Permissions

* Add [AWSLambda_FullAccess](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AWSLambda_FullAccess)



## Lambda Function

Name `test-execute-sagemaker-transform-pipleline`

```python
import boto3
import json

def lambda_handler(event, context):
    
    print(event)
    task_token = event['TaskToken']

    pipeline_name = 'TestPipelineCallbackStepFunctions' 
    pipeline_parameters = [
           { 
              "Name": "TaskToken",
              "Value": task_token
           }
       ]   

    client = boto3.client('sagemaker')
    execution = client.start_pipeline_execution(
        PipelineName=pipeline_name, 
        PipelineExecutionDescription = "MyExecution",
        PipelineExecutionDisplayName = "MyExecution",
        PipelineParameters = pipeline_parameters
    )
    executionArn = execution['PipelineExecutionArn']
    

    # For testing without sagemaker pipeline     
    # client = boto3.client('lambda')
    # response = client.invoke_async(
    #     FunctionName='test-callback-from-sagemaker-pipleline',
    #     InvokeArgs=json.dumps(dict(taskToken=task_token))
    # )

    return {
        "LambdaMessage": f"Executing SageMaker pipeline"
    }
```

IAM Role Permissions

* Add [AmazonSageMakerFullAccess](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/AmazonSageMakerFullAccess)



## Lambda Function

Name `test-callback-from-sagemaker-pipleline`

```python
import boto3

def lambda_handler(event, context):
    
    print(event)
    task_token = event['TaskToken']

    step_functions = boto3.client('stepfunctions')
    step_functions.send_task_success(
        taskToken=task_token,
        output= '{"output": "MyOutput"}'
    )

    return {
        "Message": "Call back to Step Functions"
    }
```

IAM Role Permissions
* Add [AWSLambdaBasicExecutionRole](https://us-east-1.console.aws.amazon.com/iam/home#/policies/arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole)



## SageMaker Pipleine

Upload `Create Pipeline with LambdaStep.ipynb` to SageMaker notebook instance. Run it to create the pipeline.



IAM Role Permissions

* Add AmazonSageMakerPipelinesIntegrations permission to SageMaker Pipeline Role
