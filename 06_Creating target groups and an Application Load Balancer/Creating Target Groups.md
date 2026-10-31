# Phase 6 — Creating Target Groups and an Application Load Balancer
 
**Tasks:** 6.1 Create four target groups; 6.2 Create a security group, an Application Load Balancer, and routing rules  
**Region shown in the lab session:** US East (N. Virginia) — `us-east-1`

---

## 1. Phase objective

In this phase, create four target groups and an internet-facing Application Load Balancer (ALB). The ALB provides an entry point to the application and uses listener rules to route requests to the customer or employee microservice.

The project uses two target groups per microservice because the later CodeDeploy blue/green deployment configuration needs a pair of target groups for each microservice.

### Expected resources

- `customer-tg-one`
- `customer-tg-two`
- `employee-tg-one`
- `employee-tg-two`
- `microservices-sg`
- `microservicesLB`
- Listener `HTTP:80`
- Listener `HTTP:8080`
- One `/admin/*` path rule on each listener

## 2. Key concepts

| Term | Meaning in this project |
|---|---|
| Target group | A collection of destinations to which the load balancer can forward requests. This lab uses IP-address targets. |
| Health check | A request the load balancer sends to check whether a target is healthy. |
| Listener | Checks incoming connections on a configured protocol and port and evaluates listener rules. |
| Default action | The action used when no additional listener rule matches. |
| Path-based routing | Uses a request path, such as `/admin/suppliers`, to decide which target group receives a request. |
| Blue/green deployment | Uses separate environments/task sets so CodeDeploy can shift traffic between deployment versions. The paired target groups support this design. |

**Important distinction:** `/admin/*` routing is not authentication or authorization. It directs matching requests to the employee microservice; application security controls must separately determine who is allowed to perform administrative actions.

---

# Task 6.1 — Create four target groups

## 3. Target-group configuration

All four target groups use IP addresses as the target type, HTTP, port `8080`, and `LabVPC`. Do not register targets during this phase.

| Setting | `customer-tg-one` | `customer-tg-two` | `employee-tg-one` | `employee-tg-two` |
|---|---|---|---|---|
| Target type | IP addresses | IP addresses | IP addresses | IP addresses |
| Protocol | HTTP | HTTP | HTTP | HTTP |
| Port | `8080` | `8080` | `8080` | `8080` |
| VPC | `LabVPC` | `LabVPC` | `LabVPC` | `LabVPC` |
| Health-check path | `/` | `/` | `/admin/suppliers` | `/admin/suppliers` |
| Register targets now? | No | No | No | No |

## 4. Create `customer-tg-one`

1. Open the AWS Management Console and navigate to **EC2**.
2. In the navigation pane, choose **Target Groups**.
3. Choose **Create target group**.
4. Configure:
   - **Choose a target type:** IP addresses
   - **Target group name:** `customer-tg-one`
   - **Protocol:** HTTP
   - **Port:** `8080`
   - **VPC:** `LabVPC`
   - **Health check path:** `/`
5. Choose **Next**.
6. On the **Register targets** page, leave the defaults unchanged and do not register any targets.
7. Choose **Create target group**.

## 5. Create `customer-tg-two`

Repeat the preceding steps using the same settings, except set the target group name to:

`customer-tg-two`

Health-check path remains `/`. Do not register targets.

## 6. Create `employee-tg-one`

Repeat the same process with these settings:

- **Target type:** IP addresses
- **Target group name:** `employee-tg-one`
- **Protocol:** HTTP
- **Port:** `8080`
- **VPC:** `LabVPC`
- **Health check path:** `/admin/suppliers`
- **Register targets:** None

## 7. Create `employee-tg-two`

Repeat the same process with these settings:

- **Target type:** IP addresses
- **Target group name:** `employee-tg-two`
- **Protocol:** HTTP
- **Port:** `8080`
- **VPC:** `LabVPC`
- **Health check path:** `/admin/suppliers`
- **Register targets:** None

---

# Task 6.2 — Create the security group and Application Load Balancer

## 8. Create `microservices-sg`

1. In the EC2 console, open **Security Groups**.
2. Choose **Create security group**.
3. Set:
   - **Security group name:** `microservices-sg`
   - **Description:** for example, `Security group for the microservices Application Load Balancer`
   - **VPC:** `LabVPC`
4. Add the following inbound rules:

| Type | Protocol | Port range | Source |
|---|---|---:|---|
| Custom TCP (or HTTP where available) | TCP | `80` | Anywhere IPv4 (`0.0.0.0/0`) |
| Custom TCP | TCP | `8080` | Anywhere IPv4 (`0.0.0.0/0`) |

