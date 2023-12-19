# bloblang-lambda

Run Bloblang mappings in AWS Lambda.

## What is Bloblang?

Bloblang (or blobl for short!) is a language designed specifically for performing data mapping operations. It is primarily distributed as a built-in processor for the data streaming service [Benthos](https://www.benthos.dev/), but can also be used as a standalone tool.

If you are just getting started with Bloblang, the [walkthrough is a good place to begin](https://www.benthos.dev/docs/guides/bloblang/walkthrough/)!

## Why is Bloblang on Lambda useful?

Bloblang is a powerful and fast way to transform data. When deployed to a Lambda function, you can run Bloblang anywhere you can run Lambda, including EventBridge pipes, Step Functions workflows, Kinesis streams, and more! This gives you a declarative data transformation language to use wherever you need to map data from one schema to another schema as it moves through your serverless application.

## Install

You will need to install the [Benthos Docker image](https://www.benthos.dev/docs/guides/getting_started/#docker) if you want to develop or test your Bloblang mapping locally:

```sh
docker pull jeffail/benthos:latest
```

## Develop

The [`config.yaml`](./config.yaml) file contains an example Bloblang mapping function that can be deployed to AWS Lambda. You can edit this file to create your own Bloblang mapping.

You can also run the [Bloblang editor](https://play.benthos.dev/) locally to test your mapping:

```sh
docker run -p 4195:4195 --rm jeffail/benthos blobl server --no-open --host 0.0.0.0
```

![Bloblang editor](https://github.com/lukehedger/bloblang-lambda/assets/1913316/e46274ac-329c-4e33-ae38-bdd3de735e34)

## Test

You can test your Bloblang mapping using the Benthos Docker image:

```sh
docker run -v ./test/config.yaml:/benthos.yaml -v ./test/fixtures/input.json:/input.json jeffail/benthos
```

## Deploy

Deploy your Bloblang mapping function to AWS Lambda using the [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/lambda/index.html#cli-aws-lambda):

```sh
LAMBDA_ENV=`cat config.yaml | jq -csR {Variables:{BENTHOS_CONFIG:.}}`
aws lambda create-function \
  --architectures arm64 \
  --environment "$LAMBDA_ENV" \
  --function-name bloblang-mapping \
  --handler not.used.for.provided.al2.runtime \
  --role YOUR_IAM_ROLE_ARN \
  --runtime provided.al2 \
  --zip-file fileb://benthos/benthos-lambda-al2_4.24.0_linux_arm64.zip
```

If you don't have an IAM role for your function already, you can create one using the [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/iam/index.html#cli-aws-iam):

```sh
aws iam create-role \
  --role-name lambda-bloblang-mapping \
  --assume-role-policy-document file://iam/lambda-trust-policy.json
# copy the ARN from the response and replace YOUR_IAM_ROLE_ARN above
```

> Coming soon: `BloblangFunction` CDK construct

## Invoke

Invoke your Bloblang mapping function using the [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/lambda/index.html#cli-aws-lambda):

```sh
aws lambda invoke \
  --function-name bloblang-mapping \
  --payload file://test/fixtures/input.json \
  --cli-binary-format raw-in-base64-out \
  response.json
```
