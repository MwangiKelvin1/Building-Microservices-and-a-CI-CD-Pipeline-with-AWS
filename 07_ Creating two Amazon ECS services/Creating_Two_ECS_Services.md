# Phase 7: Creating Two Amazon ECS Services

## Project

**Project:** Building Microservices and a CI/CD Pipeline in AWS  
- Task 7.1 — Create the ECS service for the customer microservice
- Task 7.2 — Create the ECS service for the employee microservice

---

## 1. Objective

The objective of Phase 7 is to create two Amazon Elastic Container Service (Amazon ECS) services in the existing ECS cluster. One service represents the customer microservice, and the other represents the employee microservice.

Both services are configured to use:

- AWS Fargate as the launch type
- An Application Load Balancer target group
- Two public subnets
- The `microservices-sg` security group
- One desired task
- The `CODE_DEPLOY` deployment controller for blue/green deployments in Phase 8

At this stage, the services can be created successfully even if no tasks are running yet. The lab guide explains that a `0/1 Task running` state is expected before the task sets are launched through CodeDeploy in Phase 8.

## 2. Architecture and resource details

### Shared ECS configuration

| Setting | Value |
|---|---|
| AWS Region | `us-east-1` — US East (N. Virginia) |
| ECS cluster | `microservices-serverlesscluster` |
| Launch type | `FARGATE` |
| Scheduling strategy | `REPLICA` |
| Desired count | `1` per service |
| Deployment controller | `CODE_DEPLOY` |
| Container port | `8080` |
| Public Subnet 1 | `subnet-090965b31c41c4082` |
| Public Subnet 2 | `subnet-0f9b8811a58feb182` |
| Security group | `sg-02c2807cfa741836c` (`microservices-sg`) |
| Assign public IP | `ENABLED` |

### Customer microservice

| Setting | Value |
|---|---|
| ECS service name | `customer-microservice` |
| Task definition | `customer-microservice:1` |
| Container name | `customer` |
| Target group | `customer-tg-two` |
| Target group ARN | `arn:aws:elasticloadbalancing:us-east-1:825501112776:targetgroup/customer-tg-two/c88708b6ba603fa7` |

### Employee microservice

| Setting | Value |
|---|---|
| ECS service name | `employee-microservice` |
| Task definition | `employee-microservice:1` |
| Container name | `employee` |
| Target group | `employee-tg-two` |
| Target group ARN | `arn:aws:elasticloadbalancing:us-east-1:825501112776:targetgroup/employee-tg-two/627d99bba47f5dcf` |

## 3. Prerequisites

Before starting Phase 7, the following resources should already exist from earlier phases:

- ECS cluster: `microservices-serverlesscluster`
- Task definition: `customer-microservice:1`
- Task definition: `employee-microservice:1`
- Target groups: `customer-tg-one`, `customer-tg-two`, `employee-tg-one`, and `employee-tg-two`
- Application Load Balancer: `microservicesLB`
- Security group: `microservices-sg`
- Two public subnets in the lab VPC
- Cloud9 environment with the `deployment` directory

The commands below use the resource IDs recorded during this lab. If the lab environment is reset or recreated, verify the IDs and task-definition revisions before reusing the commands.

## 4. Open Cloud9 and move to the deployment directory

Run:

```bash
cd ~/environment/deployment
pwd
ls
```

The working directory should be:

```text
/home/ec2-user/environment/deployment
```

---

## 5. Task 7.1 — Create the customer microservice ECS service

### 5.1 Create the customer service configuration

Create a file named `create-customer-microservice-tg-two.json`:

```bash
cat > create-customer-microservice-tg-two.json <<'EOF'
{
  "cluster": "microservices-serverlesscluster",
  "serviceName": "customer-microservice",
  "taskDefinition": "customer-microservice:1",
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:accoundid:targetgroup/customer-tg-two/c88708b6ba603fa7",
      "containerName": "customer",
      "containerPort": 8080
    }
  ],
  "desiredCount": 1,
  "launchType": "FARGATE",
  "schedulingStrategy": "REPLICA",
  "deploymentController": {
    "type": "CODE_DEPLOY"
  },
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-id",
        "subnet-id"
      ],
      "securityGroups": [
        "sg-id"
      ],
      "assignPublicIp": "ENABLED"
    }
  }
}
EOF
```