5. Review the inbound rules and choose **Create security group**.

The lab specifies inbound TCP access from any IPv4 address on ports 80 and 8080. This is broad internet access; use it only as required for this lab environment and tighten it for real deployments where appropriate.

## 9. Create `microservicesLB`

1. In the EC2 console, choose **Load Balancers**.
2. Choose **Create load balancer**.
3. Under **Application Load Balancer**, choose **Create**.
4. Configure the load balancer:

| Setting | Required value |
|---|---|
| Load balancer name | `microservicesLB` |
| Scheme | Internet-facing |
| IP address type | IPv4 |
| VPC | `LabVPC` |
| Availability zones/subnets | `Public Subnet1` and `Public Subnet2` |
| Security group | `microservices-sg` |

5. In the listener configuration, create or configure both listeners:
   - `HTTP:80` — default action forwards to `customer-tg-two`.
   - `HTTP:8080` — default action forwards to `customer-tg-one`.
6. Review the configuration and choose **Create load balancer**.

If the console asks for a default target group while configuring each listener, choose the appropriate customer target group from the list above. Do not swap the two mappings.

### Listener verification before adding rules

| Listener | Default action |
|---|---|
| `HTTP:80` | Forward to `customer-tg-two` |
| `HTTP:8080` | Forward to `customer-tg-one` |

---

## 10. Add the path-based rule to `HTTP:80`

The required logic is:

> IF the request path is `/admin/*`, THEN forward to `employee-tg-two`.

1. Open **EC2 → Load Balancers → `microservicesLB`**.
2. Open **Listeners and rules**.
3. Select the `HTTP:80` listener.
4. Choose **Add rule**.
5. Set **Priority** to `1`, provided priority 1 is not already used by another non-default rule on this listener.
6. Under **Conditions**, choose **Add condition** and select **Path** or **Path pattern**.
7. Enter this exact pattern:

   `/admin/*`

8. Under **Actions**, select **Forward to target groups**.
9. Select `employee-tg-two`.
10. Leave its weight at `1` (100%) and leave target group stickiness off.
11. Review and save/create the rule.

Do not change the existing default action. It must continue forwarding to `customer-tg-two`.

### Expected `HTTP:80` rules

| Priority | Condition | Action |
|---:|---|---|
| `1` | Path is `/admin/*` | Forward to `employee-tg-two` |
| Default (last) | If no other rule applies | Forward to `customer-tg-two` |

---

## 11. Add the path-based rule to `HTTP:8080`

The required logic is:

> IF the request path is `/admin/*`, THEN forward to `employee-tg-one`.

1. Return to `microservicesLB` → **Listeners and rules**.
2. Select the `HTTP:8080` listener.
3. Choose **Add rule**.
4. Set **Priority** to `1`, provided that priority is available on this listener.
5. Add a **Path** or **Path pattern** condition with this exact value:

   `/admin/*`

6. Under **Actions**, select **Forward to target groups**.
7. Select `employee-tg-one`.
8. Leave the weight at `1` (100%) and target group stickiness off.
9. Review and save/create the rule.

Do not change the existing default action. It must continue forwarding to `customer-tg-one`.

### Expected `HTTP:8080` rules

| Priority | Condition | Action |
|---:|---|---|
| `1` | Path is `/admin/*` | Forward to `employee-tg-one` |
| Default (last) | If no other rule applies | Forward to `customer-tg-one` |

**Priority note:** Rule priorities must be unique within a listener. Priority `1` is correct if it is available. The default rule is evaluated last and does not use a normal numeric priority.

---

# 12. Understand the final routing

The two listeners operate independently. Each listener first evaluates its numbered rules from the lowest priority number upward. The first matching rule is used; if none matches, the default action is used.

| Incoming request | Listener | Matching rule | Target group |
|---|---|---|---|
| `/admin/suppliers` | HTTP:80 | `/admin/*` | `employee-tg-two` |
| `/customers` | HTTP:80 | Default | `customer-tg-two` |
| `/admin/suppliers` | HTTP:8080 | `/admin/*` | `employee-tg-one` |
| `/customers` | HTTP:8080 | Default | `customer-tg-one` |

For a browser request, port 80 is used by default for HTTP. To explicitly use port 8080, include `:8080` after the load balancer hostname, for example:

- `http://<load-balancer-DNS-name>/admin/suppliers`
- `http://<load-balancer-DNS-name>:8080/admin/suppliers`
