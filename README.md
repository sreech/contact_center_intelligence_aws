# Contact Center Intelligence with AWS Connect, LEX, Bedrock and Kendra - Deployed through Cloud Formation Template (Stack)

## Create Github Account:
Create Github Account and then under settings under your profile, developer settings->PAT->create one
github_accountname: sreeibm

## Github PAT:
ghp_xyzxxxxxxxx


## ********Install Ubuntu from wsl**********
From wsl through cygwin or through powershell
$ wsl -d Ubuntu-22.04
/mnt/c/cygwin64/home/SreeramChintalapudi

## Install Python:
sudo apt update
sudo apt install python3-pip

##  From Ubuntu install cloud formation template Pkgs:
sudo apt install ruby
sudo gem install cfn-nag

## AWS CLI INSTALL:
sudo snap install aws-cli --classic
sudo apt install awscli

## Configure AWS User Profile:
aws configure  (now add access keys, region)

## aws access keys:
ADXXXXXX / 9gD97xxxxxx

region ca-central-1


## Kommunicate 3rdParty Key:
ADXXXXXX / 9gD97xxxxxx

## First Create Bucket:
awsconnectsree1

## Update the below Environment Variables:
```
export GITHUB_PAT=ghp_xxxxxxx# GitHub PAT copied from Pre-Deployment
export STACK_NAME=lexllmdemo # Stack name must be lower case for S3 bucket naming convention
export KENDRA_WEBCRAWLER_URL=https://www.valuebanktexas.com/ # Public or internal HTTPS website for Kendra to index via Web Crawler (e.g., https://www.investopedia.com/) - Please see https://docs.aws.amazon.com/kendra/latest/dg/data-source-web-crawler.html

export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
export S3_ARTIFACT_BUCKET_NAME=awsconnectsree1
export DATA_LOADER_S3_KEY="agent/lambda/data-loader/loader_deployment_package.zip"
export LAMBDA_HANDLER_S3_KEY="agent/lambda/agent-handler/agent_deployment_package.zip"
export LEX_BOT_S3_KEY="agent/bot/lex.zip"

export BEDROCK_LANGCHAIN_LAYER_ARN=$(aws lambda publish-layer-version --layer-name bedrock-langchain-pypdf --description "Bedrock LangChain PyPDF layer"  --license-info "MIT" --content S3Bucket=${S3_ARTIFACT_BUCKET_NAME},S3Key=agent/lambda-layers/bedrock-langchain-pdfrw.zip --compatible-runtimes python3.11 --query LayerVersionArn --output text)

export GITHUB_TOKEN_SECRET_NAME=$(aws secretsmanager create-secret --name $STACK_NAME-git-pat --secret-string $GITHUB_PAT --query Name --output text)
```
## Execution Starts:
```
aws s3 cp ../agent/ s3://${S3_ARTIFACT_BUCKET_NAME}/agent/ --recursive --exclude ".DS_Store"

aws cloudformation create-stack --stack-name ${STACK_NAME} --template-body file://../cfn/GenAI-FSI-Agent.yml --parameters ParameterKey=S3ArtifactBucket,ParameterValue=${S3_ARTIFACT_BUCKET_NAME} ParameterKey=DataLoaderS3Key,ParameterValue=${DATA_LOADER_S3_KEY} ParameterKey=LambdaHandlerS3Key,ParameterValue=${LAMBDA_HANDLER_S3_KEY} ParameterKey=LexBotS3Key,ParameterValue=${LEX_BOT_S3_KEY} ParameterKey=GitHubTokenSecretName,ParameterValue=${GITHUB_TOKEN_SECRET_NAME} ParameterKey=KendraWebCrawlerUrl,ParameterValue=${KENDRA_WEBCRAWLER_URL} ParameterKey=BedrockLangChainPyPDFLayerArn,ParameterValue=${BEDROCK_LANGCHAIN_LAYER_ARN} ParameterKey=AmplifyRepository,ParameterValue=${AMPLIFY_REPOSITORY} --capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name $STACK_NAME --query "Stacks[0].StackStatus"
aws cloudformation wait stack-create-complete --stack-name $STACK_NAME

export LEX_BOT_ID=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`LexBotID`].OutputValue' --output text)

export LAMBDA_ARN=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`LambdaARN`].OutputValue' --output text)

aws lexv2-models update-bot-alias --bot-alias-id 'TSTALIASID' --bot-alias-name 'TestBotAlias' --bot-id $LEX_BOT_ID --bot-version 'DRAFT' --bot-alias-locale-settings "{\"en_US\":{\"enabled\":true,\"codeHookSpecification\":{\"lambdaCodeHook\":{\"codeHookInterfaceVersion\":\"1.0\",\"lambdaARN\":\"${LAMBDA_ARN}\"}}}}"


aws lexv2-models build-bot-locale --bot-id $LEX_BOT_ID --bot-version "DRAFT" --locale-id "en_US"

export KENDRA_INDEX_ID=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`KendraIndexID`].OutputValue' --output text)

export KENDRA_S3_DATA_SOURCE_ID=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`KendraS3DataSourceID`].OutputValue' --output text)

export KENDRA_WEBCRAWLER_DATA_SOURCE_ID=$(aws cloudformation describe-stacks --stack-name $STACK_NAME --query 'Stacks[0].Outputs[?OutputKey==`KendraWebCrawlerDataSourceID`].OutputValue' --output text)

aws kendra start-data-source-sync-job --id $KENDRA_S3_DATA_SOURCE_ID --index-id $KENDRA_INDEX_ID

aws kendra start-data-source-sync-job --id $KENDRA_WEBCRAWLER_DATA_SOURCE_ID --index-id $KENDRA_INDEX_ID
```

## Insert into DynamoDB through PartiQL 
```
Insert into "lexllmdemo-UserExistingAccounts" value { 'userName': 'Demo User', 'planName': 'Mortgage', 'amountDue': 3325, 'dueDate': '2023-09-01',    'loanAmount': 640000, 'loanDuration': 30, 'loanInterest': 5.735, 'unpaidPrincipal': 250000, 'pin': 1234, 'prefix': 'Mr', 'planId': 'd7edc887-f09f'}
```
