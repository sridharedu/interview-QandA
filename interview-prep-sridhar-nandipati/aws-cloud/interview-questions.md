# AWS Cloud Interview Questions for Senior Engineers

## Question 1: Designing a Highly Available & Scalable Web Application on AWS

**Question:** You need to design the AWS architecture for a new e-commerce web application that is expected to handle significant traffic spikes during holiday seasons. The application consists of a frontend, backend APIs (Java/Spring Boot microservices), and a relational database (PostgreSQL). Detail the AWS services you would use for compute, database, networking, and content delivery, focusing on high availability, scalability, and security.

**Discussion Points / Sample Answer Outline:**

*   **Overall Architecture Vision:** Multi-AZ deployment.
*   **Networking (VPC):**
    *   VPC structure: Public and private subnets across multiple AZs.
    *   Internet Gateway for public subnets, NAT Gateways for outbound traffic from private subnets.
    *   Security Groups and NACLs: How would you configure them for different layers (ALB, App, DB)?
        *   *Refer to your experience at Herc Rentals/Intralinks with VPC design.*
*   **Compute (Application Tier):**
    *   Backend APIs: Amazon ECS with Fargate or EC2 Auto Scaling Groups for your Spring Boot microservices. Discuss trade-offs.
        *   *Link to your microservices and Java/Spring Boot experience on AWS.*
    *   Frontend: S3 for static content, CloudFront for global distribution and caching. Or, if it's an SSR app, EC2/ECS as well.
*   **Database (Data Tier):**
    *   Amazon RDS for PostgreSQL.
    *   Multi-AZ deployment for high availability.
    *   Read Replicas for scaling read traffic.
    *   Considerations for connection pooling from your Java applications.
        *   *Relate to your RDS experience (Herc Admin Tool, Intralinks).*
*   **Load Balancing & Content Delivery:**
    *   Application Load Balancer (ALB) for distributing traffic to backend services.
    *   Amazon CloudFront for caching static/dynamic content and SSL termination.
        *   *Your AEM/CloudFront experience at Herc Rentals is relevant here.*
    *   Amazon Route 53 for DNS management, potentially with health checks and failover policies.
*   **Scalability:**
    *   Auto Scaling for EC2/ECS services.
    *   Read replicas for RDS.
    *   CloudFront caching.
    *   Consider ElastiCache for Redis for caching frequently accessed data or session state.
        *   *Mention your Redis experience.*
*   **Security:**
    *   IAM roles for EC2 instances, ECS tasks.
    *   AWS WAF with ALB/CloudFront.
    *   KMS for encryption (EBS, S3, RDS).
    *   Security Groups/NACLs.
    *   Private subnets for application and database tiers.
*   **Monitoring & Logging:**
    *   CloudWatch Metrics, Alarms, Logs.
    *   CloudTrail for auditing.
    *   *How would your ELK/Prometheus/Grafana experience complement CloudWatch?*
*   **Deployment (CI/CD):**
    *   Briefly mention how you'd use services like AWS CodeDeploy, Jenkins, or CloudFormation/CDK.
        *   *Reference your CI/CD and IaC experience.*
*   **Cost Optimization:** Mention instance selection, S3 lifecycle policies, reserved instances/savings plans.

## Question 2: Migrating On-Premise Applications to AWS

**Question:** A client wants to migrate a traditional three-tier on-premise application (web server, Java application server, Oracle database) to AWS. Outline a migration strategy, including key AWS services you'd recommend for each tier and how you'd handle data migration. Discuss challenges and considerations.

**Discussion Points / Sample Answer Outline:**

*   **Migration Strategy (The "R"s):**
    *   Rehost ("Lift and Shift"): e.g., EC2 for web/app servers, RDS for Oracle (or Oracle on EC2).
    *   Replatform ("Lift and Reshape"): e.g., Move app servers to ECS/EKS, use RDS.
    *   Refactor/Rearchitect: e.g., Break monolith into microservices (Spring Boot on ECS/Lambda), use managed services like DynamoDB or Aurora PostgreSQL.
    *   *Discuss which strategy might be suitable initially and long-term, based on your experience.*
