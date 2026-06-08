# Deployment Steps

## Step 1: Clone the Repository

```bash
git clone https://github.com/RemillaSriVaishnavi/Serverless-CICD-Automation.git
cd Serverless-CICD-Automation.git
```

## Step 2: Verify Project Files

Ensure the repository contains the following files:

```bash
index.html
style.css
Dockerfile
buildspec.yml
README.md
```

## Step 3: Build Docker Image Locally

```bash
docker build -t website .
```

## Step 4: Run Docker Container Locally

```bash
docker run -d -p 80:80 website
```

Open browser:

```bash
http://ipaddress
```

## Step 5: Create Amazon ECR Repository

1. Open AWS Console
2. Navigate to Amazon ECR
3. Click "Create Repository"
4. Repository Name: **website**
5. Create repository


## Step 6: Push Docker Image to Amazon ECR

### Authenticate Docker with ECR

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com
```

### Tag Docker Image

```bash
docker tag website:latest xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/website:latest
```

### Push Docker Image

```bash
docker push xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/website:latest
```

## Step 7: Create ECS Cluster

1. Open Amazon ECS
2. Click "Create Cluster"
3. Cluster Name: **Demo-Cluster**
4. Infrastructure:
- AWS Fargate
5. Create cluster


## Step 8: Create Task Definition

1. Open ECS
2. Navigate to Task Definitions
3. Click "Create"

### Configuration

- Launch Type: **AWS Fargate**
- Task Definition Name: **websitetask-def**
- Container Name: **website**
- Image URI: **xxxxxxxxxxxx.dkr.ecr.us-east-1.amazonaws.com/website:latest**
- Container Port: **80**

Save task definition.


## Step 9: Create ECS Service

1. Open ECS Cluster
2. Click "Create Service"

### Service Configuration

- Launch Type: **FARGATE**
- Task Definition: **websitetask-def**
- Service Name: **website**
- Desired Tasks: **1**


## Step 10: Configure Load Balancer

1. Select: **Application Load Balancer**
2. Create new load balancer
3. Listener Port: **80**
4. Create Target Group
5. Deploy service


## Step 11: Create AWS CodeBuild Project

1. Open AWS CodeBuild
2. Click "Create Project"

### Configuration

- Project Name: **website**
- Source Provider: **GitHub**
- Repository:
Connect GitHub repository

- Environment:
```bash
Managed Image
Ubuntu
Docker Enabled
```

- Buildspec:
```bash
Use buildspec.yml
```
Create project.

## Step 12: Create buildspec.yml

Create a file named:
```bash
buildspec.yml
```

Add the following content:

```yaml
version: 0.2

env:
  variables:
    AWS_ACCOUNT_ID: "xxxxxxxxxxxx"
    REPOSITORY_NAME: "website"
    CONTAINER_NAME: "website"
    AWS_DEFAULT_REGION: "us-east-1"

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - IMAGE_TAG=$CODEBUILD_BUILD_NUMBER
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$REPOSITORY_NAME
      - echo IMAGE TAG = $IMAGE_TAG
      - echo REPOSITORY_URI = $REPOSITORY_URI
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $REPOSITORY_URI

  build:
    commands:
      - echo Build started on `date`
      - docker build --no-cache -t $REPOSITORY_NAME:$IMAGE_TAG .
      - docker tag $REPOSITORY_NAME:$IMAGE_TAG $REPOSITORY_URI:$IMAGE_TAG
      - docker tag $REPOSITORY_NAME:$IMAGE_TAG $REPOSITORY_URI:latest

  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing Docker images...
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - docker push $REPOSITORY_URI:latest
      - printf '[{"name":"%s","imageUri":"%s"}]' $CONTAINER_NAME $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
      - cat imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

## Step 13: Create AWS CodePipeline

1. Open AWS CodePipeline
2. Click "Create Pipeline"

### Pipeline Configuration

- Pipeline Name: **DeployToEcsFargateService**
- Source Provider: **GitHub**
- Repository: Select GitHub repository
- Build Provider: **AWS CodeBuild**
- Build Project: **website**
- Deploy Provider: **Amazon ECS**
- Cluster: **Demo-Cluster**
- Service: **website**

Create pipeline.

## Step 14: Push Code to GitHub

```bash
git add .
git commit -m "Initial deployment"
git push origin main
```

## Step 15: Verify Automatic Deployment

After pushing code:

- CodePipeline starts automatically
- CodeBuild builds Docker image
- Docker image pushed to ECR
- ECS service updates automatically
- Application changes reflect automatically


## Step 16: Monitor Logs Using CloudWatch

1. Open CloudWatch
2. Navigate to: **Log Groups**
3. Verify logs for:
- CodeBuild
- ECS Tasks


## Step 17: Configure CloudWatch Alarms

1. Open CloudWatch
2. Create Alarm

### Monitor:
- CPU Usage
- Memory Usage
- ECS Health


## Step 18: Access Deployed Application

Open Load Balancer DNS URL in browser:

```bash
ecselb-1488723080.us-east-1.elb.amazonaws.com
```
Verify website deployment.


## Step 19: Update Website Automatically

Modify any file:

```bash
index.html
style.css
script.js
```

Push changes:

```bash
git add .
git commit -m "Updated website"
git push origin main
```

Pipeline automatically redeploys the latest version.


## Step 20: Final Verification

Verify:
- GitHub webhook trigger
- Successful CodePipeline execution
- Successful CodeBuild execution
- Docker image in ECR
- ECS task running
- CloudWatch logs generated
- Website updated successfully
