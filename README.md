# 🚀 EKS-TERRAFORM

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![EKS](https://img.shields.io/badge/Amazon%20EKS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)

A Terraform-based Infrastructure as Code (IaC) project to provision and manage an **Amazon EKS (Elastic Kubernetes Service)** cluster on AWS, automated via a **Jenkins CI/CD pipeline**.

---

## 📁 Repository Structure

```
EKS-TERRAFORM/
├── backend.tf        # Terraform remote backend configuration (S3)
├── main.tf           # EKS cluster and IAM roles configuration
├── provider.tf       # AWS provider configuration
└── README.md         # Project documentation
```

---

## 📋 File Overview

### `provider.tf`
Configures the AWS provider for Terraform, including the target region and required provider version.

### `backend.tf`
Sets up the Terraform remote backend using **Amazon S3** to store the `terraform.tfstate` file remotely, enabling team collaboration and state locking.

### `main.tf`
Contains the core infrastructure definitions:
- **Amazon EKS Cluster** — control plane configuration
- **IAM Roles & Policies** — required permissions for the EKS cluster and worker nodes

---

## ⚙️ Prerequisites

Before you begin, ensure you have the following installed and configured:

| Tool | Version | Purpose |
|------|---------|---------|
| [Terraform](https://developer.hashicorp.com/terraform/downloads) | >= 1.0.0 | Infrastructure provisioning |
| [AWS CLI](https://aws.amazon.com/cli/) | >= 2.0 | AWS authentication |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | Latest | Kubernetes cluster management |
| [Jenkins](https://www.jenkins.io/) | Latest | CI/CD pipeline automation |

### AWS Permissions Required
Your AWS IAM user/role must have permissions for:
- `eks:*`
- `iam:*`
- `ec2:*`
- `s3:*` (for backend state)

---

## 🚀 Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/ganeshsnp987/EKS-TERRAFORM.git
cd EKS-TERRAFORM
```

### 2. Configure AWS Credentials

```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and Region
```

### 3. Initialize Terraform

```bash
terraform init
```

### 4. Validate Configuration

```bash
terraform validate
```

### 5. Preview the Infrastructure Plan

```bash
terraform plan
```

### 6. Apply / Destroy Infrastructure

```bash
# To create the EKS cluster
terraform apply --auto-approve

# To destroy the EKS cluster
terraform destroy --auto-approve
```

---

## 🔧 Jenkins CI/CD Pipeline

This project includes a Jenkins pipeline that automates the full Terraform lifecycle.

### Jenkinsfile Stages

| Stage | Description |
|-------|-------------|
| `Checkout from Git` | Clones this repository from the `main` branch |
| `Terraform version` | Verifies Terraform is installed on the Jenkins agent |
| `Terraform init` | Initializes the Terraform working directory |
| `Terraform validate` | Validates the Terraform configuration files |
| `Terraform plan` | Generates and displays an execution plan |
| `Terraform apply/destroy` | Applies or destroys infrastructure based on the `action` parameter |

### Pipeline Setup

1. Create a new **Pipeline** job in Jenkins.
2. Add a **String Parameter** named `action` with allowed values: `apply` or `destroy`.
3. Paste the following Jenkinsfile or point to the repo's `Jenkinsfile`:

```groovy
pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: ['apply', 'destroy'], description: 'Terraform Action')
    }

    environment {
        AWS_CREDENTIALS = credentials('aws_cred')  // Injects AWS_CREDENTIALS_ACCESS_KEY_ID
                                                   // and AWS_CREDENTIALS_SECRET_ACCESS_KEY
    }

    stages {
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ganeshsnp987/EKS-TERRAFORM.git'
            }
        }

        stage('Terraform Version') {
            steps {
                sh 'terraform --version'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan'
            }
        }

        stage('Terraform Apply / Destroy') {
            steps {
                sh "terraform ${params.action} --auto-approve"
            }
        }
    }

    post {
        success {
            echo "✅ Terraform ${params.action} completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check the logs above."
        }
        always {
            cleanWs()  // Clean workspace after every run
        }
    }
}

```

### Running the Pipeline

- Select **Build with Parameters** in Jenkins.
- Set `action` to `apply` to create the EKS cluster.
- Set `action` to `destroy` to tear it down.

---

## 🔐 Storing Secrets in Jenkins

To securely pass AWS credentials to your pipeline, add the following as **Jenkins Credentials** (Secret Text or AWS Credentials plugin):

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_DEFAULT_REGION`

Then inject them in your Jenkinsfile using the `withCredentials` block:

```groovy
withCredentials([[
    $class: 'AmazonWebServicesCredentialsBinding',
    credentialsId: 'aws-credentials'
]]) {
    sh 'terraform apply --auto-approve'
}
```

---

## 🌐 Accessing the EKS Cluster

After a successful `apply`, configure `kubectl` to connect to your cluster:

```bash
aws eks update-kubeconfig --region <your-region> --name <your-cluster-name>
kubectl get nodes
```

---

## 🧹 Cleanup

To avoid unnecessary AWS charges, destroy the infrastructure when not in use:

```bash
terraform destroy --auto-approve
```

Or trigger the Jenkins pipeline with `action = destroy`.

---

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 👤 Author

**ganeshsnp987**
- GitHub: [@ganeshsnp987](https://github.com/ganeshsnp987)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