*   **AWS Services for Each Tier:**
    *   **Web Tier:** EC2 Auto Scaling, ALB. Or S3/CloudFront if static.
    *   **Application Tier (Java):**
        *   EC2 with Auto Scaling (if rehosting).
        *   Amazon ECS or EKS for containerized Java applications.
            *   *Your Java, Spring Boot, Microservices, Docker experience is key.*
        *   Consider AWS Elastic Beanstalk for simpler deployments.
    *   **Database Tier (Oracle):**
        *   Amazon RDS for Oracle (if license allows and feature set matches).
        *   Oracle on EC2 (more control, but more management).
        *   AWS Database Migration Service (DMS) for migration.
        *   Long-term: Consider migrating to Aurora PostgreSQL or MySQL for cost/flexibility.
            *   *Your experience with Oracle and PostgreSQL is relevant.*
*   **Data Migration:**
    *   AWS DMS: For migrating Oracle data to RDS Oracle or other engines. Continuous replication for minimal downtime.
    *   Native database tools (Oracle Data Pump, RMAN) for backup/restore to EC2 or RDS.
    *   Storage Gateway for transferring backups to S3.
*   **Networking:**
    *   VPC design, VPN/Direct Connect for hybrid connectivity during migration.
    *   Route 53 for DNS cutover.
*   **Security:**
    *   Replicating on-premise security model in AWS (Security Groups, NACLs, IAM).
*   **Challenges & Considerations:**
    *   Application dependencies and compatibility.
    *   Performance testing in AWS.
    *   Cost analysis and optimization.
    *   Downtime minimization during cutover.
    *   Team skills and training.
    *   Licensing (especially for Oracle).
*   **Post-Migration:** Monitoring (CloudWatch), optimization, potential refactoring.

## Question 3: Serverless Architecture for a Data Processing Pipeline

**Question:** Design a serverless data processing pipeline on AWS. Data arrives in an S3 bucket, needs to be transformed (e.g., data type conversion, filtering, enrichment by calling an external API), and then loaded into DynamoDB for quick lookups and S3 for archival/analytics.

**Discussion Points / Sample Answer Outline:**

*   **Triggering Mechanism:**
    *   S3 Event Notification on `s3:ObjectCreated:*` events.
*   **Core Processing Logic (AWS Lambda):**
    *   Lambda function triggered by S3 event.
    *   Permissions: IAM role for Lambda to read from source S3 bucket, write to target DynamoDB table and S3 bucket, and call external API.
    *   Runtime: Java, Python, Node.js (mention your preference if any).
        *   *Your Java experience for Lambda functions.*
    *   Error Handling: Dead Letter Queue (DLQ) for Lambda (e.g., an SQS queue) for failed processing.
    *   Memory/Timeout configuration for Lambda.
*   **Transformation Steps within Lambda:**
    *   Downloading object from S3.
    *   Parsing data (CSV, JSON, etc.).
    *   Data type conversions, filtering records.
    *   Enrichment: Making an HTTP call to an external API (handle timeouts, retries).
        *   *Consider using AWS Secrets Manager for API keys.*
*   **Loading Data:**
    *   **DynamoDB:**
        *   BatchWriteItem for efficient writes.
        *   Data modeling for DynamoDB table (primary key, sort key, any GSIs).
        *   *Relate to your DynamoDB/NoSQL experience.*
    *   **S3 (Archival/Analytics):**
        *   Store transformed data, perhaps in a different format (e.g., Parquet) or different prefix.
        *   Consider S3 lifecycle policies for long-term storage (Glacier).
*   **Scalability & Concurrency:**
    *   Lambda's automatic scaling. Understand concurrency limits.
    *   DynamoDB auto-scaling or provisioned capacity.
*   **Monitoring & Logging:**
    *   CloudWatch Logs for Lambda logs.
    *   CloudWatch Metrics for Lambda invocations, errors, duration.
    *   CloudTrail for API calls.
*   **Workflow Orchestration (if complex):**
    *   AWS Step Functions if the processing involves multiple stages, retries, or complex branching logic beyond a single Lambda.
*   **Security:**
    *   IAM roles with least privilege.
    *   Encryption for S3 (SSE-S3, SSE-KMS) and DynamoDB.
*   **Cost Considerations:** Lambda (invocations, duration), S3 (storage, requests), DynamoDB (capacity, storage), API Gateway (if used for external API), Data Transfer.

## Question 4: Cost Optimization Strategies on AWS

