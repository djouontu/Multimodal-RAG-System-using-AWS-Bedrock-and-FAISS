# **Execution & Architecture Blueprint: Multimodal RAG Recommender System**

This repository provides an end-to-end implementation for an LLM-powered **Multimodal Retrieval-Augmented Generation (RAG) Recommender System**. It integrates textual and visual data ingestion via Amazon S3, dense semantic vector indexing using FAISS, and real-time generative synthesis using Anthropic Claude 3.5 Sonnet and Amazon Titan via AWS Bedrock, all deployed on a lightweight Streamlit dashboard hosted on AWS EC2.

---

## 📋 Table of Contents
1. [System Architecture & Workflow](#1-system-architecture--workflow)
2. [Technical Specifications Matrix](#2-technical-specifications-matrix)
3. [AWS Bedrock Model Access Setup](#3-aws-bedrock-model-access-setup)
4. [Data Architecture & S3 Staging](#4-data-architecture--s3-staging)
5. [Local Environment Workspace Configuration](#5-local-environment-workspace-configuration)
6. [AWS CLI Credentials Setup](#6-aws-cli-credentials-setup)
7. [Streamlit Production Deployment on AWS EC2](#7-streamlit-production-deployment-on-aws-ec2)

---

## 1. System Architecture & Workflow

The pipeline bridges the gap between text queries, visual catalogs, and generative recommendations through an advanced multimodal ingestion and semantic lookup sequence:

                        ┌──────────────────┐
                        │ Raw Multi-Modal  │
                        │  Data / Catalog  │
                        └────────┬─────────┘
                                 │
                  ┌────────────────────────────┐
                  ▼                            ▼
              ┌─────────────────┐    ┌─────────────────┐
              │  Text Contents  │    │ Visual Contents │
              └────────┬────────┘    └────────┬────────┘
                       │                      │
                       ▼                      ▼
            ┌─────────────────────┐   ┌─────────────────────┐
            │ Titan Text Embed V2 │   │ Claude 3.5 Sonnet   │
            │  (Dense Vectors)    │   │ (Visual Descriptors)│
            └──────────┬──────────┘   └──────────┬──────────┘
                       │                         │
                       └──────────────┬──────────┘
                                      ▼
                            ┌────────────────────┐
                            │    FAISS Vector    │
                            │ Database / Storage │
                            └──────────┬─────────┘
                                       │
             ┌───────────────┐         │  Semantic Top-K Match
             │  User Query   ├─────────┼──────────────┐
             └───────────────┘         ▼              ▼
                          ┌──────────────────┐   ┌──────────────────┐
                          │ Matched Text     │   │ Matched Image    │
                          │ Chunks           │   │ Metadata         │
                          └────────┬─────────┘   └────────┬─────────┘
                                   │                      │
                                   └──────────┬───────────┘
                                              ▼
                                  ┌─────────────────────────┐
                                  │   Claude 3.5 Sonnet     │
                                  │  Generative Synthesis   │
                                  └────────────┬────────────┘
                                               ▼
                                 ┌─────────────────────────┐
                                 │   Streamlit UI Output   │
                                 └─────────────────────────┘
            

---

### Pipeline Execution Phases
* **Data Ingestion & Staging:** Structured data tables, raw documentation, and corresponding asset images are stored within an unified **Amazon S3** namespace.
* **Multimodal Feature Engineering:** * Text fields are chunked and converted into high-density vector embeddings via `Amazon Titan Text Embeddings V2`.
  * Graphic and visual elements are evaluated by `Anthropic Claude 3.5 Sonnet` to create textual descriptors grounded directly in the same semantic coordinate space.
  * Embeddings along with payload reference paths are stored in a high-speed **FAISS** matrix index.
* **Semantic Search & Retrieval:** Real-time user queries are embedded on the fly. The system executes a geometric $k$-Nearest Neighbor ($k\text{-NN}$) lookup over the FAISS vector space to capture relevant text blocks and visual metadata matching the user's explicit or contextual intent.
* **Contextual Generation:** The retrieved text and visual payloads are structured into a dynamic prompt and forwarded to **AWS Bedrock**. Claude 3.5 Sonnet synthesizes the data elements to deliver human-readable, context-aware recommendation outputs to the frontend client.

---

## 2. Technical Specifications Matrix

| Module / Layer | Technology Chosen | Purpose / Architectural Benefit |
| :--- | :--- | :--- |
| **Cloud Computing Engine** | AWS EC2 (`t2.small` Ubuntu) | Cost-effective, reliable compute node for hosting application logic and public HTTP/WS traffic. |
| **Object Cloud Storage** | Amazon S3 | Scalable, low-latency repository target for staging text catalogs, structured tables, and image files. |
| **Text Embedding Vectorizer** | Amazon Titan Text Embeddings V2 | Translates textual assets into high-density mathematical vector coordinates for distance calculations. |
| **Multimodal Perception Layer** | Anthropic Claude 3.5 Sonnet | Dual-purpose: Generates descriptive text structures from images and serves as the primary synthesis model. |
| **Vector Storage & Lookup** | FAISS (Facebook AI Similarity Search) | Executes ultra-fast, local dense geometric distance comparisons to extract top-$K$ contexts. |
| **UI Presentation Layer** | Streamlit Framework | Formulates frontend widgets, interactive text controls, and responsive media rendering. |

---

## 3. AWS Bedrock Model Access Setup

### Step 1: Authentication
1. Navigate to the [AWS Management Console](https://aws.amazon.com/console).
2. Sign in with your Identity and Access Management (IAM) user credentials or root account.

### Step 2: Open Bedrock Console
1. In the top global search bar, search for **Bedrock**.
2. Select **Amazon Bedrock** to enter the service console interface.

### Step 3: Model Access Request
1. On the left sidebar menu, scroll to the bottom deployment settings block and choose **Model access**.
2. Click the **Manage model access** button in the top right quadrant of the interface.

### Step 4: Model Matrix Selection
1. Check the activation boxes for the necessary vendor models:
   * **Amazon:** Titan Text Embeddings V2
   * **Anthropic:** Claude 3.5 Sonnet
2. *Note:* Confirm your selected AWS region supports both models (e.g., `us-east-1` N. Virginia). Modify the region configuration dropdown if a model is unavailable.
3. Scroll down and click **Request model access**. The status parameter will reflect a **Granted** token value once deployment is successful.

---

## 4. Data Architecture & S3 Staging

The pipeline parses operational parameters from either a local repository cache folder or scales explicitly into cloud-native object storage containers.

### Provisioning an S3 Bucket
1. Open the **Amazon S3** console view.
2. Select the **Create bucket** action box.
3. Configure target configuration parameters:
   * **Bucket name:** Enter a globally unique identifier conforming to standard DNS rules.
   * **AWS Region:** Select the exact geographical zone matching your configured Bedrock infrastructure.
4. Maintain default values for Bucket Versioning, Server-Side Encryption, and Block Public Access unless organization policy dictates specific overrides.
5. Click **Create bucket** at the base of the control sheet.

### Uploading Assets
1. Open your target bucket from the active S3 workspace listing.
2. Select **Upload** and toggle either **Add files** or **Add folder** to stage data catalogs, system requirements, or image assets.

---

## 5. Local Environment Workspace Configuration

This application workspace requires a base installation of **Python 3.10.4**. Follow the platform instructions below to set up an isolated runtime execution frame.

### Standard Workspace Provisioning

#### Windows Environment (Command Prompt)
```cmd
:: Move execution path to project folder root
cd C:\\path\\to\\your\\project

:: Generate an isolated environment instance folder
python -m venv myenv

:: Activate target environment binaries
myenv\\Scripts\\activate

:: Install required package dependencies
pip install -r requirements.txt

:: To run the streamlit app

`streamlit run llm_app.py`
```
---

## 6. AWS CLI - Credentials Setup

This guide provides step-by-step instructions on how to configure your AWS credentials using the AWS Command Line Interface (AWS CLI). 

### Prerequisites

- **AWS Account:** Ensure you have an active AWS account. You can create one at [AWS Sign-Up](https://aws.amazon.com/).
- **AWS CLI Installed:** You must have AWS CLI installed on your machine. Follow the [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) if it's not already installed.

### Steps to Configure AWS Credentials

### Step 1: Open the Terminal

- Open a terminal window on your computer. This could be Command Prompt, PowerShell, or Terminal, depending on your operating system.

### Step 2: Obtain Your AWS Access Keys
- Navigate to the AWS Management Console.
- Go to IAM (Identity and Access Management).
- Select Users from the sidebar and click on your username.
- Under the Security credentials tab, find Access keys and click Create access key.
- Download the key file or copy the Access Key ID and Secret Access Key. Keep these credentials secure.

### Step 3: Run the AWS Configure Command

- Enter the following command in the terminal:
  ```bash
  aws configure
  ```
  You will be prompted to enter four pieces of information:

AWS Access Key ID: Your unique access key for AWS services.
AWS Secret Access Key: Your secret key for authentication.
Default Region Name: The AWS region you want to use by default (e.g., us-west-2, us-east-1).

---

## 7. Streamlit Deployment on EC2 instance (Optional)

This guide provides step-by-step instructions for deploying a Streamlit application on an AWS EC2 instance. 

### Prerequisites

- AWS Account
- Basic knowledge of AWS EC2, SSH, and Streamlit

### Deployment Steps

### 1. Launching EC2 Instance

- Launch an EC2 instance on AWS with the following specifications:
  - Ubuntu 22.04 LTS
  - Instance Type: t2.small (or your preferred type according to size)
  - Security Group: Allow inbound traffic on port 8501 for Streamlit

- Create and download a PEM key for SSH access to the EC2 instance.

- Disable Inheritance and Restrict Access on PEM key For Windows Users:
    - Locate the downloaded PEM key file (e.g., your-key.pem) using File Explorer.

    - Right-click on the PEM key file and select "Properties."

    - In the "Properties" window, go to the "Security" tab.

    - Click on the "Advanced" button.

    - In the "Advanced Security Settings" window, you'll see an "Inheritance" section. Click on the "Disable inheritance" button.

    - A dialog box will appear; choose the option "Remove all inherited permissions from this object" and click "Convert inherited permissions into explicit permissions on this object."

    - Once inheritance is disabled, you will see a list of users/groups with permissions. Remove permissions for all users except for the user account you are using (typically an administrator account).

    - Click "Apply" and then "OK" to save the changes.


### 2. Accessing EC2 Instance

1. Use the following SSH command to connect to your EC2 instance:
  ```
  ssh -i "your-key.pem" ubuntu@your-ec2-instance-public-ip
  ```

2. Gain superuser access by running: `sudo su`

3. Updating and Verifying Python
  - Update the EC2 instance with the latest packages:
    `apt update`

  - Verify Python installation:
    `python3 --version`

4. Installing Python Packages
`apt install python3-pip`

5. Transferring Files to EC2
    Use SCP to transfer your Streamlit application code to the EC2 instance:

    ```scp -i "your-key.pem" -r path/to/your/app ubuntu@your-ec2-instance-public-ip:/path/to/remote/location```

6. Setting Up Streamlit Application
    Change the working directory to the deployment files location:

    `cd /path/to/remote/location`

    Install dependencies from your requirements file:

    `pip3 install -r requirements.txt`

7. Running the Streamlit Application
    Test your Streamlit application (Use external link):
    `streamlit run app.py`


    For a permanent run, use nohup:
    `nohup streamlit run app.py`

8. Cleanup and Termination
To terminate the nohup process:
  - `sudo su`
  - `ps -df`
  - `kill {process id}`


```
