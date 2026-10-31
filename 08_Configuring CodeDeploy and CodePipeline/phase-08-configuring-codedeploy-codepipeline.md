# Phase 8: Configuring CodeDeploy and CodePipeline

## 1. Phase Overview

Phase 8 configures AWS CodeDeploy and AWS CodePipeline to deploy the
customer and employee microservices to Amazon ECS on AWS Fargate using
blue/green deployments. The phase creates separate pipelines for the two
microservices, tests their deployments, and observes how CodeDeploy
updates the Application Load Balancer (ALB) listener rules and
target-group associations.

The goal is to automate the path from deployment configuration and
container image selection to a running production task set, reducing the
need to manually update ECS services whenever a new image is deployed.

## 2. Objectives

By the end of this phase, the environment should have:

-   One CodeDeploy application named `microservices`.
-   A deployment group for the customer microservice named
    `microservices-customer`.
-   A deployment group for the employee microservice named
    `microservices-employee`.
-   A CodePipeline named `update-customer-microservice`.
-   A CodePipeline named `update-employee-microservice`.
-   Each pipeline configured to use the deployment files in the
    CodeCommit repository and the appropriate container image in Amazon
    ECR.
-   Blue/green deployments configured for the ECS services.
-   Application behavior tested through the load balancer.
-   An understanding of how CodeDeploy changes ALB listener rules and
    target-group associations during deployment.

## 3. Environment and Resources

  ----------------------------------------------------------------------------------------------
  Component                           Value
  ----------------------------------- ----------------------------------------------------------
  AWS Region                          `us-east-1` (US East -- N. Virginia)

  CodeDeploy application              `microservices`

  Customer deployment group           `microservices-customer`

  Employee deployment group           `microservices-employee`

  Customer pipeline                   `update-customer-microservice`

  Employee pipeline                   `update-employee-microservice`

  CodeCommit repository               `deployment`

  Source branch                       `dev`

  Customer ECR repository             `customer`

  Employee ECR repository             `employee`

  ECS cluster                         `microservices-serverlesscluster`

  Application Load Balancer DNS       `microserviceslb-1648296002.us-east-1.elb.amazonaws.com`

  Customer application port           HTTP `80`

  Employee application port used for  HTTP `8080`
  testing                             
  ----------------------------------------------------------------------------------------------

The load balancer DNS name above is the one used during this lab
session. The DNS name assigned to an ALB can differ between
environments.

## 4. Architecture

``` mermaid
flowchart TD
    Dev[Deployment files in CodeCommit<br/>deployment / dev]
    CustImg[Amazon ECR<br/>customer image]
    EmpImg[Amazon ECR<br/>employee image]
    CustPipe[CodePipeline<br/>update-customer-microservice]
    EmpPipe[CodePipeline<br/>update-employee-microservice]
    CD[CodeDeploy<br/>microservices]
    ECS[Amazon ECS on AWS Fargate]
    ALB[Application Load Balancer]
    CustSvc[Customer microservice]
    EmpSvc[Employee microservice]
    DB[(Amazon RDS<br/>supplier database)]

    Dev --> CustPipe
    Dev --> EmpPipe
    CustImg --> CustPipe
    EmpImg --> EmpPipe
    CustPipe --> CD
    EmpPipe --> CD
    CD --> ECS
    ALB --> CustSvc
    ALB --> EmpSvc
    CustSvc --> DB
    EmpSvc --> DB
```

### Architecture explanation

1.  **AWS CodeCommit** stores the deployment files used by the
    pipelines. The repository is `deployment`, and the configured source
    branch is `dev`.
2.  **Amazon ECR** stores the container images. The customer and
    employee pipelines each use the matching ECR repository.
3.  **AWS CodePipeline** coordinates source retrieval, image selection,
    and deployment.
4.  **AWS CodeDeploy** manages blue/green deployment of the ECS services
    and coordinates the transition of traffic between task sets.
5.  **Amazon ECS on AWS Fargate** runs the customer and employee
    containers without requiring the lab user to manage the underlying
    container hosts.
6.  **The Application Load Balancer** receives application traffic and
    forwards requests according to its listener rules and target-group
    configuration.