**Question:** You've noticed that the AWS bill for a project (e.g., at Herc Rentals or Intralinks) has been increasing significantly. Describe the process and tools you would use to identify areas for cost optimization and suggest specific strategies across services like EC2, S3, RDS, and data transfer.

**Discussion Points / Sample Answer Outline:**

*   **Analysis & Identification Tools:**
    *   **AWS Cost Explorer:** Analyze cost and usage trends, filter by service, tags, account.
    *   **AWS Budgets:** Set custom budgets and get alerts.
    *   **AWS Trusted Advisor:** Provides recommendations for cost savings (e.g., idle resources, underutilized EC2).
    *   **Cost and Usage Reports (CUR):** Detailed raw data for deep analysis (can be queried with Athena).
    *   Tagging strategy: Ensure resources are tagged appropriately for cost allocation.
*   **EC2 Cost Optimization:**
    *   **Right-sizing instances:** Analyze CloudWatch metrics (CPU, memory utilization) to identify over-provisioned instances and downsize them.
    *   **Pricing Models:**
        *   Reserved Instances (RIs) or Savings Plans for steady-state workloads.
        *   Spot Instances for fault-tolerant workloads (batch processing, dev/test).
            *   *Any experience using Spot at BOFA or other projects?*
    *   **Auto Scaling:** Ensure configurations are optimized to scale down aggressively during low demand.
    *   **Scheduling:** Shut down non-production instances (dev/test) during off-hours.
*   **S3 Cost Optimization:**
    *   **Storage Classes:** Use S3 Intelligent-Tiering or implement lifecycle policies to move data to cheaper tiers (Standard-IA, One Zone-IA, Glacier, Glacier Deep Archive).
        *   *Recall your S3 usage at TOPS or Herc Rentals.*
    *   **Delete incomplete multipart uploads.**
    *   **Analyze S3 requests:** Identify and reduce unnecessary GET/PUT requests.
*   **RDS Cost Optimization:**
    *   **Right-sizing instances:** Similar to EC2, monitor CloudWatch metrics.
    *   **Reserved Instances:** For production databases.
    *   **Storage Auto Scaling:** If applicable, or provision appropriately.
    *   **Snapshot retention policies:** Avoid keeping excessive old snapshots.
    *   Consider Aurora Serverless if workload is intermittent.
*   **Data Transfer Costs:**
    *   Analyze data transfer out to the internet (often the most expensive).
    *   Use CloudFront CDN to reduce data transfer out for web content.
    *   Optimize inter-AZ data transfer (e.g., by placing instances needing high bandwidth in the same AZ if HA allows, or using services like VPC Endpoints).
    *   Data transfer within the same region between EC2 and S3/RDS is often free or cheaper.
*   **Lambda Optimization:**
    *   Right-size memory (cost is tied to memory and duration).
    *   Optimize code for faster execution.
    *   Use Provisioned Concurrency judiciously if cold starts are an issue.
*   **Other Services:**
    *   DynamoDB: On-demand vs. provisioned capacity.
    *   ElastiCache: Right-sizing nodes, RIs.
*   **Process & Governance:**
    *   Regular cost reviews.
    *   Implement tagging policies.
    *   Educate development teams on cost-aware architecture.

## Question 5: Security Best Practices for an AWS Environment

**Question:** Describe key security best practices you would implement when setting up and managing an AWS environment for a project handling sensitive data (e.g., Intralinks VDRPro). Cover aspects like identity and access management, network security, data protection, and logging/monitoring.

**Discussion Points / Sample Answer Outline:**

*   **IAM (Identity and Access Management):**
    *   **Principle of Least Privilege:** Grant only necessary permissions.
    *   **IAM Roles:** For EC2 instances, ECS tasks, Lambda functions, instead of storing access keys on instances.
    *   **Users and Groups:** Avoid using the root user for daily tasks. Create IAM users and organize them into groups with specific policies.
    *   **Strong Password Policies & MFA:** Enforce for all IAM users.
    *   Regularly review IAM policies and user access.
*   **Network Security (VPC):**
    *   **Private Subnets:** Deploy application servers and databases in private subnets.
    *   **Security Groups:** Act as stateful firewalls. Restrict inbound traffic to only necessary ports and sources (e.g., app SG allows traffic from ALB SG, DB SG allows traffic from app SG). Deny all by default.
    *   **NACLs:** Stateless firewalls at the subnet level for broader rules (use sparingly, primarily for deny rules).
    *   **VPC Endpoints:** For private access to AWS services (S3, DynamoDB) without traversing the internet.
    *   **AWS WAF:** Protect web applications (on ALB, API Gateway, CloudFront) from common exploits.
        *   *Your experience with WAF for AEM at Herc Rentals.*
