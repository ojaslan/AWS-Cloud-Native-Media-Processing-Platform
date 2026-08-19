# ☁️ Cloud-Native Video Processing & Media Delivery Platform

A **scalable, highly available, fault-tolerant, secure, cost-optimized, and observable AWS architecture** for video upload, asynchronous processing, metadata management, and global media delivery.

## 🏗️ Architecture

flowchart TB

    %% USERS
    U[Web / Mobile Users]
    ADMIN[Admin]
    GLOBAL[Global End Users]

    %% EDGE
    R53[Amazon Route 53<br/>DNS]
    CF[Amazon CloudFront<br/>CDN]
    WAF[AWS WAF]

    U --> R53
    ADMIN --> R53
    R53 --> CF
    CF --> WAF

    %% AWS VPC
    subgraph AWS[AWS Cloud]
        subgraph VPC[VPC 10.0.0.0/16]

            %% PUBLIC TIER
            subgraph PUBLIC[Public Subnets - Across 3 AZs]
                ALB[Application Load Balancer]
                API[API / Web Application<br/>EC2 Auto Scaling Group]
                REDIS[Amazon ElastiCache<br/>Redis]

                ALB --> API
                API --> REDIS
            end

            %% INGESTION
            subgraph INGEST[Private Subnets - Ingestion Tier - Across 3 AZs]
                UPLOAD[Upload Service<br/>EC2 Auto Scaling]
                RAW[S3<br/>Raw Video Uploads]
                SQS[Amazon SQS<br/>Upload Queue]

                UPLOAD --> RAW
                RAW --> SQS
            end

            %% PROCESSING
            subgraph PROCESS[Private Subnets - Processing Tier - Across 3 AZs]
                WORKER[Video Processing Workers<br/>EC2 Auto Scaling Group]
                EFS[Amazon EFS<br/>Shared Storage]
                PROCESSED[S3<br/>Processed Videos]

                WORKER --> EFS
                WORKER --> PROCESSED
            end

            %% DATABASE
            subgraph DATA[Private Subnets - Data Tier - Across 3 AZs]
                RDS[Amazon RDS<br/>PostgreSQL Multi-AZ]
                DDB[Amazon DynamoDB<br/>Job Metadata / Status]
                MEDIA[S3<br/>Thumbnails / Subtitles / Transcripts]
            end

            %% INTERNAL FLOW
            API --> UPLOAD
            SQS --> WORKER
            WORKER --> DDB
            WORKER --> RDS
            PROCESSED --> MEDIA

        end

        %% NAT / INTERNET
        NAT[NAT Gateway]
        IGW[Internet Gateway]

        VPC --> NAT
        NAT --> IGW
    end

    %% DELIVERY
    CF --> ALB
    CF --> MEDIA
    MEDIA --> GLOBAL

    %% STORAGE LIFECYCLE
    RAW -. Lifecycle Policy .-> ARCHIVE[S3 Glacier / IA]

    %% MONITORING
    MON[Amazon CloudWatch<br/>Logs / Metrics / Alarms]

    API -.-> MON
    WORKER -.-> MON
    ALB -.-> MON
    RDS -.-> MON

## 📌 Project Overview

This project demonstrates how to design a **cloud-native video processing and media delivery platform on AWS** using a decoupled, event-driven architecture.

The platform separates the application into multiple tiers:

* **Application Tier** – Stateless APIs and web application
* **Ingestion Tier** – Secure video upload and queue-based processing
* **Processing Tier** – Scalable video processing workers
* **Data Tier** – Relational and NoSQL metadata storage
* **Delivery Tier** – Global content delivery through CloudFront

## 🔄 Request & Processing Flow

```text
Users
  ↓
Route 53
  ↓
CloudFront
  ↓
AWS WAF
  ↓
Application Load Balancer
  ↓
EC2 Auto Scaling Group
  ↓
Upload Service
  ↓
Amazon S3
  ↓
Amazon SQS
  ↓
EC2 Worker Auto Scaling Group
  ↓
Amazon EFS / S3
  ↓
Processed Videos
  ↓
Amazon CloudFront
  ↓
Global End Users
```

## ☁️ AWS Services Used

