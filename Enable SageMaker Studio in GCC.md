# Enable SageMaker Studio in GCC

On GCC, we cannot setup SageMaker studio directly because SageMaker cannot create new roles without setting its Permission Boundary.

We have to manually create IAM Roles before setup SageMaker Studio. 



### Manual Creation of Roles

1. Create an IAM role [AmazonSageMakerServiceCatalogProductsLaunchRole](https://console.aws.amazon.com/iam/home?#/roles/AmazonSageMakerServiceCatalogProductsLaunchRole) 

   * This role passes an AmazonSageMakerServiceCatalogProductsUseRole role to the provisioned AWS Service Catalog product resources.
   * Set proper PermissionBoundary.
   * After role creation, add an inline policy `AmazonSageMakerAdmin-ServiceCatalogProductsServiceRolePolicy` with following JSON. Reference https://docs.aws.amazon.com/sagemaker/latest/dg/security-iam-awsmanpol-sc.html
   * Note: You can note create following IAM policy directly because you will hit the limit of `6144` characters for a managed policy. 

   ```
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "apigateway:GET",
           "apigateway:POST",
           "apigateway:PUT",
           "apigateway:PATCH",
           "apigateway:DELETE"
         ],
         "Resource": "*",
         "Condition": {
           "StringLike": {
             "aws:ResourceTag/sagemaker:launch-source": "*"
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": [
           "apigateway:POST"
         ],
         "Resource": "*",
         "Condition": {
           "ForAnyValue:StringLike": {
             "aws:TagKeys": [
               "sagemaker:launch-source"
             ]
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": [
           "apigateway:PATCH"
         ],
         "Resource": [
           "arn:aws:apigateway:*::/account"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "cloudformation:CreateStack",
           "cloudformation:UpdateStack",
           "cloudformation:DeleteStack"
         ],
         "Resource": "arn:aws:cloudformation:*:*:stack/SC-*",
         "Condition": {
           "ArnLikeIfExists": {
             "cloudformation:RoleArn": [
               "arn:aws:sts::*:assumed-role/AmazonSageMakerServiceCatalog*"
             ]
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": [
           "cloudformation:DescribeStackEvents",
           "cloudformation:DescribeStacks"
         ],
         "Resource": "arn:aws:cloudformation:*:*:stack/SC-*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "cloudformation:GetTemplateSummary",
           "cloudformation:ValidateTemplate"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "codebuild:CreateProject",
           "codebuild:DeleteProject",
           "codebuild:UpdateProject"
         ],
         "Resource": [
           "arn:aws:codebuild:*:*:project/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "codecommit:CreateCommit",
           "codecommit:CreateRepository",
           "codecommit:DeleteRepository",
           "codecommit:GetRepository",
           "codecommit:TagResource"
         ],
         "Resource": [
           "arn:aws:codecommit:*:*:sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "codecommit:ListRepositories"
         ],
         "Resource": "*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "codepipeline:CreatePipeline",
           "codepipeline:DeletePipeline",
           "codepipeline:GetPipeline",
           "codepipeline:GetPipelineState",
           "codepipeline:StartPipelineExecution",
           "codepipeline:TagResource",
           "codepipeline:UpdatePipeline"
         ],
         "Resource": [
           "arn:aws:codepipeline:*:*:sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "cognito-idp:CreateUserPool",
           "cognito-idp:TagResource"
         ],
         "Resource": "*",
         "Condition": {
           "ForAnyValue:StringLike": {
             "aws:TagKeys": [
               "sagemaker:launch-source"
             ]
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": [
           "cognito-idp:CreateGroup",
           "cognito-idp:CreateUserPoolDomain",
           "cognito-idp:CreateUserPoolClient",
           "cognito-idp:DeleteGroup",
           "cognito-idp:DeleteUserPool",
           "cognito-idp:DeleteUserPoolClient",
           "cognito-idp:DeleteUserPoolDomain",
           "cognito-idp:DescribeUserPool",
           "cognito-idp:DescribeUserPoolClient",
           "cognito-idp:UpdateUserPool",
           "cognito-idp:UpdateUserPoolClient"
         ],
         "Resource": "*",
         "Condition": {
           "StringLike": {
             "aws:ResourceTag/sagemaker:launch-source": "*"
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": [
           "ecr:CreateRepository",
           "ecr:DeleteRepository",
           "ecr:TagResource"
         ],
         "Resource": [
           "arn:aws:ecr:*:*:repository/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "events:DescribeRule",
           "events:DeleteRule",
           "events:DisableRule",
           "events:EnableRule",
           "events:PutRule",
           "events:PutTargets",
           "events:RemoveTargets"
         ],
         "Resource": [
           "arn:aws:events:*:*:rule/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "firehose:CreateDeliveryStream",
           "firehose:DeleteDeliveryStream",
           "firehose:DescribeDeliveryStream",
           "firehose:StartDeliveryStreamEncryption",
           "firehose:StopDeliveryStreamEncryption",
           "firehose:UpdateDestination"
         ],
         "Resource": "arn:aws:firehose:*:*:deliverystream/sagemaker-*"
       },
       {
           "Effect": "Allow",
         "Action": [
           "glue:CreateDatabase",
           "glue:DeleteDatabase"
         ],
         "Resource": [
           "arn:aws:glue:*:*:catalog",
           "arn:aws:glue:*:*:database/sagemaker-*",
           "arn:aws:glue:*:*:table/sagemaker-*",
           "arn:aws:glue:*:*:userDefinedFunction/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "glue:CreateClassifier",
           "glue:DeleteClassifier",
           "glue:DeleteCrawler",
           "glue:DeleteJob",
           "glue:DeleteTrigger",
           "glue:DeleteWorkflow",
           "glue:StopCrawler"
         ],
         "Resource": [
           "*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "glue:CreateWorkflow"
         ],
         "Resource": [
           "arn:aws:glue:*:*:workflow/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "glue:CreateJob"
         ],
         "Resource": [
           "arn:aws:glue:*:*:job/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "glue:CreateCrawler",
           "glue:GetCrawler"
         ],
         "Resource": [
           "arn:aws:glue:*:*:crawler/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "glue:CreateTrigger",
           "glue:GetTrigger"
         ],
         "Resource": [
           "arn:aws:glue:*:*:trigger/sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "iam:PassRole"
         ],
         "Resource": [
           "arn:aws:iam::*:role/service-role/AmazonSageMakerServiceCatalog*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "lambda:AddPermission",
           "lambda:CreateFunction",
           "lambda:DeleteFunction",
           "lambda:GetFunction",
           "lambda:GetFunctionConfiguration",
           "lambda:InvokeFunction",
           "lambda:RemovePermission"
         ],
         "Resource": [
           "arn:aws:lambda:*:*:function:sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "logs:CreateLogGroup",
           "logs:CreateLogStream",
           "logs:DeleteLogGroup",
           "logs:DeleteLogStream",
           "logs:DescribeLogGroups",
           "logs:DescribeLogStreams",
           "logs:PutRetentionPolicy"
         ],
         "Resource": [
           "arn:aws:logs:*:*:log-group:/aws/apigateway/AccessLogs/*",
           "arn:aws:logs:*:*:log-group::log-stream:*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": "s3:GetObject",
         "Resource": "*",
         "Condition": {
           "StringEquals": {
             "s3:ExistingObjectTag/servicecatalog:provisioning": "true"
           }
         }
       },
       {
         "Effect": "Allow",
         "Action": "s3:GetObject",
         "Resource": [
           "arn:aws:s3:::sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "s3:CreateBucket",
           "s3:DeleteBucket",
           "s3:DeleteBucketPolicy",
           "s3:GetBucketPolicy",
           "s3:PutBucketAcl",
           "s3:PutBucketNotification",
           "s3:PutBucketPolicy",
           "s3:PutBucketPublicAccessBlock",
           "s3:PutBucketLogging",
           "s3:PutEncryptionConfiguration",
           "s3:PutBucketTagging",
           "s3:PutObjectTagging",
           "s3:PutBucketCORS"
         ],
         "Resource": "arn:aws:s3:::sagemaker-*"
       },
       {
         "Effect": "Allow",
         "Action": [
           "sagemaker:CreateEndpoint",
           "sagemaker:CreateEndpointConfig",
           "sagemaker:CreateModel",
           "sagemaker:CreateWorkteam",
           "sagemaker:DeleteEndpoint",
           "sagemaker:DeleteEndpointConfig",
           "sagemaker:DeleteModel",
           "sagemaker:DeleteWorkteam",
           "sagemaker:DescribeModel",
           "sagemaker:DescribeEndpointConfig",
           "sagemaker:DescribeEndpoint",
           "sagemaker:DescribeWorkteam",
           "sagemaker:CreateCodeRepository",
           "sagemaker:DescribeCodeRepository",
           "sagemaker:UpdateCodeRepository",
           "sagemaker:DeleteCodeRepository"
         ],
         "Resource": [
           "arn:aws:sagemaker:*:*:*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "sagemaker:CreateImage",
           "sagemaker:DeleteImage",
           "sagemaker:DescribeImage",
           "sagemaker:UpdateImage",
           "sagemaker:ListTags"
         ],
         "Resource": [
           "arn:aws:sagemaker:*:*:image/*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": [
           "states:CreateStateMachine",
           "states:DeleteStateMachine",
           "states:UpdateStateMachine"
         ],
         "Resource": [
           "arn:aws:states:*:*:stateMachine:sagemaker-*"
         ]
       },
       {
         "Effect": "Allow",
         "Action": "codestar-connections:PassConnection",
         "Resource": "arn:aws:codestar-connections:*:*:connection/*",
         "Condition": {
           "StringEquals": {
             "codestar-connections:PassedToService": "codepipeline.amazonaws.com"
           }
         }
       }
     ]
   }
   ```

2. Create an IAM role [AmazonSageMakerServiceCatalogProductsUseRole](https://console.aws.amazon.com/iam/home?#/roles/AmazonSageMakerServiceCatalogProductsUseRole) for service SageMaker.

   * Set proper PermissionBoundary
   * Attach the policy `AmazonSageMakerAdmin-ServiceCatalogProductsServiceRolePolicy`.



### Setup SageMaker Studio

* Choose "Standard Setup"
* Use IAM role AmazonSageMakerServiceCatalogProductsUseRole
* Select VPC and subnet

