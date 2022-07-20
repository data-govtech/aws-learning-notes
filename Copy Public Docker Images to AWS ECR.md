# Copy Public Docker Images to AWS ECR

Tags: docker-image, docker-hug, pull-rate-limit, aws-cdk, aws-lambda, aws-ecr

## Problem

Docker Hub is starting to [introduce rate limits for anonymous users](https://docs.docker.com/docker-hub/download-rate-limit/). You docker image building in AWS Codepipeline may failed with following message if the build is too frequent.

```
You have reached your pull rate limit.
You may increase the limit by authenticating and upgrading.
```

## Solution

If you have access to AWS CLI for your AWS account, you may follow these steps to copy images to your AWS ECR. (Credit to [Aurore Malherbes](https://www.padok.fr/en/blog/docker-hub-rate-limit))

- Create a new ECR called `node-alpine` through the console or CLI
- Pull locally the node image, the one at the top of your Dockerfile `docker pull node:12-alpine`
- Tag it with your ECR url `docker tag node:12-alpine <AWS_ACCOUNT_ID>dkr.ecr.<AWS_REGION>.amazonaws.com/node-alpine:12`
- Push this image to your ECR: `docker push <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/node-alpine:12`
- Modify the first line of your Dockerfile from `FROM node:12-alpine` to `FROM node:<AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/node-alpine:12`



### Example Scripts for node-alphine and nginx Images

* Pull and tag docker images

```
docker pull node:12.16.1-alpine3.9
docker tag node:12.16.1-alpine3.9 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/node:12.16.1-alpine3.9
docker pull nginx:1.17.8-alpine
docker tag nginx:1.17.8-alpine 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/nginx:1.17.8-alpine
```

* Login to AWS using CLI v1

```
$(aws ecr get-login --no-include-email)
```

* Login to AWS (You need to setup your AWS profile) using AWS CLI v2

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com
```

* Push in images

```
docker push 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/node:12.16.1-alpine3.9
docker push 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/nginx:1.17.8-alpine
```



### Example Scripts for Python Images

* Pull and tag docker images

```
docker pull python:3.9
docker tag python:3.9 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/python:3.9
```

* Login to AWS (You need to setup your AWS profile)

```
aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com
```

* Push in images

```
docker push 305326993135.dkr.ecr.ap-southeast-1.amazonaws.com/python:3.9
```