*   **Data Protection:**
    *   **Encryption at Rest:**
        *   S3: Server-Side Encryption (SSE-S3, SSE-KMS, SSE-C).
        *   EBS: Encrypt volumes using KMS.
        *   RDS: Enable encryption using KMS.
        *   DynamoDB: Enable encryption at rest.
    *   **Encryption in Transit:**
        *   Use TLS/SSL for all data transfer (e.g., configure ALBs/CloudFront with ACM certificates).
        *   Enforce HTTPS.
    *   **AWS KMS:** Manage encryption keys. Control key policies.
    *   **Data Backup and Replication:** Regular backups (EBS snapshots, RDS snapshots), cross-region replication for DR.
*   **Logging & Monitoring for Security:**
    *   **AWS CloudTrail:** Enable across all regions. Monitor API calls for suspicious activity. Integrate with CloudWatch Logs.
    *   **Amazon CloudWatch:**
        *   Monitor critical metrics and set alarms (e.g., unauthorized API calls, security group changes).
        *   CloudWatch Logs for collecting application and system logs. Use filter patterns for security events.
    *   **VPC Flow Logs:** Capture information about IP traffic going to and from network interfaces in your VPC.
    *   **AWS Config:** Track resource configuration changes and assess compliance against rules.
    *   **Amazon GuardDuty:** Threat detection service that continuously monitors for malicious activity and unauthorized behavior.
    *   **AWS Security Hub:** Centralized view of security alerts and compliance status.
*   **Infrastructure Security:**
    *   Regularly patch OS and application dependencies on EC2 instances.
    *   Use Amazon Inspector for vulnerability assessments.
*   **Incident Response Plan:** Have a plan for responding to security incidents.

## Question 6: Choosing between ECS and EKS for Container Orchestration

**Question:** Your team is developing new microservices using Docker and Java/Spring Boot. You need to decide whether to deploy them on Amazon ECS (with Fargate or EC2) or Amazon EKS. What factors would you consider to make this decision? What are the pros and cons of each in different scenarios?

**Discussion Points / Sample Answer Outline:**

*   **Team's Existing Expertise:**
    *   **Kubernetes (EKS):** If the team has strong Kubernetes experience, EKS provides a native Kubernetes environment. This allows leveraging existing Kubernetes tools (kubectl, Helm, Kustomize) and knowledge.
        *   *Your resume mentions Kubernetes - this is a good point to bring it up.*
    *   **AWS-Native (ECS):** If the team is more familiar with AWS services and prefers a simpler, more AWS-integrated experience, ECS might be easier to adopt.
*   **Control Plane Management:**
    *   **EKS:** Managed Kubernetes control plane, but worker nodes and some networking aspects still require management (unless using Fargate with EKS).
    *   **ECS:** Fully managed control plane by AWS. Simpler from this perspective.
*   **Compute Options (Worker Nodes):**
    *   **ECS:**
        *   **Fargate:** Serverless, no EC2 instance management. Simpler operations, pay-per-use for task resources.
        *   **EC2:** More control over instance types, networking, OS. Can use Spot Instances for cost savings.
    *   **EKS:**
        *   **Managed Node Groups (EC2):** AWS helps manage EC2 worker nodes (patching, upgrades).
        *   **Self-Managed EC2 Nodes:** Full control.
        *   **Fargate:** Serverless option for EKS pods.
*   **Feature Set & Ecosystem:**
    *   **EKS (Kubernetes):** Vast open-source ecosystem, extensive features, highly configurable, supports complex scheduling and networking policies. Service mesh (e.g., Istio, Linkerd) integration is common.
    *   **ECS:** Simpler feature set but very well integrated with AWS services (IAM roles for tasks, ALB/NLB, CloudWatch, Cloud Map for service discovery).
*   **Scalability & Performance:**
    *   Both are highly scalable. EKS can offer more fine-grained control over pod placement and resource management for very large clusters.
*   **Portability & Vendor Lock-in:**
    *   **EKS (Kubernetes):** Offers better portability to other cloud providers or on-premise Kubernetes clusters.
    *   **ECS:** More AWS-specific, but this can also mean tighter and simpler integration.
