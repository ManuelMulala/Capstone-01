# AWS Capstone: Containerized Web App + Serverless API + Observability

## Architecture
- **Web app:** Nginx container image stored in Amazon ECR and run on ECS Fargate, deployed with CloudFormation (`infrastructure.yml`)
- **API:** Amazon API Gateway (REST) `POST /submit` integrated with an AWS Lambda function (Node.js)
- **Observability:** Lambda logs in CloudWatch Logs and a CloudWatch dashboard showing Lambda Invocations and Errors

Region: `eu-west-1` (Ireland)

## Repository structure
```
infrastructure.yml   # CloudFormation: ECS cluster, task definition, Fargate service
lambda-api/          # Lambda function code (index.js)
webapp/              # Dockerfile and index.html
screenshots/         # Evidence
```

---

# Required screenshots

## 1. Live ECS web application
The Fargate task serves the Nginx page on its public IP.

![ECS web app](screenshots/ecs-app.png)

## 2. Successful API Gateway test
```
curl -i -X POST https://wc3q93d3yf.execute-api.eu-west-1.amazonaws.com/prod/submit \
  -H "Content-Type: application/json" \
  -d '{"name":"Test User","message":"Hello from curl"}'
```
The API returned HTTP 200 with a success message and a generated id.

![API test](screenshots/api-test.png)

## 3. CloudWatch dashboard
`CapstoneDashboard` graphs Lambda Invocations and Errors for `capstone-submit`.

![CloudWatch dashboard](screenshots/dashboard.png)

---

# Build walkthrough

## Step 1: Serverless API

### Lambda function
Created `capstone-submit` (Node.js). It logs the request body with `console.log` and returns a success message with a random id.

![Lambda function list](screenshots/lambda-01-function-list.png)
![Lambda code deployed](screenshots/lambda-02-code-deployed.png)
![Lambda test succeeded](screenshots/lambda-03-test-success.png)

### API Gateway
Created a REST API with a `/submit` resource and a `POST` method using Lambda proxy integration, then deployed it to stage `prod`.

![API created](screenshots/apigw-01-api-created.png)
![/submit resource](screenshots/apigw-02-submit-resource.png)
![POST method](screenshots/apigw-03-post-method.png)

## Step 2: Containerized web app

### Docker build and local test
Built the image for `linux/amd64` and tested it locally on port 8080.

```
docker build --platform linux/amd64 -t capstone-webapp .
docker run --rm -p 8080:80 capstone-webapp
```

![Docker build](screenshots/docker-01-build.png)
![Local test](screenshots/docker-02-local-test.png)

### Amazon ECR
Pushed the image to a private ECR repository.

![ECR repository](screenshots/ecr-01-repository.png)
![ECR image](screenshots/ecr-02-image.png)

### CloudFormation deployment
Deployed the ECS cluster, task definition, security group and Fargate service from `infrastructure.yml`:

```
aws cloudformation deploy --template-file infrastructure.yml --stack-name capstone-ecs \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides ImageUri=<ecr-image-uri> VpcId=<vpc-id> SubnetIds=<subnet-ids>
```

![Stack complete](screenshots/cfn-01-stack-complete.png)
![Stack events](screenshots/cfn-02-events.png)
![Stack resources](screenshots/cfn-03-resources.png)

### ECS service and running task
![ECS service](screenshots/ecs-01-service.png)
![Task running](screenshots/ecs-02-task-running.png)

## Step 3: Observability with CloudWatch
Sent 20 test requests to the API and refreshed the web app, then checked the Lambda logs and built the dashboard.

![Lambda logs in CloudWatch](screenshots/cw-01-lambda-logs.png)
![Dashboard list](screenshots/cw-02-dashboard-list.png)

---

## Cleanup
```
aws cloudformation delete-stack --stack-name capstone-ecs
aws ecr delete-repository --repository-name capstone-webapp --force --region eu-west-1
```
Then delete the API Gateway API, the Lambda function, the log group and the dashboard.
