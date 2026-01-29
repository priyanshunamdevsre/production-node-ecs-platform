# production-node-ecs-platform
Production-grade Node.js application deployed on AWS ECS with Terraform-managed infrastructure, secure CI/CD, IAM-based RDS authentication, container security scanning, and CloudWatch monitoring.



## Dockerized Node.js Application
This repository contains a Dockerfile to build and run a Node.js application in a Docker container. The Dockerfile is optimized for production use.

## Getting Started

To get started with running this Dockerized Node.js application, follow these steps:

### Prerequisites

- Docker installed on your system. You can download and install Docker from [here](https://www.docker.com/get-started).

### Building the Docker Image

1. Clone this repository to your local machine:

    ```bash
    git clone git@github.com:satyam0710/node-boilerplate-ecs.git
    ```

2. Navigate to the root directory of the cloned repository:

    ```bash
    cd node-express-mysql-boilerplate
    ```

3. Build the Docker image using the provided Dockerfile:

    ```bash
    docker build -t <image-name> .
    ```

### Running the Docker Container

Once you have built the Docker image, you can run the container using the following command:

```bash
docker run -p 3000:3000 -d <image-name>
```

This command will start the container in detached mode and expose port 3000 of the container to port 3000 on your host machine.

## Accessing the Application
You can access the running application by navigating to http://localhost:3000 in your web browser.

## After Application is live
You can access the running application by navigating to https://app.clouddemo.top in your web browser.

# Automated Deployment Workflow for Node Web App on AWS ECS
This GitHub Actions workflow automates the deployment process for a Node Web App  to **AWS Elastic Container Service (ECS)**. Workflow description:

* **Docker Image Building**: Efficiently builds Docker images encapsulating the Node Web app.

* **Vulnerability Scanning**: Conducts thorough vulnerability scans on the Docker images to ensure robust security measures.

* **ECS Deployment**: Seamlessly deploys the application onto AWS ECS by registering a new task definition revision and updating the ECS service, enabling controlled and reliable deployments.

## Workflow Steps

* **Checkout Code**: Checks out the source code from the repository.

* **Setup AWS Credentials**: Configures AWS credentials to authenticate with Amazon ECR.

* **Login to Amazon ECR**: Logs in to ECR to enable Docker image pushes.

* **Build and Push Docker Image**: Builds Docker images for the Node.js application, tags them with `latest` and the commit ID, and pushes them to ECR.

* **Run Trivy Vulnerability Scanner**: Scans the Docker image for vulnerabilities using Trivy and outputs the results.

* **Render ECS Task Definition**: Updates the container image in the ECS task definition JSON.

* **Deploy to ECS Service**: Registers a new task definition revision and updates the ECS service, waiting for service stability.

* **Optionally sends Alerts to Alack**: Slack alerts can be turned on using input enable_slack_alert.

## Improvements we can do

* Enhance the workflow to support multiple environments such as staging, testing, and production.
* Parameterize environment-specific configurations within ECS task definitions.
* Use GitHub Actions environments and approvals for controlled production deployments.

# Node Boilerplate Infrastructure

## Usage

* Example modules

```
module "vpc" {
  source                     = "./modules/vpc"
  name                       = "${var.name}-${var.environment}"
  region                     = var.region
  cidr_block                 = var.cidr_block
  private_subnet_cidr_blocks = var.private_subnet_cidr_blocks
  public_subnet_cidr_blocks  = var.public_subnet_cidr_blocks
  azs                        = var.azs
  environment                = var.environment
  tags                       = var.tags
  public_subnet_tags         = var.public_subnet_tags
  private_subnet_tags        = var.private_subnet_tags
}

module "ecs" {
  source              = "git@github.com:satyam0710/terraform-aws-modules.git//terraform/modules/ecs?ref=main"
  name                = var.name
  environment         = var.environment
  ecs_cluster_name    = var.ecs_cluster_name
  ecs_service_name    = var.ecs_service_name
  launch_type         = var.launch_type
  cpu                 = var.cpu
  memory              = var.memory
  desired_count       = var.desired_count
  vpc_id              = module.vpc.id
  subnet_ids          = module.vpc.private_subnet_ids
  assign_public_ip    = false
}
```

### Steps To Use

* Customize the module by providing values for the required variables (for example in `terraform.tfvars`).

* Clone this repository to your local machine.

```
git clone <repo_url>
```

* Change directory to terraform

```
cd ecs-terraform/
```

* Run terraform init to initialize the module.

```
terraform init
```

* Run terraform plan to see the execution plan.

```
terraform plan -var-file terraform.tfvars
```

* Run terraform apply to create the resources.

```
terraform apply -var-file terraform.tfvars
```

* To switch the workspace

```
terraform workspace select staging
```

## Terraform pre-commit hooks


Execute this command to run `pre-commit` on all files in the repository (not only changed files):

```
cd ecs-terraform
pre-commit run -a  #This will format the code and creates terraform docs
```

### [tfenv](https://github.com/tfutils/tfenv) - Terraform version manager

[tfenv](https://github.com/tfutils/tfenv) is a Terraform version manager which would help us on setting up the right
terraform client version and update it as we keep moving forward.

Install via Homebrew:

```shell
brew install tfenv
```

```shell
tfenv install 1.5.7
tfenv use 1.5.7
cd terraform
terraform init
```

### [TF Lint](https://github.com/terraform-linters/tflint)

```shell
pip install pre-commit
brew install tflint
brew install terraform-docs
brew install gawk
brew install checkov
brew install tfsec
tflint --init
```

## Accessing MySQL / RDS from ECS (IAM Authentication)

ECS tasks can securely access **Amazon RDS (MySQL)** using **IAM authentication**, without storing database passwords.

### Prerequisites

* RDS MySQL with **IAM DB authentication enabled**
* ECS task **task role** with `rds-db:connect` permission
* Database user created with IAM authentication plugin

---

### Create MySQL User for IAM Authentication

Connect to the database:

```bash
mysql -h <DB_ENDPOINT> -u <admin_user> -p
```

Create an IAM-authenticated user:

```sql
CREATE USER 'demoapp' IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';
GRANT ALL PRIVILEGES ON *.* TO 'demoapp'@'%';
FLUSH PRIVILEGES;
```

> This user **does not have a password**. Authentication is done via IAM.

---

### IAM Policy for RDS Access

Attach this policy to the **ECS task role**:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "rds-db:connect",
      "Resource": "arn:aws:rds-db:<REGION>:<ACCOUNT_ID>:dbuser:<DB_RESOURCE_ID>/demoapp"
    }
  ]
}
```

**Replace:**

* `<REGION>` – AWS region
* `<ACCOUNT_ID>` – AWS account ID
* `<DB_RESOURCE_ID>` – RDS DB resource ID (from RDS console)
* `demoapp` – MySQL user created above

---

### ECS Task Configuration

In the **ECS task definition**:

* Assign the IAM role containing the policy above as the **task role**
* Pass DB connection details via environment variables or Secrets Manager:

  * `DB_HOST`
  * `DB_PORT`
  * `DB_DATABASE`
  * `DB_USER=demo_app`

Your application generates an IAM auth token at runtime to connect to RDS.

---

# Monitoring (ECS)

For monitoring ECS workloads, **enable ECS Container Insights** at the cluster level.

Container Insights provides:

* CPU & memory utilization (task + service level)
* Task count and restarts
* Network metrics
* Logs integration with CloudWatch

**How to use:**

* Enable `containerInsights = enabled` on the ECS cluster
* View metrics in **CloudWatch → Container Insights**
* Use **CloudWatch Logs** for application logs
* Use **CloudWatch Alarms** for CPU / memory thresholds

> No Prometheus, Grafana, or agents are required for ECS monitoring.

---
## Readings
* [How to Architect a Node.Js Project](https://dev.to/shadid12/how-to-architect-a-node-js-project-from-ground-up-1n22)

## Contributing

This boilerplate is open to suggestions and contributions, documentation contributions are also welcome! 😊