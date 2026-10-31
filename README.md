# Building Microservices and a CI/CD Pipeline with AWS

## Project Description

This hands-on project takes a monolithic Node.js web application, a coffee suppliers listing app, and re-architects it into independent, containerized microservices on AWS. Participants split the application into **customer** (read-only) and **employee** (admin) microservices, package them as Docker images, store them in Amazon ECR, and run them on a serverless Amazon ECS Fargate cluster behind an Application Load Balancer. An automated CI/CD pipeline built with CodeCommit, CodePipeline, and CodeDeploy then deploys updates using a **blue/green deployment strategy**.

By completing this project, I gained the skills needed to design, containerize, deploy, and iterate on microservice-based workloads on AWS.

## Architecture

![Architecture Diagram](<AWS Cloud CI_CD Architecture Blueprint.png>)

## Scenario

A café corporation has acquired a coffee supplier whose supplier listings application runs as a monolith on a single EC2 instance, with reliability and performance issues. As a newly hired engineer, you are tasked with:

- Splitting the monolith into microservices that can be scaled independently
- Removing single points of failure so one service going down doesn't take the whole application with it
- Building a CI/CD pipeline that automatically deploys updates to the production container cluster

| Microservice | Primary User | Functionality |
|--------------|--------------|---------------|
| **Customer** | Café franchise managers | Read-only view of supplier contact information |
| **Employee** | Corporate office staff | Add, edit, and delete supplier listings (served under `/admin/`) |

## Learning Outcomes

By the end of this project, you will be able to:

- Analyze how a Node.js web application is deployed and connected to a relational database (Amazon RDS)
- Create an AWS Cloud9 IDE and store application code in AWS CodeCommit
- Split a monolithic application into separate containerized microservices
- Build, tag, and version Docker images in Amazon ECR
- Create an Amazon ECS Fargate cluster, task definitions, and services
- Configure an Application Load Balancer with multiple target groups and path-based routing
- Build CI/CD pipelines that deploy containers using blue/green deployments
- Iterate on the application by pushing code changes that automatically trigger the pipeline
- Scale microservices independently

## Solution Requirements

| ID | Requirement | Description |
|----|-------------|-------------|
| R1 | Design | Architecture diagram of the solution |
| R2 | Cost optimized | Cost estimate built with the AWS Pricing Calculator (us-east-1, 12 months) |
| R3 | Microservices-based architecture | Functional, microservice-based deployment |
| R4 | Portability | Application code is not tied to a specific host machine (Docker) |
| R5 | Scalability / resilience | Ability to add compute capacity, with reliable and scalable routing |
| R6 | Automated CI/CD | Pipeline is invoked automatically when code is pushed to version control |

## Technologies Used

**AWS services (11+)**

- Amazon VPC
- Amazon EC2 (instances, Application Load Balancer, target groups)
- Amazon ECS on AWS Fargate
- Amazon ECR
- Amazon RDS (MySQL)
- AWS Cloud9
- AWS CodeCommit
- AWS CodeDeploy
- AWS CodePipeline
- AWS IAM
- Amazon CloudWatch Logs

**Tools and languages**

- Node.js and Express
- Docker
- Git
- Bash and AWS CLI
- JSON (ECS task definitions) and YAML (CodeDeploy AppSpec files)

## Project Phases

| Phase | Description | Requirements |
|-------|-------------|--------------|
| 1 | Plan the design and estimate cost | R1, R2 |
| 2 | Analyze the monolithic application's infrastructure | R3 |
| 3 | Create a Cloud9 environment and check code into CodeCommit | R3 |
| 4 | Split into two microservices and test them in Docker containers | R3, R4 |
| 5 | Create ECR repositories, an ECS cluster, task definitions, and AppSpec files | R3 |
| 6 | Create target groups and an Application Load Balancer | R5 |
| 7 | Create the two Amazon ECS services | R5 |
| 8 | Configure CodeDeploy and CodePipeline | R5, R6 |
| 9 | Update the code to re-trigger the pipeline, restrict access, and scale | R5, R6 |

## Traffic Routing

The Application Load Balancer (`microservicesLB`) uses path-based routing across two listeners and four target groups, which enables blue/green deployments:

| Listener | Rule | Target Group |
|----------|------|--------------|
| HTTP:80 | Default | `customer-tg-two` |
| HTTP:80 | Path is `/admin/*` | `employee-tg-two` |
| HTTP:8080 (test) | Default | `customer-tg-one` |
| HTTP:8080 (test) | Path is `/admin/*` | `employee-tg-one` |

CodeDeploy swaps traffic between the "one" and "two" target groups on each deployment.

## 🔄 CI/CD Flow

1. A developer commits code or pushes a new Docker image tagged `latest` to Amazon ECR
2. CodePipeline detects the change (CodeCommit `deployment` repo or ECR image)
3. CodePipeline passes the task definition and AppSpec file to CodeDeploy
4. CodeDeploy launches a replacement task set (green) on ECS Fargate
5. Traffic is rerouted to the green task set
6. The original task set (blue) is terminated after 5 minutes, leaving a rollback window

## 🏫 Getting Started

### Prerequisites

- An AWS account or AWS Academy Learner Lab environment with access to the required services
- Basic familiarity with the Linux command line
- Basic understanding of Docker, Git, and the AWS Management Console

### Setup Steps

1. **Start the lab environment** and note your remaining budget
2. **Review the project phases** above and complete them in order
3. **Create the architecture diagram and cost estimate** (Phase 1) before building
4. **Build, deploy, and test** each microservice following Phases 2 through 9

## Repository Structure

```text
.
├── README.md
├── 01_Planning the design and estimating cost/
├── 02_Analyzing the infrastructure of the monolithic application/
├── 03_Creating a development environment and checking code into a Git repository/
├── 04_Configuring the application as two microservices and testing them in Docker containers/
├── 05_Create ECR repositories and upload the Docker images/
├── 06_Creating target groups and an Application Load Balancer/
├── 07_Creating two Amazon ECS services/
├── 08_Configuring CodeDeploy and CodePipeline/
├── 09_Adjusting the microservice code to cause a pipeline to run again/
├── Screenshot/
└── AWS Cloud CI_CD Architecture Blueprint.png
```

## 📚 Getting Help

If you encounter issues while working through the project:

- Review the phase instructions carefully
- Consult the AWS documentation for [Amazon ECS](https://docs.aws.amazon.com/ecs/), [AWS CodeDeploy](https://docs.aws.amazon.com/codedeploy/), and [AWS CodePipeline](https://docs.aws.amazon.com/codepipeline/)
- Ensure all prerequisites are met and resources are properly deployed

## 👤 Author

| Name | GitHub Username |
|------|-----------------|
| Kelvin Mwangi Gitimu Kabuchu | [@MwangiKelvin1](https://github.com/MwangiKelvin1) |

## 📄 License

This project is for educational purposes as part of the AWS Academy curriculum.