*   **Networking:**
    *   **EKS:** More complex networking options (VPC CNI, Calico, etc.). Offers more flexibility for specific network policies.
    *   **ECS:** Simpler networking, integrates directly with AWS VPC networking features.
*   **Cost:**
    *   EKS control plane has an hourly cost. Worker node/Fargate costs are separate.
    *   ECS control plane is free. Pay only for Fargate tasks or EC2 instances.
    *   Fargate (on either ECS or EKS) can be more expensive than EC2 if tasks run continuously, but simpler to manage.
*   **Use Case Considerations:**
    *   **Simple Microservices, AWS-centric:** ECS with Fargate might be the quickest and simplest.
    *   **Existing Kubernetes workloads/expertise, complex requirements, multi-cloud strategy:** EKS is a strong candidate.
    *   **Need for specific EC2 instance types or GPU support:** EC2 launch type for ECS, or EC2 worker nodes for EKS.
*   **Your Experience:**
    *   *Relate to your experience with ECS at Herc Rentals/Intralinks. If you used EKS at BOFA, compare that experience.*

## Question 7: Implementing Observability for Microservices on AWS

**Question:** You are running a suite of Java/Spring Boot microservices on AWS (e.g., using ECS or EKS). How would you implement a comprehensive observability solution, covering logging, metrics, and tracing? Mention specific AWS services and third-party tools (like ELK, Prometheus, Grafana as listed on your resume).

**Discussion Points / Sample Answer Outline:**

*   **Logging:**
    *   **Application Logs (Spring Boot):**
        *   Use SLF4J with Logback/Log4j2. Configure structured logging (JSON).
        *   Include correlation IDs (trace IDs) in logs.
    *   **Collection:**
        *   **ECS/EKS with EC2:** Configure Docker logging driver (e.g., `awslogs` for CloudWatch Logs, or Fluentd/Fluent Bit sidecar/daemonset to send to ELK).
        *   **ECS/EKS with Fargate:** `awslogs` driver to CloudWatch Logs by default. Can also use Fluent Bit sidecar to send to ELK.
    *   **Centralization & Analysis (ELK Stack):**
        *   Elasticsearch for storing and indexing logs.
        *   Logstash (or Fluentd/Fluent Bit) for parsing and enrichment.
        *   Kibana for searching, visualizing, and creating dashboards.
        *   *Describe your experience setting up or using an ELK pipeline.*
    *   **CloudWatch Logs:** Can be an alternative or supplement, especially for AWS service logs.
*   **Metrics:**
    *   **Application Metrics (Spring Boot):**
        *   Micrometer with Spring Boot Actuator to expose metrics (JVM metrics, HTTP server metrics, custom business metrics).
        *   Expose in Prometheus format.
            *   *Your experience with Micrometer, Prometheus, Grafana is key here.*
    *   **Collection & Storage (Prometheus):**
        *   Prometheus server scraping metrics from application endpoints.
        *   Service discovery for Prometheus in ECS/EKS (e.g., ECS Prometheus SD, Kubernetes SD).
    *   **Visualization & Alerting (Grafana):**
        *   Grafana dashboards for visualizing application and system metrics.
        *   Alertmanager (with Prometheus) or Grafana alerting for notifications.
    *   **AWS CloudWatch Metrics:**
        *   For AWS infrastructure metrics (EC2, RDS, ALB, ECS/EKS cluster metrics).
        *   Can be scraped by Prometheus (e.g., via `cloudwatch_exporter`) or visualized directly in CloudWatch Dashboards or Grafana (CloudWatch data source).
*   **Tracing (Distributed Tracing):**
    *   **Instrumentation:**
        *   Use OpenTelemetry SDKs or AWS X-Ray SDK for Java applications to instrument incoming/outgoing requests and other operations.
        *   Auto-instrumentation agents can also be used.
        *   Ensure trace context propagation (W3C Trace Context, B3).
    *   **Backend & Visualization:**
        *   AWS X-Ray: Managed tracing service. Integrates well with other AWS services.
        *   Jaeger or Zipkin: Open-source alternatives. Can be self-hosted or use a managed service.
    *   **Integration:**
        *   Link traces with logs (e.g., include `trace_id` in logs) and metrics.
