# DevOps-09-AWS-java-maven-app

This project demonstrates an automated **CI/CD Multibranch Pipeline** implemented with **Jenkins**.

The pipeline automates the following steps:

* Building and testing a Java Maven application
* Building a Docker image
* Publishing the Docker image to Docker Hub
* Deploying the application to an AWS EC2 instance
* Running the application inside a Docker container

## 1. CI/CD Workflow

```text
GitHub
   │
   │ Push / Pull Request
   ▼
Jenkins Multibranch Pipeline
   │
   ├── Build & Test
   │
   ├── Build Docker Image
   │
   ├── Push Image to Docker Hub
   │
   └── Deploy to AWS EC2
              │
              ▼
        Docker Container
              │
              ▼
      Java Maven Application
```

---

# 2. AWS

## 2.1 Create an EC2 Instance

Go to:

**AWS Console → EC2 → Launch Instance**

### 2.1.1 Configure the EC2 Instance

#### Tags

Add the following tags:

| Key    | Value                    |
| ------ | ------------------------ |
| `Name` | `my-instance`            |
| `Type` | `web-server-with-docker` |

#### Application and OS Images

* **AMI:** Amazon Linux
* **Instance type:** `t3.micro`

### 2.1.2 Key Pair

Create a new key pair:

* **Name:** `docker-server`
* **Key pair type:** RSA
* **Private key file format:** `.pem`

After creating the key pair, the `docker-server.pem` file is downloaded.

### 2.1.3 Network Settings

Select **Edit** and configure:

* **VPC:** Default VPC
* **Subnet:** No preference
* **Auto-assign public IP:** Enable
* **Firewall:** Create security group
* **Security group name:** `security-group-docker-server`

Configure SSH access:

* **Type:** SSH
* **Protocol:** TCP
* **Port:** `22`
* **Source:** My IP

Finally, click **Launch instance**.

---

## 2.2 Connect to the EC2 Instance

Set the correct permissions on the private key:

```bash
chmod 400 ~/Downloads/docker-server.pem
```

Move the key to the SSH directory:

```bash
mv ~/Downloads/docker-server.pem ~/.ssh/
```

Connect to the EC2 instance:

```bash
ssh -i ~/.ssh/docker-server.pem ec2-user@<EC2_PUBLIC_IP>
```

---

## 2.3 Install Docker on the EC2 Instance

Update the system and install Docker:

```bash
sudo yum update
sudo yum install docker
```

Start the Docker service:

```bash
sudo service docker start
```

Add the current user to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

Log out:

```bash
exit
```

Then reconnect:

```bash
ssh -i ~/.ssh/docker-server.pem ec2-user@<EC2_PUBLIC_IP>
```

Verify that Docker works without `sudo`:

```bash
docker run redis
```

---

## 2.4 Configure the EC2 Security Group

Go to:

**EC2 → Security Groups → `security-group-docker-server` → Edit inbound rules**

### 2.4.1 Application Port

Allow incoming traffic on port `8080`:

| Type       | Protocol | Port   | Source      |
| ---------- | -------- | ------ | ----------- |
| Custom TCP | TCP      | `8080` | `0.0.0.0/0` |

### 2.4.2 Jenkins SSH Access

Allow Jenkins to connect to the EC2 instance through SSH:

| Type | Protocol | Port | Source                |
| ---- | -------- | ---- | --------------------- |
| SSH  | TCP      | `22` | `<JENKINS_PUBLIC_IP>` |

Save the inbound rules.

---

# 3. Jenkins

## 3.1 Create the AWS Multibranch Pipeline

Create a Jenkins **Multibranch Pipeline** named:

```text
aws-multibranch-pipeline
```

Configure it to use this GitHub repository.

A Multibranch Pipeline automatically discovers branches containing a `Jenkinsfile` and creates a separate pipeline job for each branch.

---

## 3.2 Install the SSH Agent Plugin

Go to:

**Manage Jenkins → Plugins → Available plugins**

Search for:

```text
SSH Agent
```

Install the **SSH Agent Plugin**.

---

## 3.3 Create SSH Credentials

Go to:

**aws-multibranch-pipeline → Credentials → Stores scoped to aws-multibranch-pipeline → aws-multibranch-pipeline → Global credentials → Add Credentials**

Configure:

* **Kind:** SSH Username with private key
* **ID:** `ec2-server-key`
* **Username:** `ec2-user`
* **Private Key:** Enter directly
* Paste the contents of `docker-server.pem`

Click **Create**.

---

## 3.4 Run the Jenkins Pipeline

Go to:

**aws-multibranch-pipeline → jenkins-jobs → Build Now**

The pipeline will:

1. Build the Java application with Maven
2. Run the tests
3. Build the Docker image
4. Push the Docker image to Docker Hub
5. Connect to the EC2 instance through SSH
6. Pull the Docker image
7. Start the application container

---

# 4. Verify the Deployment

Connect to the EC2 instance:

```bash
ssh -i ~/.ssh/docker-server.pem ec2-user@<EC2_PUBLIC_IP>
```

## 4.1 Check the Docker Images

```bash
docker images
```

## 4.2 Check the Running Containers

```bash
docker ps
```

If the deployment was successful, the application should be accessible at:

```text
http://<EC2_PUBLIC_IP>:8080/
```

---

# Acknowledgements

This demo project was created as part of the DevOps Bootcamp by **TechWorld with Nana**.

Many thanks to Nana for creating such a comprehensive and practical learning experience.
