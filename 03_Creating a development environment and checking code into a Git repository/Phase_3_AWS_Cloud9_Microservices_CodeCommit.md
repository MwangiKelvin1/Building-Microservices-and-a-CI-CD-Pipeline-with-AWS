# Creating a Development Environment and Checking Code into a Git Repository

## Phase objective

In this phase, an AWS Cloud9 development environment is prepared, the
source code for the existing monolithic application is copied into that
environment, and two working directories are created for the future
`customer` and `employee` microservices. The source code is then
committed to an AWS CodeCommit Git repository.

This phase contains four tasks:

-   **Task 3.1:** Create an AWS Cloud9 IDE as your work environment.
-   **Task 3.2:** Copy the application code to your IDE.
-   **Task 3.3:** Create working directories with starter code for the
    two microservices.
-   **Task 3.4:** Create a Git repository for the microservices code and
    push the code to CodeCommit.

------------------------------------------------------------------------

# Task 3.1 --- Create an AWS Cloud9 IDE as Your Work Environment

## Objective

Create and open the AWS Cloud9 environment that will be used to work on
the application code.

## Choose your preferred configuration

  Setting                   Required value
  ------------------------- --------------------
  Cloud9 environment name   `MicroservicesIDE`
  Compute                   New EC2 instance
  Instance type             `t3.small`
  Operating system          Amazon Linux 2
  Connection                SSH
  VPC                       `LabVPC`
  Subnet                    `Public Subnet1`


Our Cloud9 IDE opens and its terminal is available.


The working directory used by the remaining tasks is:

``` bash
~/environment
```
------------------------------------------------------------------------

# Task 3.2 --- Copy the Application Code to Your IDE

## Objective

Copy the source code for the existing monolithic Node.js application
from the `MonolithicAppServer` EC2 instance into the Cloud9 environment.

I used a temporary directory to receive the files before
they are copied into the two microservice directories in Task 3.3.

## Prerequisites

You need:

-   The `labsuser.pem` key file supplied in the AWS Details panel of the
    lab.
-   The private IPv4 address of the `MonolithicAppServer` instance.
-   A working Cloud9 terminal.
-   Permission and network connectivity to connect from Cloud9 to the
    application server using SSH/SCP.

## Step 1 --- Create the temporary directory

Run:

``` bash
mkdir -p ~/environment/temp
```

Confirm that the directory exists:

``` bash
ls -ld ~/environment/temp
```

## Step 2 --- Find the private IP address of the source server

1.  Open the **Amazon EC2** console.
2.  Find the instance named `MonolithicAppServer`.
3.  Select the instance.
4.  Locate its **Private IPv4 address**.
5.  Copy that private IP address for use in the next command.

Do not substitute the public IP address when the lab specifically
requires the private IPv4 address.

## Step 3 --- Copy the source code with SCP

In the Cloud9 terminal, set a shell variable to the private IP address
you found. Replace `REPLACE_WITH_PRIVATE_IP` with the actual address:

``` bash
appServerPrivIp=REPLACE_WITH_PRIVATE_IP
```

For example, the variable should contain only the IPv4 address, not a
URL or username.

Run the SCP command:

``` bash
scp -r -i ~/environment/labsuser.pem \
  ubuntu@$appServerPrivIp:/home/ubuntu/resources/codebase_partner/* \
  ~/environment/temp/
```

This command:

-   `scp` copies files over SSH.
-   `-r` copies directories recursively.
-   `-i` specifies the SSH private key.
-   `ubuntu@$appServerPrivIp` identifies the remote user and source
    server.
-   `/home/ubuntu/resources/codebase_partner/*` specifies the
    application source files.
-   `~/environment/temp/` is the destination directory in Cloud9.


## Step 4 --- Verify the copied source files

Run:

``` bash
ls -la ~/environment/temp
```

Open the Cloud9 file browser and confirm that the application source
files appear under `temp`.

------------------------------------------------------------------------