*   **Overall Strategy:**
    *   **Correlation:** Emphasize the importance of correlating logs, metrics, and traces using common IDs (e.g., `trace_id`, `user_id`).
    *   **Dashboards:** Create holistic dashboards in Grafana/Kibana showing key service health indicators.
    *   **Alerting:** Comprehensive alerting strategy based on metrics and logs.
    *   **CI/CD Integration:** Include observability configurations in your deployment pipelines.
*   **Challenges:** Agent/SDK management, data volume and cost (especially for logs and traces), ensuring consistent instrumentation across services.

## Question 8: Designing a Secure S3 Data Lake Architecture

**Question:** Your organization (e.g., TOPS project) wants to build a data lake on Amazon S3 to store raw and processed data for analytics and machine learning. Outline the key security considerations and AWS services/features you would use to protect this data lake.

**Discussion Points / Sample Answer Outline:**

*   **Data Ingestion Security:**
    *   Secure data transfer to S3 (TLS).
    *   IAM roles with least privilege for services/applications ingesting data (e.g., Kafka Connect S3 Sink, AWS Glue crawlers/jobs, Kinesis Firehose).
    *   VPC Endpoints for S3 if ingesting from within VPC to avoid public internet.
*   **S3 Bucket Security:**
    *   **Bucket Policies:** Define strict access controls. Default deny. Grant access only to specific IAM roles, users, or AWS services.
    *   **Block Public Access:** Enable S3 Block Public Access at the account and bucket level.
    *   **Object Ownership:** Control who owns objects uploaded to the bucket.
    *   **ACLs (Access Control Lists):** Generally avoid for fine-grained access; prefer bucket policies and IAM.
*   **Encryption:**
    *   **Encryption at Rest:**
        *   Server-Side Encryption with S3-Managed Keys (SSE-S3) - default.
        *   Server-Side Encryption with KMS-Managed Keys (SSE-KMS) for more control and auditability. Use customer-managed CMKs or AWS-managed CMKs.
        *   Consider client-side encryption before data reaches S3 for utmost sensitivity.
    *   **Encryption in Transit:** Enforce HTTPS (TLS) for all S3 access.
*   **Access Control for Analytics Services:**
    *   **IAM Roles:** For services like AWS Glue, Amazon Athena, Amazon EMR, SageMaker to access specific S3 prefixes/paths.
    *   **AWS Lake Formation:** Provides a centralized way to define and manage fine-grained access controls (table, column, row-level) for data in S3, integrated with Glue Data Catalog and analytics engines.
        *   *This is a key service for modern S3 data lake security.*
*   **Network Security:**
    *   **VPC Endpoints for S3:** Allow resources in private subnets to access S3 without going over the internet.
    *   Restrict S3 bucket access to specific VPCs or VPC Endpoints using bucket policies.
*   **Monitoring & Auditing:**
    *   **AWS CloudTrail:** Log all S3 API calls (data events like GetObject, PutObject, and management events).
    *   **S3 Server Access Logs:** Detailed records for requests made to a bucket.
    *   **Amazon Macie:** Discover and protect sensitive data (PII, financial data) stored in S3.
    *   **AWS Config:** Track S3 bucket configuration changes (e.g., policy changes, public access settings).
    *   **CloudWatch Alarms:** For specific CloudTrail events or S3 metrics indicating potential security issues.
*   **Data Lifecycle & Retention:**
    *   S3 Lifecycle Policies to move data to Glacier/Deep Archive or delete it based on retention requirements.
    *   S3 Object Lock for WORM (Write Once, Read Many) compliance if needed.
*   **Data Masking/Tokenization:** For sensitive data elements before exposing to less privileged users/roles in analytics tools.
*   **Cross-Account Access (if applicable):** Use IAM roles and resource-based policies carefully if other AWS accounts need access to parts of the data lake.
*   **Your Experience:**
    *   *Relate to data security practices in the TOPS project with its Kafka and S3 components.*
    *   *Discuss how you handled data governance and access control for sensitive data in any of your previous roles.*
---
These questions are designed to be open-ended, allowing Sridhar to showcase the depth and breadth of his AWS experience, especially as it relates to his resume.Okay, the file `interview-prep-sridhar-nandipati/aws-cloud/interview-questions.md` has been successfully populated with AWS-specific interview questions and discussion points tailored to Sridhar's experience.

All tasks for this sub-objective are complete. I will now submit the subtask report.