| Service                      | Purpose                               |
| ---------------------------- | ------------------------------------- |
| Amazon Route 53              | DNS and domain routing                |
| Amazon CloudFront            | Global video/content delivery         |
| AWS WAF                      | Web application security              |
| Application Load Balancer    | Traffic distribution                  |
| Amazon EC2                   | API and video processing workloads    |
| EC2 Auto Scaling             | Horizontal scalability                |
| Amazon S3                    | Raw and processed video storage       |
| Amazon SQS                   | Asynchronous processing queue         |
| Amazon EFS                   | Shared storage for processing workers |
| Amazon ElastiCache for Redis | Caching and fast application access   |
| Amazon RDS PostgreSQL        | Relational application data           |
| Amazon DynamoDB              | Job metadata and processing status    |
| NAT Gateway                  | Private subnet outbound connectivity  |
| Internet Gateway             | VPC internet connectivity             |

## 🏛️ Architecture Layers

### 1. Application Tier

The Application Load Balancer distributes incoming requests across **stateless EC2 instances running in an Auto Scaling Group across multiple Availability Zones**.

Redis is used for caching frequently accessed application data.

### 2. Ingestion Tier

The upload service generates **pre-signed S3 URLs**, allowing clients to upload videos directly to Amazon S3.

Amazon SQS decouples uploads from video processing and prevents processing workloads from being directly dependent on user requests.

### 3. Processing Tier

Video processing workers run on EC2 instances managed by an **Auto Scaling Group**.

Workers consume jobs from SQS and process videos using shared storage through Amazon EFS.

Processed videos are stored in Amazon S3.

### 4. Data Tier

Different storage systems are used based on workload requirements:

* **Amazon RDS PostgreSQL** – Application and relational data
* **Amazon DynamoDB** – Job metadata and processing status
* **Amazon S3** – Videos, thumbnails, subtitles, and transcripts
* **Amazon ElastiCache Redis** – Frequently accessed/cacheable data

### 5. Media Delivery

Processed videos are delivered globally through **Amazon CloudFront**, reducing latency and improving the user experience.

S3 lifecycle policies can transition older media to **Amazon S3 Glacier** for cost optimization.

## 🔐 Security

The architecture follows a defense-in-depth approach:

* AWS WAF for application-layer protection
* Private subnets for processing and database workloads
* Security groups for network-level access control
* S3-based object storage
* Pre-signed URLs for controlled uploads
* NAT Gateway for controlled outbound access from private subnets
* Multi-AZ database architecture
* Separation of application, processing, and data tiers

## 📈 Scalability & High Availability

The architecture is designed for horizontal scalability and fault tolerance:

* Multi-AZ deployment across **3 Availability Zones**
* EC2 Auto Scaling Groups
* Application Load Balancer
* Queue-based asynchronous processing
* Amazon S3 highly durable object storage
* DynamoDB for scalable job metadata
* RDS PostgreSQL Multi-AZ
* CloudFront global edge delivery

## 💰 Cost Optimization

Cost optimization strategies include:

* EC2 Auto Scaling based on workload
* S3 lifecycle policies
* Transitioning older media to Glacier
* CloudFront caching
* Redis caching to reduce database load
* Asynchronous processing using SQS
* Using the appropriate AWS storage service for each workload

## 🎯 Key Design Principles

* **Scalable**
* **Highly Available**
* **Fault Tolerant**
* **Secure**
* **Event Driven**
* **Decoupled**
* **Observable**
* **Cost Optimized**
* **Multi-AZ**
* **Globally Distributed**

## 🛠️ Suggested GitHub Repository Structure

```text
cloud-native-video-platform/
│
├── README.md
│
├── architecture.png
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── modules/
│
└── docs/
    └── architecture-notes.md
```

> **Note:** This repository represents an AWS architecture/design project. Terraform and implementation files can be added separately to demonstrate infrastructure-as-code deployment.

## 👨‍💻 Skills Demonstrated

**AWS | Cloud Architecture | VPC | EC2 | Auto Scaling | ALB | S3 | SQS | RDS | DynamoDB | ElastiCache | EFS | CloudFront | Route 53 | WAF | High Availability | Scalability | Fault Tolerance | Cost Optimization | Infrastructure as Code**
