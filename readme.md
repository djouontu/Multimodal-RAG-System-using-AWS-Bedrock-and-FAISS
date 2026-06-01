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
