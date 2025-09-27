# Java App CI/CD Pipeline on AWS 🚀

This project demonstrates how to set up a **CI/CD pipeline on AWS** for a simple Java web application.  
We’ll use AWS services like **CodePipeline, CodeBuild, CodeDeploy, EC2, S3, IAM, CloudFormation, CodeArtifact, and CloudWatch** to automate deployments from GitHub to an EC2 instance.  

---

## 📌 Architecture

**Flow:**  
`GitHub (Source Code)` → `AWS CodePipeline` → `CodeBuild` → `CodeDeploy` → `EC2 (Web Server)`

Supporting services:  
- **S3** → stores build artifacts  
- **IAM** → manages roles & permissions  
- **CodeArtifact** → Maven dependency repository  
- **CloudFormation** → infrastructure provisioning  
- **CloudWatch** → monitoring & logs  

---

## ✅ Prerequisites

- AWS account with permissions for IAM, EC2, S3, CodePipeline, CodeBuild, CodeDeploy  
- GitHub repository (this repo: `java-app-cicd-nextwork`)  
- Installed tools:  
  - [Git](https://git-scm.com/downloads)  
  - [Apache Maven](https://maven.apache.org/download.cgi)  
  - [Amazon Corretto 8 (Java 8)](https://docs.aws.amazon.com/corretto/)  
  - [AWS CLI](https://aws.amazon.com/cli/)  
  - [VSCode](https://code.visualstudio.com/) (optional, for SSH editing)  

---

## 🔹 Step 1: Provision Infrastructure

Use the CloudFormation template included in this repo:

```bash
aws cloudformation create-stack \
  --stack-name nextwork-webapp \
  --template-body file://nextworkwebapp.yaml \
  --parameters ParameterKey=MyIP,ParameterValue=<your-ip>/32 \
  --capabilities CAPABILITY_NAMED_IAM
````

This will create:

* VPC + Subnet + Internet Gateway + Route Table
* Security group (HTTP access allowed)
* EC2 instance (`t3.micro`) with IAM instance profile
* Output: **public EC2 URL** for your web app

---

## 🔹 Step 2: Set Up AWS Services

1. **CodeArtifact**

   * Create a CodeArtifact repo with Maven as the upstream.
   * Connect EC2 to CodeArtifact following AWS instructions.

2. **S3**

   * Create a bucket to store pipeline artifacts.

3. **IAM Roles** (with policies attached):

   * **EC2 role** → access to CodeArtifact + STS
   * **CodeBuild role** → access to CodeArtifact, S3, CloudWatch Logs, CodeConnections
   * **CodeDeploy role** → access to EC2, AutoScaling, ELB, CloudWatch, SNS
   * **CodePipeline role** → access to CodeBuild, CodeDeploy, S3, CodeConnections

---

## 🔹 Step 3: Configure CI/CD Pipeline

1. **Source stage**

   * Connect to this GitHub repo (`master` branch).
   * Use CodeConnections (created with your GitHub PAT).

2. **Build stage**

   * AWS CodeBuild runs Maven build with `buildspec.yml`.
   * Artifact is packaged into `.zip` and stored in S3.

   **Maven commands used in buildspec:**

   ```bash
   mvn clean
   mvn install
   mvn package
   ```

3. **Deploy stage**

   * CodeDeploy uses `appspec.yml` + `scripts/` to deploy build artifact onto EC2 instance.

4. **Execution Mode**

   * Set to **Superseded** → new runs replace older ones.

---

## 🔹 Step 4: First Run & Testing

1. Push your code to the **master branch**:

   ```bash
   git add .
   git commit -m "Initial pipeline test"
   git push origin master
   ```

2. CodePipeline triggers automatically (Source → Build → Deploy).

3. Once complete, visit the **EC2 public IP** (from CloudFormation output) in your browser.
   You should see your deployed web page 🎉

---

## 🔹 Step 5: Rollback & Monitoring

* **Rollback** → CodeDeploy can revert to the **last successful deployment** if the new one fails.
* **Monitoring** → CloudWatch collects build logs, deployment logs, and EC2 metrics.

---

## 🔹 Step 6: Cleanup (to avoid AWS charges)

Run these steps after testing:

1. Delete the CloudFormation stack:

   ```bash
   aws cloudformation delete-stack --stack-name nextwork-webapp
   ```
2. Delete CodePipeline, CodeBuild project, and CodeDeploy app.
3. Empty and delete the S3 bucket.
4. Delete CodeArtifact repo.
5. Remove IAM roles and policies created for this project.

---

## 🔹 Running Locally (Optional)

You can also run the app locally without AWS:

```bash
# Clone the repo
git clone https://github.com/nuelStarkOps/java-app-cicd-nextwork.git
cd java-app-cicd-nextwork

# Build the project
mvn clean install

# Package into a .war file
mvn package

# Deploy manually to Tomcat (if installed locally)
cp target/*.war /path/to/tomcat/webapps/
```

---

## 🎯 Conclusion

In this project, we:

* Built a full **CI/CD pipeline** on AWS using CodePipeline, CodeBuild, and CodeDeploy
* Automated deployments from **GitHub → EC2**
* Managed infra with **CloudFormation** (`nextworkwebapp.yaml`)
* Configured IAM roles, rollbacks, and monitoring with CloudWatch

This pipeline ensures **consistent, reliable, and automated deployments** — a core DevOps skill.