### 5.2 Validate the JSON

Run:

```bash
python3 -m json.tool create-customer-microservice-tg-two.json
```

If the JSON is valid, the command prints the formatted configuration. If there is a syntax error, correct it before creating the service.

### 5.3 Create the ECS service

Run:

```bash
aws ecs create-service \
  --region us-east-1 \
  --cli-input-json file://create-customer-microservice-tg-two.json
```

A successful response should include service details such as:

- `serviceName`: `customer-microservice`
- `status`: `ACTIVE`
- `desiredCount`: `1`
- `runningCount`: `0` initially
- `deploymentController.type`: `CODE_DEPLOY`

### 5.4 Verify the customer service

Run:

```bash
aws ecs describe-services \
  --region us-east-1 \
  --cluster microservices-serverlesscluster \
  --services customer-microservice \
  --query 'services[0].{Name:serviceName,Status:status,TaskDefinition:taskDefinition,Desired:desiredCount,Running:runningCount,Controller:deploymentController.type}' \
  --output table
```

Confirm that the service is `ACTIVE`, the desired count is `1`, and the deployment controller is `CODE_DEPLOY`.

---

## 6. Task 7.2 — Create the employee microservice ECS service

### 6.1 Create the employee service configuration

Create a separate configuration file named `create-employee-microservice-tg-two.json`:

```bash
cat > create-employee-microservice-tg-two.json <<'EOF'
{
  "cluster": "microservices-serverlesscluster",
  "serviceName": "employee-microservice",
  "taskDefinition": "employee-microservice:1",
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:us-east-1:accountid:targetgroup/employee-tg-two/627d99bba47f5dcf",
      "containerName": "employee",
      "containerPort": 8080
    }
  ],
  "desiredCount": 1,
  "launchType": "FARGATE",
  "schedulingStrategy": "REPLICA",
  "deploymentController": {
    "type": "CODE_DEPLOY"
  },
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-id",
        "subnet-id"
      ],
      "securityGroups": [
        "sg-id"
      ],
      "assignPublicIp": "ENABLED"
    }
  }
}
EOF
```

### 6.2 Validate the JSON

Run:

```bash
python3 -m json.tool create-employee-microservice-tg-two.json
```

Confirm that the configuration contains:

- `employee-microservice:1`
- The `employee-tg-two` target group ARN
- Container name `employee`
- Container port `8080`

### 6.3 Create the ECS service

Run:

```bash
aws ecs create-service \
  --region us-east-1 \
  --cli-input-json file://create-employee-microservice-tg-two.json
```

A successful response should show `employee-microservice` with status `ACTIVE`, desired count `1`, and deployment controller `CODE_DEPLOY`.

### 6.4 Verify the employee service

Run:

```bash
aws ecs describe-services \
  --region us-east-1 \
  --cluster microservices-serverlesscluster \
  --services employee-microservice \
  --query 'services[0].{Name:serviceName,Status:status,TaskDefinition:taskDefinition,Desired:desiredCount,Running:runningCount,Controller:deploymentController.type}' \
  --output table
```

---

## 7. Final verification — Check both services

After creating both services, run one command to verify them together:

```bash
aws ecs describe-services \
  --region us-east-1 \
  --cluster microservices-serverlesscluster \
  --services customer-microservice employee-microservice \
  --query 'services[].{Name:serviceName,Status:status,Desired:desiredCount,Running:runningCount,Controller:deploymentController.type}' \
  --output table
```

### Expected result at the end of Phase 7

The lab's expected state before Phase 8 is:

| Controller | Desired | Name | Running | Status |
|---|---:|---|---:|---|
| `CODE_DEPLOY` | 1 | `customer-microservice` | 0 | `ACTIVE` |
| `CODE_DEPLOY` | 1 | `employee-microservice` | 0 | `ACTIVE` |

Your recorded verification output matched this expected state.

> **Interpretation:** `ACTIVE` confirms that the ECS service resource exists. `Desired = 1` means ECS is configured for one task. `Running = 0` is expected at this point in this lab; Phase 8 configures CodeDeploy and CodePipeline to launch and deploy the task sets. Do not increase the desired count or change the deployment controller just to make the running count become one.

---