7.  **Amazon RDS** stores the supplier data used by the application.

## 5. Task 8.1 --- Create the CodeDeploy Application and Deployment Groups

### Purpose

The CodeDeploy application groups deployment resources for the
microservices. The deployment groups define the ECS services and
traffic-management settings that CodeDeploy uses during blue/green
deployment.

### Configuration

-   CodeDeploy application: `microservices`
-   Compute platform: Amazon ECS
-   Customer deployment group: `microservices-customer`
-   Employee deployment group: `microservices-employee`

The deployment groups must correspond to the correct ECS services and
load-balancer target groups. The customer deployment group controls
deployments for the customer microservice, while the employee deployment
group controls deployments for the employee microservice.

### Result

The CodeDeploy application and both deployment groups were created and
later used successfully by their respective pipelines.

## 6. Task 8.2 --- Create the Customer Microservice Pipeline

### Purpose

The customer pipeline automates deployment of the customer container
image to the ECS customer service using CodeDeploy blue/green
deployment.

### Pipeline

`update-customer-microservice`

### Pipeline configuration

-   Source provider: AWS CodeCommit
-   Repository: `deployment`
-   Branch: `dev`
-   Deployment provider: Amazon ECS (Blue/Green)
-   CodeDeploy application: `microservices`
-   CodeDeploy deployment group: `microservices-customer`

The pipeline uses the deployment repository's customer task-definition
and AppSpec files as its deployment inputs. The pipeline also uses the
customer container image from the `customer` Amazon ECR repository.

### Deployment behavior

When the pipeline executes, it obtains the configured source artifacts
and deploys the selected image to the ECS service. CodeDeploy creates or
updates a replacement task set, checks deployment progress, and shifts
production traffic when the configured deployment conditions are met.

## 7. Task 8.3 --- Test the Customer Pipeline

### Purpose

This task validates that the customer pipeline can execute a deployment
and that the customer microservice remains available through the load
balancer.

### Validation performed

-   The customer pipeline executed successfully.
-   The CodeDeploy deployment group `microservices-customer` showed a
    successful deployment.
-   The customer application home page loaded through the ALB using HTTP
    on port `80`.

### Application URL

`http://microserviceslb-1648296002.us-east-1.elb.amazonaws.com/`

### Result

The customer microservice was deployed through the automated pipeline.
The application home page was accessible through the load balancer after
deployment.

## 8. Task 8.4 --- Create the Employee Microservice Pipeline

### Purpose

The employee pipeline automates deployment of the employee microservice,
which provides the administrative supplier functionality.

### Pipeline

`update-employee-microservice`

### Source configuration

-   Source provider: AWS CodeCommit
-   Repository: `deployment`
-   Branch: `dev`
-   Source artifact: `SourceArtifact`

### Employee image source

-   Action name: `Image`
-   Provider: Amazon ECR
-   Repository: `employee`
-   Image tag: `latest`
-   Output artifact: `image-employee`

### Deployment configuration

-   Deploy provider: Amazon ECS (Blue/Green)
-   CodeDeploy application: `microservices`
-   CodeDeploy deployment group: `microservices-employee`
-   Task definition file: `taskdef-employee.json`
-   AppSpec file: `appspec-employee.yaml`
-   Task-definition artifact: `SourceArtifact`
-   AppSpec artifact: `SourceArtifact`

The employee task-definition template uses the image placeholder
`<IMAGE1_NAME>`. The pipeline's image action supplies the employee image
reference during deployment. Keeping the placeholder in the
task-definition template is important because it allows the pipeline to
substitute the image from ECR rather than attempting to pull an
unrelated image reference.

### Result

The employee pipeline was configured and a successful deployment was
achieved after correcting the dynamic image reference configuration.

## 9. Task 8.5 --- Test the Employee Pipeline

### Purpose

This task checks whether the employee microservice is deployed
successfully and whether the application can be accessed through the
load balancer.

### Deployment result

The CodeDeploy deployment group `microservices-employee` showed
**Succeeded**. The successful deployment completed the blue/green
traffic-shift process, with the replacement task set serving production
traffic after the deployment and bake period.