# Task 3.3 --- Create Working Directories with Starter Code for the Two Microservices

## Objective

Create two working directories, `customer` and `employee`, and place a
separate copy of the monolithic application source code in each
directory.

The two copies will later be modified so that each microservice has a
distinct responsibility.

## Intended microservice responsibilities

  -----------------------------------------------------------------------
  Microservice            Primary user            Intended functionality
  ----------------------- ----------------------- -----------------------
  `customer`              Customers, represented  Read-only view of
                          by café franchise       supplier contact
                          location managers       information so
                                                  customers can identify
                                                  suppliers from whom to
                                                  buy coffee beans.

  `employee`              Employees in the café   Add, modify, and delete
                          corporate office        supplier records to
                                                  keep supplier listings
                                                  accurate and up to
                                                  date.
  -----------------------------------------------------------------------

At this stage, both directories start with copies of the same monolithic
application code. The code is not yet split into separate functional
implementations; that work is done in a later phase.

The employee microservice will eventually be restricted to employees in
a later project phase.

## Step 1 --- Navigate to the environment directory

Run:

``` bash
cd ~/environment
```

## Step 2 --- Create the microservices directory

If it does not already exist, create it:

``` bash
mkdir -p ~/environment/microservices
```

Navigate into it:

``` bash
cd ~/environment/microservices
```

## Step 3 --- Create the two microservice directories

Run:

``` bash
mkdir -p customer employee
```

Verify the directories:

``` bash
ls -ld customer employee
```

## Step 4 --- Copy the application code into both directories

The source files are in `~/environment/temp`. Copy the contents into
each microservice directory:

``` bash
cp -a ~/environment/temp/. ~/environment/microservices/customer/
cp -a ~/environment/temp/. ~/environment/microservices/employee/
```

The `/.` source pattern copies the contents of `temp`, including hidden
files, into the destination. It does not create an extra `temp`
directory inside each destination.

## Step 5 --- Verify both copies

Run:

``` bash
ls -la ~/environment/microservices/customer
ls -la ~/environment/microservices/employee
```

## Step 6 --- Verify the final directory structure

Run:

``` bash
cd ~/environment/microservices
```

At minimum, the structure should include:

``` text
.
./customer
./employee
```

The two directories should each contain a copy of the application source
code.

## Evidence to capture

-   `customer` and `employee` directories created.
-   Application files visible in both directories.
-   Temporary directory removed after verification.
-   Final project structure visible in the terminal or Cloud9 file
    browser.

------------------------------------------------------------------------

# Task 3.4 --- Create a Git Repository for the Microservices Code and Push the Code to CodeCommit

## Objective

Create an AWS CodeCommit repository named `microservices`, initialize
Git in the local project directory, commit the two unmodified
application copies on the `dev` branch, and push that branch to
CodeCommit.

## Configuration

  -------------------------------------------------------------------------------------------------------------
  Setting                             Value
  ----------------------------------- -------------------------------------------------------------------------
  AWS service                         AWS CodeCommit

  Repository name                     `microservices`

  AWS Region                          `us-east-1` --- US East (N. Virginia)

  Local project directory             `~/environment/microservices`

  Branch                              `dev`

  Remote name                         `origin`

  HTTPS remote URL                    `https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices`
  -------------------------------------------------------------------------------------------------------------

## Step 1 --- Create the CodeCommit repository

1.  Open the AWS Management Console.

2.  Search for **CodeCommit**.

3.  Select Region **US East (N. Virginia)** (`us-east-1`).

4.  Create a repository named:

    ``` text
    microservices
    ```

5.  Complete the creation process and confirm that the repository
    exists.

## Step 2 --- Navigate to the local project directory

Run:

``` bash
cd ~/environment/microservices
```

Confirm the directory:

``` bash
pwd
```

## Step 3 --- Initialize Git

The lab's command is:

``` bash
git init
```

This initializes Git version control in the project directory if it has
not already been initialized.

## Step 4 --- Create or rename the branch to `dev`

Run:

``` bash
git branch -m dev
```

Check the branch:

``` bash
git branch
```

Expected output:

``` text
* dev
```

## Step 5 --- Stage the project files

Run:

``` bash
git add .
```

This stages the files in the current project directory.

## Step 6 --- Commit the starter code

Run:

``` bash
git commit -m 'two unmodified copies of the application code'
```

The commit message records that both microservice directories contain
unmodified copies of the application code at this point in the project.

The commit created during this lab was:

``` text
25861e7 (HEAD -> dev) two unmodified copies of the application code
```

If Git requests a username and email, configure a Git identity as
described in Step 9 below. 

## Step 7 --- Add the CodeCommit remote

Run this command exactly as a single line:

``` bash
git remote add origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices
```

The URL must not include the words `git clone`.

Check the remote configuration:

``` bash
git remote -v
```

Expected output:

``` text
origin  https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices (fetch)
origin  https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices (push)
```

If `origin` is already configured, inspect it with `git remote -v`
rather than repeating `git remote add origin`.

If the existing `origin` points to the wrong URL, update it with:

``` bash
git remote set-url origin https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices
```

## Step 8 --- Push the `dev` branch to CodeCommit

Run:

``` bash
git push -u origin dev
```

The successful push from this lab returned:

``` text
Enumerating objects: 2191, done.
Counting objects: 100% (2191/2191), done.
Delta compression using up to 2 threads
Compressing objects: 100% (2083/2083), done.
Writing objects: 100% (2191/2191), 1.94 MiB | 6.18 MiB/s, done.
Total 2191 (delta 546), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Validating objects: 100%
To https://git-codecommit.us-east-1.amazonaws.com/v1/repos/microservices
 * [new branch]      dev -> dev
branch 'dev' set up to track 'origin/dev'.
```

### What the result means

-   Git transferred the committed objects to CodeCommit.
-   The remote branch `dev` was created.
-   The local branch `dev` now tracks `origin/dev`.
-   The initial code commit is available in the remote repository.

## Step 9 --- Verify in the AWS Console

1.  Open CodeCommit in the AWS Management Console.
2.  Ensure Region `us-east-1` is selected.
3.  Open the `microservices` repository.
4.  Select the `dev` branch.
5.  Verify that the `customer` and `employee` directories and their
    source files are present.
6.  Open the commit history if you want to inspect the initial commit.



------------------------------------------------------------------------

# Phase 3 --- Final Checklist

## Task 3.1: Cloud9 IDE

-   [x] Created the `MicroservicesIDE` Cloud9 environment.
-   [x] Used the required instance and network settings from the lab
    guide.
-   [x] Opened the Cloud9 IDE and confirmed the terminal works.

## Task 3.2: Copy application code

-   [x] Downloaded `labsuser.pem` from the lab's AWS Details panel.
-   [x] Uploaded the key to Cloud9 and set its permissions.
-   [x] Created `~/environment/temp`.
-   [x] Retrieved the private IPv4 address of `MonolithicAppServer`.
-   [x] Copied the source code into `temp` using SCP.
-   [x] Verified that the source files were copied successfully.

## Task 3.3: Create the microservice directories

-   [x] Created `~/environment/microservices/customer`.
-   [x] Created `~/environment/microservices/employee`.
-   [x] Copied the application source into both directories.
-   [x] Verified both copies before deleting the temporary files.
-   [x] Removed the empty `temp` directory.

## Task 3.4: Push the code to CodeCommit

-   [x] Created or confirmed the `microservices` repository in
    CodeCommit.
-   [x] Initialized Git in the local project directory.
-   [x] Set the branch to `dev`.
-   [x] Staged and committed the initial source code.
-   [x] Configured the `origin` remote.
-   [x] Successfully pushed `dev` to CodeCommit.
-   [x] Verified the source files in the CodeCommit console.

**Task 3.4 push status:** Completed successfully. The terminal confirmed
that `dev` was pushed and set to track `origin/dev`.