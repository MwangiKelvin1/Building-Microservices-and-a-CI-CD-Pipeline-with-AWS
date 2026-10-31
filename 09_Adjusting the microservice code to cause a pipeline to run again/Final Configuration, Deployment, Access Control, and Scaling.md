# Phase 9 — Final Configuration, Deployment, Access Control, and Scaling

## Overview

**Project:** Building Microservices and a CI/CD Pipeline in AWS  
**AWS Region:** `us-east-1`  
**Load balancer:** `microservicesLB`  
**ECS cluster:** `microservices-serverlesscluster`

This phase finalizes the employee microservice's access-routing behavior and navigation styling, runs the employee deployment pipeline, and scales the customer microservice to three running tasks.

## Task 9.1 — Restrict Employee Routes by Source IP

### Objective
Add a source-IP condition to the employee `/admin/*` listener rules so employee routes are handled according to the configured permitted source IP.

### Load balancer details

- **Load balancer DNS:** `microservicesLB-1648296002.us-east-1.elb.amazonaws.com`
- **HTTP listener on port 80**
- **HTTP listener on port 8080**
- **Employee target group on port 80:** `employee-tg-one`
- **Employee target group on port 8080:** `employee-tg-two`

### Configuration
The `/admin/*` rules include both:
- Path-pattern condition: `/admin/*`
- Source-IP condition: the configured permitted IP/CIDR

The employee forwarding actions are preserved:
- Port 80 → `employee-tg-one`
- Port 8080 → `employee-tg-two`

### Important behavior
The source-IP condition is a routing condition, not necessarily an explicit deny rule. When a request does not match the employee rule, it can fall through to the listener's default customer route.

## Task 9.2 — Update Employee Navigation Styling

### Objective
Change the employee application's navigation bar from a dark theme to a light theme.

### Change made
In `employee/views/nav.html`, replace:

```html
navbar-dark bg-dark
```

with:

```html
navbar-light bg-light
```

## Task 9.3 — Run the Employee Microservice Pipeline

### Objective
Deploy the updated employee microservice through its CI/CD pipeline.

### Verification
The pipeline stages reported:
- **Source:** Succeeded
- **Deploy:** Succeeded

## Task 9.4 — Validate the Application in a Browser

### Objective
Check the employee route and customer fallback behavior through the Application Load Balancer.

### URLs to test

1. Employee suppliers page through port 80:

   `http://microservicesLB-1648296002.us-east-1.elb.amazonaws.com/admin/suppliers`

2. Employee suppliers page through port 8080:

   `http://microservicesLB-1648296002.us-east-1.elb.amazonaws.com:8080/admin/suppliers`

3. Customer application:

   `http://microservicesLB-1648296002.us-east-1.elb.amazonaws.com/`

### Expected observations
- From the permitted source IP, the employee route should reach the employee application and display **“Manage coffee suppliers”** with the updated light navigation styling.
- From an unauthorized source IP, the employee rule should not match; the request may fall through to the default customer route.
- The project guide's expected customer fallback includes **“Coffee suppliers”** and the 404 message: `Sorry, we don't seem to have that page in stock`.

> **Evidence note:** The pipeline and scaling results were shared, but browser-validation results for this task were not confirmed in the conversation. Mark the checks below only after performing the tests.


## Task 9.5 — Scale the Customer Microservice to Three Tasks

### Objective
Set the desired task count for `customer-microservice` to three.

### Command used

```bash
aws ecs update-service \
  --cluster microservices-serverlesscluster \
  --service customer-microservice \
  --desired-count 3 \
  --region us-east-1
```

### Verification result

The final terminal output showed:

| Desired | Pending | Running | Service |
|---:|---:|---:|---|
| 3 | 0 | 3 | `customer-microservice` |