The employee container was deployed from the `employee` ECR repository
through the `update-employee-microservice` pipeline.

### Employee application URL tested

`http://microserviceslb-1648296002.us-east-1.elb.amazonaws.com:8080/admin/suppliers`

The employee suppliers page loaded at this URL during testing.

### Customer application URL tested

`http://microserviceslb-1648296002.us-east-1.elb.amazonaws.com/`

The customer home page loaded on port `80`.

### Outstanding application-path observation

Opening the customer application's supplier-list path at:

`http://microserviceslb-1648296002.us-east-1.elb.amazonaws.com/suppliers`

returned a `404` page during testing. The employee supplier page was
accessible at `/admin/suppliers` on port `8080`.

This observation is recorded as an application-path or listener-routing
item to verify. It should not be treated as proof that the employee
deployment failed, because the employee deployment itself succeeded and
the `/admin/suppliers` endpoint loaded. The expected behavior in the
project instructions is that the café application links work after both
microservices are deployed, so the `404` should be checked against the
ALB listener rules and the application's route configuration before
considering Task 8.5 fully validated.

### Deployment verification

The deployment group reported a successful deployment. The employee
deployment ID observed during the lab was `d-Q9I68GPHM`.

## 10. Task 8.6 --- Observe How CodeDeploy Modified the Load Balancer Listener Rules

### Purpose

CodeDeploy manages traffic shifting during blue/green deployment. As
part of that process, it can change which target group is associated
with a listener rule and which target group receives production traffic.

### Checks specified by the project

1.  Open the Amazon EC2 console and choose **Target Groups**.
2.  Refresh the page if it was already open.
3.  Check whether `customer-tg-two` is still associated with the load
    balancer.
4.  Inspect the rules for the HTTP listener on port `80`.
5.  Inspect the rules for the HTTP listener on port `8080`.

### Expected observations described in the lab guide

-   After the customer blue/green deployment, `customer-tg-two` may no
    longer be associated with the load balancer because CodeDeploy is
    managing the listener rules.
-   The default rule for HTTP `80` may have changed from forwarding to
    `customer-tg-two` to forwarding to `customer-tg-one`.
-   The two HTTP `8080` listener rules should continue forwarding to the
    corresponding "one" target groups after the deployments described in
    the guide.
-   Target-group associations can differ if the lab has been repeated,
    because CodeDeploy may have switched the active target groups during
    earlier deployments.

### Current documentation status

The CodeDeploy deployments are confirmed successful. The exact current
listener-rule and target-group associations still need to be compared
with the observations above. The `/suppliers` `404` noted in Task 8.5
makes this inspection particularly relevant.

Do not record the expected rule configuration as an observed fact until
the current listener rules have been checked in the AWS console.

## 11. Troubleshooting Notes

### 11.1 Employee image placeholder

**Symptom:** A deployment attempts to pull an unintended image reference
rather than the employee image from ECR.

**Cause identified:** The task-definition template must preserve the
`<IMAGE1_NAME>` placeholder so that the pipeline can replace it with the
image produced by the ECR source action.

**Resolution:** Restore the placeholder in `taskdef-employee.json`,
commit the corrected file to the `deployment` repository, and rerun the
pipeline.

### 11.2 Employee container could not connect to the database

**Symptom:** Employee container logs showed a MySQL connection timeout
(`connect ETIMEDOUT`).

**Relevant configuration reviewed:**

-   Employee task security group: `microservices-sg`
-   RDS database: `supplierdb`
-   Database port: TCP `3306`
-   Database security group: `DBSecurityGroup`

The task security group's outbound rules initially did not include
MySQL/Aurora TCP `3306`. An outbound rule for TCP `3306` was added
during troubleshooting, after which the employee deployment succeeded.

**Security note:** The troubleshooting rule used destination
`0.0.0.0/0`, which is broad. For a hardened configuration, restrict
database traffic to the RDS security group or another narrowly defined
destination appropriate to the lab architecture. Do this only after
confirming the required traffic path and validating the application.

### 11.3 Customer supplier-list path returns 404

**Symptom:** The ALB home page loads on port `80`, but `/suppliers`
returns a `404` page. The employee endpoint `/admin/suppliers` loads on
port `8080`.

**Next checks:**

-   Inspect the HTTP `80` listener's default and path-based rules.
-   Inspect the HTTP `8080` listener's path-based rules.
-   Confirm the rules forward to the intended customer and employee
    target groups.
-   Confirm the relevant target groups are healthy and associated with
    the ALB.
-   Compare the current configuration with Task 8.6 of the project
    guide.
-   Re-test the café application's supplier-list link after verifying
    the routing configuration.

A successful CodeDeploy deployment alone does not prove that every
application URL and listener rule is correct.

## 12. Security and Reliability Considerations

### Blue/green deployments

Blue/green deployments maintain a current task set while a replacement
task set is deployed and validated. Traffic is shifted to the
replacement task set when deployment checks succeed. This can reduce
deployment downtime and provide a controlled transition between
versions.

### Separate pipelines

Keeping customer and employee pipelines separate allows each
microservice to be deployed independently. A change to one service does
not automatically require rebuilding and deploying the other service,
provided the change does not introduce a dependency that requires
coordinated updates.

### Image provenance

The employee pipeline retrieves the container image from the `employee`
ECR repository. The task-definition placeholder must be configured
correctly so the deployed task uses the image selected by the pipeline.

### Network access

Security-group rules should allow only the network flows required by the
application. In particular, database access should be limited to the
application tasks that need it, rather than broadly permitting MySQL
traffic to every IPv4 address.

### Listener and target-group configuration

The ALB listener rules are part of the application's traffic-control
layer. After a blue/green deployment, verify the active target groups
and routing rules rather than assuming they remain unchanged.

## 13. CI/CD Relevance

This phase demonstrates the key elements of a CI/CD workflow:

1.  **Source control:** Deployment configuration is stored in
    CodeCommit.
2.  **Image storage:** Container images are stored in Amazon ECR.
3.  **Pipeline orchestration:** CodePipeline coordinates the deployment
    workflow.
4.  **Deployment automation:** CodeDeploy performs the ECS blue/green
    deployment.
5.  **Traffic management:** The ALB directs requests to the active
    target groups.
6.  **Validation:** Pipeline status, deployment status, running ECS
    tasks, and application URLs are checked.
7.  **Troubleshooting:** Failed image substitution and database
    connectivity were investigated and corrected.
8.  **Operational verification:** Listener rules and target-group
    associations are checked after deployment.

The phase illustrates that a pipeline being green is an important
deployment signal, but application-level validation is also required. A
successful deployment should be followed by checks of the expected
routes, load-balancer rules, target health, and application behavior.

## 14. Phase Outcome

### Completed

-   Created the CodeDeploy application `microservices`.
-   Created and used the `microservices-customer` deployment group.
-   Created and used the `microservices-employee` deployment group.
-   Configured the `update-customer-microservice` pipeline.
-   Configured the `update-employee-microservice` pipeline with
    CodeCommit and ECR inputs.
-   Achieved successful customer and employee CodeDeploy deployments.
-   Confirmed that the customer home page loads through the ALB on port
    `80`.
-   Confirmed that the employee `/admin/suppliers` page loads through
    the ALB on port `8080`.
-   Corrected the employee task-definition image placeholder during
    troubleshooting.
-   Investigated employee database connectivity and added the missing
    outbound MySQL rule during the lab.

### Follow-up verification

-   Confirm the current HTTP `80` and HTTP `8080` listener rules and
    target-group associations.
-   Investigate why `/suppliers` on port `80` returned `404`, because
    the lab guide expects the café application links to work after both
    microservices have been deployed.
-   If the security group is hardened, verify that the employee task can
    still connect to RDS after replacing the broad MySQL destination
    with a narrowly scoped rule.

### Conclusion

Phase 8 established automated blue/green deployment pipelines for both
the customer and employee microservices using AWS CodePipeline,
CodeDeploy, Amazon ECR, Amazon ECS on Fargate, and an Application Load
Balancer. Both deployment groups reached a successful state. The
customer home page and employee administrative supplier page were
accessible through the load balancer. Final validation of listener rules
and the customer `/suppliers` route remains necessary before declaring
all application paths verified.
