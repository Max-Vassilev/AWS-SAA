# AWS-SAA

4: IAM and AWS CLI

IAM Policy Structure
- version – policy language version
- id (optional) – identifier for the policy
- statement (required) – one or more individual statements
Statement Structure
- sid (optional) – identifier for the statement
- effect (required) – allow or deny
- principal (optional for identity-based, required for resource-based) – account or role the policy applies to
- action (required) – list of actions allowed or denied
- resource (required) – list of resources the action applies to

Protect IAM users with MFA and password policy.

Ways to access AWS:
- Management Console
- CLI
- SDK

IAM Credentials Report: Generated file with detailed information for the IAM users (user, mfa_active, password_last_used and others)

IAM Access Advisor: User level - shows. service permissions granted to the user and when they were last accessed (good to determine which permissions can be removed for the user as they are not used)

IAM policy consists of:
- Version
- Statement (with Effect, Action, Resource, optional Principal)


5: EC2 Fundamentals

m5.2xlarge
- m: instance class
- 5.2: version
- xlarge: size
 
❗Instance classes:
- m, t – General purpose
- c – Compute optimised
- r – Memory (RAM) optimised
- i – Storage optimised

Ports:
- 22: SSH
- 21: FTP
- 22: SFTP (because it uses SSH)
- 80: HTTP
- 443: HTTPS
- 3389: Remote Desktop Protocol (RDP) - log into Windows instance

EC2 purchasing options:
- On-demand
- Reserved (72% discount )and convertible reserved (66% discount) - can be resold.
- Savings plan - instead of committing to a specific instance type, you commit to usage in dollars (72% discount)
- Spot instances (90% discount)
- Dedicated instance - no other customers will share your hardware
- Dedicated hosts - book an entire physical server (you get access to the server itself and gain visibility on lower level)
- Capacity reservations - reserve capacity in a specific AZ any duration

The right order to cancel and terminate spot instances is to first cancel the spot request to make sure no new instances will be automatically launched and then you terminate the associated spot instances.

❗Spot fleet (ultimate way to save money on compute) - choses the right spot instance pool to give us maximum amount of savings:
Allows you to automatically request and manage a group of Spot Instances (and optionally on-demand instances). You specify how many instances you need, and it finds and runs them at the lowest cost.
Strategies:
- lowest price: the spot fleet will launch instances from the pool with lowest price.
- diversified: the spot instances will be distributed across all the pools you have defined before. This is great for availability.
- capacity optimised: pool with optimal capacity for the number of instances you want.
- NEW - price capacity optimised: first choosing the pool with highest capacity available and then select within it the one that has the lowest price.

6: EC2 Solutions Architect Associate Level

Private vs Public vs Elastic Public IP

❗EC2 Placement Groups:
They give us control over how our EC2 instances will be placed within the AWS infrastructure.
Strategies:
- Cluster: All EC2s are in the same AZ - low latency but risky
- Spread: EC2s are spread on multiple AZs  - fault tolerance but limited to 7 EC2s in one AZ
- Partition: EC2s are spread in multiple AZs but in each AZ there are up to 7 partitions (racks) of EC2s - low latency and fault tolerance 

❗Elastic Network Interfaces - ENI (AZ service):
Automatically provides IP addresses and security groups to EC2 instances. ENI gets automatically created by launching an EC2 but you can manually create it and attach it to EC2.

❗EC2 Hibernate:
Saves RAM to disk on stop, resumes from same state on start — as if the EC2 instance was never stopped.

7: EC2 Instance Storage


EC2 EBS (volumes):
It is like a USB stick for EC2 - it stores your data.
Use case: Preserve root volume when an instance is terminated.

❗EBS Snapshots:
It is AZ service but you can create an S3 snapshot of an EBS and move it across AZs and even regions.
EBS shapshot features:
- EC2 snapshot archive: 75% cheaper but takes 12 to 24h to restore.
- Recycle bin for EBS snapshots: allows you to recover snapshot if it is accidentally deleted
- Fast snapshot restore (FSR): force full initialisation of a snapshot to have no latency on the first use (expensive)

Amazone Machine Image - AMI:
Template that contains the OS, software, and settings needed to launch a EC2 instance.

EC2 Instance Store:
Temporary high-performance storage located on physical disks attached to the host machine. Data is lost when the instance is stopped, terminated, or fails.

❗ EBS Volume Types:
- gp2 & gp3 – cost-effective, low-latency SSD storage for boot volumes, virtual desktops, and dev/test environments. gp3 lets you set IOPS and throughput independently, while in gp2 they scale together.
- io1 & io2 – highest performance and consistency; SSD for mission-critical, low-latency workloads.
- st1 – throughput-optimized HDD for frequently accessed, large workloads (e.g., big data).
- sc1 – cold HDD for infrequent, low-cost access (e.g., archival data).

 
❗EBS Multi-Attach: 
Use case: higher application availability in clustered Linux applications. Only works with EC2s from the same AZ - up to 16 EC2 instances. And only works with io1 and io2 EBS.

EBS Encryption:
Data gets encrypted at flight and at rest.

EFS (regional service): Shared file system for multiple EC2s across AZs in a region.
Performance and storage:
- Scale: auto adjusts with file changes
- Performance mode: General Purpose (low latency) or Max I/O (higher throughput)
- Throughput mode: Bursting (scales with storage), Provisioned (fixed), Elastic (auto adjusts by workload)
- Storage classes: Standard (frequently accessed), Infrequent Access (lower cost for less-used files), Archive (lowest cost for rarely accessed data.


EFS Storage Clases:
- Standard: For frequently used files
- IA: higher cost to retrieve, lower cost to store
- Archive: rarely accessed data - 50% cheaper than standard
Implement lifecycle policies to move files between storage clases.

8: High Availability and Scalability:

ELB: Automatically balances incoming traffic across multiple resources to improve availability and performance.

❗Load balancers:
1. ALB: 
- HTTP/S
- Layer 7
- 2016

2. Network LB: 
- TCP, TLS, UDP
- Layer 4
- 2017
- One static IP per AZ
- Ultra-high performance and low latency
- Health checks support: http/s and tcp protocols

3. Gateway LB:
- IP Protocol
- Layer 3
- 2020
- Routes traffic through virtual appliances (firewalls and Intrusion Detection Systems) for inspection, enabling seamless scaling.

❗ELB Sticky Sessions:
Makes sure the client is connected to the same EC2 instance as the first time in order to save session data. This way users don’t have tp re-authenticate every time.
This is done using cookies so at some point they expire.
- Supported by ALB, and Network LB
It may bring inballance to the load.

❗ELB Cross-Zone Load Balancing:
Distributes traffic evenly across all instances in all availability zones for better resource use and reliability.
With ALB it is automatically enabled by default. WIth Network and Gateway LBs it is disabled by default.
Example: 
We have 2 AZs (AZ1 and AZ2), with 2 EC2 instances in each (4 in total). If 70% of the traffic goes to AZ1, traffic won’t be evenly distributed: each EC2 in AZ1 will get 35%, and each EC2 in AZ2 will get 15%. Cross-Zone Load Balancing fixes this problem and distributed traffic evenly (25% per each EC2).

SSL/TLS certificates: In-flight encryption of data.

❗Server Name Indication (SNI):
- Multiple SSL/TLS certificates in one server
- Works with ALB and NLB

❗Connection Draining / Deregistration Delay:
- Complete final in-flight requests while an instance is marked as unhealthy. ELB stops using the EC2 instance but finishes last requests.
- Works with ALB and NLB

ASG:
- Automatically adds/removes EC2 instances based on demand
- You should create a launch template
- Works well with CloudWatch (alarm is activated and new instances are created/removed)

ASG Scaling Policies:
- Dynamic (target/simple) scaling
- Scheduled scaling
- Predictive scaling



Wrong questions:
- X-forwarded-for header: Passes users IP address to the backend (through the ALB)
- ALB supports HTTP/S and Websockets
- A Network LB has 1 static IP per AZ

9: AWS Fundamentals: RDS + Aurora + ElastiCache

RDS Storage Autoscaling: RDS detects when you will run out of storage and adds more. You just set a maximum amount. 

RDS Read Replicas:
- Used for read scalability
- Create a replica of the DB and use it just for SELECT (read) operations.
- Up to 15
- In the same AZ, in another AZ or even in another Region
- Cost for recreating replica DB: Free in the same AZ and paid for another AZ or Region

❗RDS Multi AZ:
- Used for disaster recovery
- Not used for scaling. It is just for a failover in case anything happens to the master DB
- Read Replica can be configured as multi AZ
- Snapshot of the RDS is created and restored in another RDS

❗RDS Custom for Oracle and MySQL:
Normally RDS is PaaS. But with Custom it is IaaS - the user can ssh in the server and do stuff.

Aurora:
- PostgreSQL and MySQL
- AWS product - RDS with improved performance
- 20% more expensive
- Keeps 6 copies of the DB across 3 AZs

RDS and Aurora Backup:
- Automated back ups every 5 mins - they expire. In RDS they could be disabled but not with Aurora.
- DB Snapshots - they don’t expire.

❗RDS Proxy:
A managed middleman between your application and your database in Amazon RDS or Aurora.
Makes database access faster, safer, and more reliable under heavy load or failovers.
It minimises the pull connections to the RDS.

❗ElastiCache:
In-memory, high-performance, low-latency DB. 
It reduces load on databases for read-intensive workloads by caching common queries, so results are retrieved from the cache instead of querying the DB each time.
It doesn’t necessarily have to be attached to RDS - can be simply used for storing user session data. 

10: Route 53


It is the only AWS service that offers 100% availability SLA.

Hosted zone:
Where Route 53 stores all DNS records for a domain

❗Record types:
- A: maps hostname to IPv4 (domain pointing to an IP)
- AAAA: maps hostname to IPv6
- CNAME: maps a hostname to another hostname (domain pointing to domain - ALB for example)
- NS: name servers for the hosted zone 
❗Record alias targets:
Like a CNAME but points to AWS resources (e.g., CloudFront, S3, ELB) - even better for connecting to an ALB than a CNAME, because queries are free.

❗Route 53 - Records TTL (time to live):
When a user goes to demo.something.com, it points to an IP like 1.2.3.4.
This info is cached on the device, and the device will check the IP again after X seconds.
These X seconds are the TTL.

Health checks:
Monitor the status of your resources, detect failures, and can trigger alerts.

❗Routing Policies:
Determine how traffic is directed to resources
- Simple: route traffic to single resource
- Weighted: spreads traffic across different resources (70% to one EC2 IP means 7/10 times customer will land on this EC2)
- Latency: Routes to the resource with the lowest network latency (out of all EC2 instances associated to the domain, the user accesses the closest one)
- Record Failover: if health check fails for one EC2 instance, the traffic goes to another.
- Geolocation: routes user to the EC2 instance in their country/region (or a default) — useful for compliance, licensing, or delivering localised content.
- Geoproximity: shift traffic from one region to another by increasing the bias (0-100, higher bias is more traffic to the region). 
- IP based: Routes traffic to specific resources based on the user's IP address or range.
- Multi-value (up to 8): user is routed to a random healthy EC2 instance from the list.


12: Amazon S3 Introduction

Maximum object size: 5TB (5 000 GB)

❗S3 Bucket Policy (json file):
Defines who can access a bucket and what actions they can perform.

Object Access Control List (ACL): 
Sets permissions for individual objects within a bucket.

Versioning:
Keeps multiple versions of objects to protect against accidental deletion or overwrites.

S3 Replication:
Automatically copies objects to another bucket for backup or redundancy. It only works when versioning is enabled.
- Cross Region Replication (CRR)
- Same Region Replication (SRR)

S3 Storage Classes:
- Standard: frequent access, 99.99% availability
- Standard-IA: infrequent access, 99.9% availability
- Standard One Zone-IA: infrequent access, single AZ, 99.5% availability
- Glacier Instant Retrieval: archive, immediate access
- Glacier Flexible Retrieval: archive, minutes to hours retrieval
- Glacier Deep Archive: archive, long-term, cheapest, hours to retrieve
- Glacier Intelligent Tiering: auto moves data between frequent/infrequent access
- Express One Zone: S3 Glacier storage in a single AZ, cheaper than multi-AZ, with 1–5 minute retrieval for fast archive access.

13: Advanced Amazon S3


Lifecycle rules:
Automatically move objects between storage classes

❗Requester pays:
The owner pays for storage, while the requester pays for requests and data transfer.

❗S3 Event Notifications:
They let S3 send alerts to SNS, SQS, or Lambda when objects are created, deleted, or modified. You can filter by prefix or suffix to target specific files. Also events can be sent to EventBridge for further processing and sending it to more destinations.

Multi-part upload:
Brakes big file in smaller pieces to upload it to S3.

S3 Transfer Acceleration: Speeds up uploads and downloads by routing data through AWS’s edge locations.

❗S3 Byte-range Fetches:
Lets you download a piece of a big file instead of the whole thing (part of a video or a piece of data file).

❗S3 Batch Operations:
They let you apply actions like copy, delete, or tag to many objects at once using a single job.

❗Storage Lens:
Provides insights and analytics about your storage usage and activity across your buckets, helping you optimise costs and improve data management.

S3 Analytics:
…


14: S3 Security


S3 Object Encryption:
- SSE-S3: S3 encrypts the files.
- SSE-KMS: KMS service is used for encryption.
- SSE-C: The key is managed outside of AWS but still there is server-side encryption because we send the key to AWS.
- Client-Side Encryption: the client encrypts the files.
 
S3 Encryption in transit (in flight):
Uses SSL/TLS to secure data while it moves between your client and S3.

How to force encryption in transit ? - Using a bucket policy.

❗Cross-Origin Resource Sharing (CORS):
Lets websites on other domains access your bucket safely. It controls which sites can connect, what actions they can do (like GET or POST), and which headers they can send.

MFA Delete:
Adds extra security by requiring multi-factor authentication to permanently delete objects or change versioning.

❗S3 Access Logs:
Any request from any account will go to another S3 bucket (allow or deny).

❗S3 Pre-Signed URLs: Temporary links that allow access to a specific S3 file without AWS credentials and expire after a set time.

❗Glacier Vault Lock vs S3 Object Lock: Glacier Vault Lock = whole vault, S3 Object Lock = individual objects.
Types of Object Lock:
- Compliance Mode: No one can delete/overwrite until time expires.
- Governance Mode: Protects objects, but admins can bypass.
- Legal Hold: No time limit, but the object can be removed manually.

❗S3 Access Points:  
Separate S3 bucket in folders (finance, sales …) and each folder will have its own access point with its own policy with permissions.

❗S3 Object Lambda: 
Lets you modify S3 objects on-the-fly when they are retrieved, without changing the original data (combines well with S3 Access Points).


15: CloudFront and AWS Global Accelerator


AWS CloudFront:
CDN that improves performance by caching content near users, reducing latency, and integrating with services like S3, EC2, and api gateway.

CloudFront Geo Restrictions:
Controls access to your content based on users’ geographic locations (allow or block specific countries).

CloudFront Cache invalidation: Remove or refresh cached content in CloudFront so users get the latest version immediately.

AWS Global Accelerator: A service that directs user traffic through the AWS global network to the optimal endpoint, improving application performance, reducing latency, and increasing availability across regions.

CloudFront vs Global Accelerator:
- CloudFront makes content load faster by caching it closer to users (in Edge locations).
- Global Accelerator ensures that traffic finds the fastest and most reliable route to your application (no caching).

16: AWS Storage Extras  
AWS Snowball:
…

AWS FSx:
Managed file system with automatic backups and scaling.
- FSx for Windows: managed Windows file shares
- FSx for Lustre: high-performance for ML and HPC
- FSx for NetApp ONTAP: enterprise storage with snapshots and replication
- FSx for OpenZFS: run ZFS workloads on AWS

FSx File System Deployment Options:
- Scratch file system: temporary storage and data will not be replicated - you have a file and you will lose it if the server fails. But you get 6X performance of persistence file system.
- Persistence file system: durable storage, data is replicated across multiple AZs, survives server failures, but performance is lower than scratch (up to 6× slower).

❗Storage Gateway:
Bridge between S3 and on-premises storage
- File gateway: access files in S3 like normal files
- Volume gateway: use cloud as virtual hard drives
- Tape gateway: back up data with virtual tapes
 
❗AWS Transfer Family: Service for transferring data into S3. It works with FTP, FTPS, SFTP.

❗AWS DataSync:
Synchronise data from on-premises DC to S3 and the other way around. 

❗Storage Comparison:
- S3: Object storage
- S3 Glacier: Object archival
- EBS Volume: Network storage for 1 EC2 instance at a time
- Instance Storage: Physical storage for EC2 instance
- EFS: Network file system for linux EC2 instances
- Storage Gateway: Bridge between S3 and on-premises storage (File/Volume/Tape)
- Transfer Family: FTP, FTPS, SFTP on top of S3 or EFS
- DataSync: Synchronise data from on-premises DC to S3 and the other way around
- Snow Family: Physically move large amount of data to the cloud
- Snowball Edge: …
- Database: … 
- FSx for Windows: NFS for Windows servers
- FSx for Lusture: NFS for Linux servers - high-performance computing
- FSx for NetApp ONTAP: High OS compatibility
- FSx for OpenZFS: Managed ZFS NFS

17: Decoupling Applications: SQS, SNS, Kinesis, Active MQ


SQS:
Message queue that buffers messages between services - allowing them to communicate without being directly connected.
This ensures components will work independently end reliably.
Encrypts messages by default - using SSE-SQS. Also KMS can be used.

SQS Message Visibility Timeout: Time a message stays hidden after being read so it’s not processed twice.

SQS Long Polling: Lets a consumer wait up to 20 seconds for messages before responding. This reduces empty responses and lowers costs.

SNS: Pub/sub service that sends messages to many subscribers at once.

SQS and SNS Fan Out Pattern: SNS sends a message to multiple SQS queues for parallel processing.

SNS Message Filtering: Subscribers only get messages that match filter rules.

❗Amazon Kinesis Data Streams:
Real time data streaming service.
- Provisioned mode:
- On-demand mode:

❗Amazon Data Firehose (used with Kinesis but not only):
Takes data, transforms it if needed and loads it to a destination (such as S3, Redshift or Elasticsearch).

Amazon MQ:
Message queue for RabbitMQ and ActiveMQ. 

18: Containers on AWS: ECS, Fargate, ECR & EKS


ECS:
Runs and manages Docker containers in the cloud.
Fargate is the serverless version of ECS.

Example: Push the container image to a Docker Hub. Create an ECS Fargate cluster with three EC2 instances that pull the image from the registry. 
An Application Load Balancer (ALB) automatically distributes traffic across the instances.

ECS Auto Scaling:
- CPU
- RAM
- ALB request count per target


ECR:
Service for Container registry.

EKS: Run and manage Kubernetes clusters on AWS without managing the control plane.

❗AWS App Runner:
Automatically builds, deploys, and scales web apps or APIs from source code or a container without managing servers (uses container images).

❗AWS App2Container (A2C):
Simple way to containerise and deploy existing .NET or Java applications to the cloud.


19: Serverless From the Perspective of a Solutions Architect


Lambda:
- Run code without managing server
- Pay per execution and execution time

execution
- memory: 128mb–10gb
- max execution time: 15 min
- concurrency: 1000+
deployment
- zip size: 50mb
- unzipped size: 250mb

❗Lambda Concurrency:
Number of concurrent Lambda functions that can be executed in parallel for your whole account.
- By default the maximum for concurrent functions for the account is 1000. 
- This number can be increased by opening a ticket to AWS support.
- It is recommended to reserve smaller amount per function (50 for example - after that you will have 950 left for the account.).

Cold Start: 
First request is slower.

❗Provisioned Concurrency: 
Keeps a set number of Lambda instances pre-initialized and ready, reducing cold starts.
 ❗Lambda SnapStart: 
Takes a snapshot after initialisation and reuses it, making cold starts faster without keeping instances always running.

❗Lambda@Edge Function:
Runs full backend code at CloudFront edges for complex logic (can be used for authentication).
CloudFront Function:
Runs lightweight JavaScript at edges for simple tasks like redirects or headers (can be used for URL rewrite or adding header).

❗RDS Invoking Lambda:
Automatically runs custom code in response to database changes.
Event Notifications:
Just sends alerts or messages when something happens, without running custom code.

AWS DynamoDB:
NoSQL DB
Max size of item inside: 400kb
- Provisioned Mode
- On-Demand Mode

❗DynamoDB Accelerator (DAX): In-memory cache for faster DynamoDB reads.
For DynamoDB it is better to use DAX instead of ElastiCache.

DynamoDB Streams: 
Keeps track of changes in a table, like new items, updates, or deletes, so other services can react to them.

DynamoDB Global Tables (requires Streams): 
multi-region replication

DynamoDB Time to Live (TTL): 
auto-deletes expired items

DynamoDB Backups: 
point-in-time recovery for disasters

❗API Gateway:
Serverless service that lets you expose backends like Lambda or EC2 as APIs while handling routing, security, and scaling.

❗AWS Step Functions:
Lets you coordinate AWS services into workflows, handling sequences, parallel tasks, and errors automatically.

❗Amazon Congnito: Create user base for your web or mobile app - serverless service for user authentication, authorisation, and management.

20: …

21: Databases in AWS


RDS: Managed relational database, supports multiple engines (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server), automatic backups, Multi-AZ for high availability, read replicas for scaling
Use case: traditional relational workloads

❗Aurora: PostgreSQL and MySQL Data is highly available - stored in 6 replicas across 3 AZs. Aurora Serverless - for unpredictable workloads. Aurora Global - up to 16 DB read instances in each region. Aurora DB Cloning: create new DB cluster from existing one, much faster than snapshot and restore. Use case: Same as RDS, but less maintenance and more flexibility

Amazon ElastiCache: In-Memory data store

DynamoDB (serverless):
NoSQL DB - high scale, low latency, serverless, auto-scaling needs. DAX for read cache and lower latency, fully managed NoSQL, auto-scaling, key-value and document data model

S3: Great for bigger objects Object lock vs vault lock: Object lock for WORM, vault lock for compliance S3 Batch: Perform large-scale operations on many objects at once Use case: static files, website hosting, key value store for big files

DocumentDB (not serverless): NoSQL DB - MongoDB-compatible apps with complex documents.

❗Neptune: Fully managed graph database

❗Keyspaces (for Apache Cassandra):
Exam: Keyspaces == Apache Cassandra (NoSQL) for AWS.

❗Timestream:  Fully managed time-series database, optimised for IoT and operational applications, built-in analytics, auto-scaling, metrics, sensor data, and logs

22: Data & Analytics


❗Athena: Serverless query service to analyse data in S3. You can create a table in the SQL query but Athena doesn’t store data - so it’s not a DB.
Results can be stored in S3.
Use case: analytics, reporting, and more.

❗Redshift:
Database for analytics and warehousing.

❗OpenSearch:
With OpenSearch you can search any field, even partially matches.
Example use case: Can be used in e-commerce websites to quickly find products and to track trends with easy dashboards.

❗EMR (Elastic MapReduce):
Service that processes and analyses big data with tools like Spark and Hadoop, making analytics and machine learning easier.
Use cases: Data processing, ML, Big Data.

QuickSight:
Service for creating interactive dashboards and visualising data, making it easy to share insights.
It can be integrated with services like S3, Redshift, and RDS.

AWS Glue: ETL
Prepare data for analytics.
Glue Job Bookmark: prevent re-processing old data.

Lake Formation:
Have all of your data on one place so you can do analytics on top of it.

❗Managed Service for Apache Flink:
Flink is a framework used for processing data streams in real time.
It can read from Kinesis Data streams but it can’ read from Firehose.

AWS MSK (Kafka):
Allows you to stream data (alternative to Kinesis).

23: ML


Recognition: Find things on images and videos

Transcribe:
Convert speech into text. It can automatically remove sensitive data (age or other personal data).
Also supports many languages

Polly:
Convert text into speech.

Amazon Translate:
Translate

Amazon Lex (same technology that powers Alexa):
Conversational bots - chat bots

Connect:
Call center in the cloud

Amazon Comprehend: 
Understands text, finds emotions, important words, and topics.

Comprehend Medical:
Reads medical text and finds important information like diseases, medications, treatments, and tests.

SageMaker AI:
Build custom machine learning model.

Amazon Kendra:
AI powered search tool that helps you find answers in documents.

Amazon Personalize:
AI service that helps you create personalised recommendations for users, like showing products, movies, or content they’ll like.

24: Monitoring and Audit: CloudWatch, CloudTrail & Config


CloudWatch Metrics:
Data that shows how your system is performing.

CloudWatch Logs:
Detailed records of events and actions in your system.

CloudWatch Logs Insights: 
A query tool that lets you search and analyse your existing logs.

CloudWatch Logs Live Tail:
Streams and shows logs from multiple sources in real time.

❗Amazon EventBridge:
A service that reacts to events in your AWS account and can also run scheduled tasks.
Example 1 (based on event): Configure it so when the state of EC2 instance changes it sends an email.
Example 2 (scheduled): Every 1 hour run a Lambda function every hour to clean up temporary files.

CloudWatch Container Insights: For metrics and logs coming from ECS, Fargate, RKS
For EKS you need containerised version of CloudWatch agent.

CloudTrail:
Get governance, compliance, and audit of all API activity in your AWS account - shows who did what and when.
3 Kinds of Events in CloudTrail:
- Management Events: resource operations (create, delete, update)
- Data Events: actions on data (like reading/writing to S3 or invoking a Lambda; not logged by default because they generate lots of events)
- Insights Events: unusual or anomalous activity
CloudTrail integration with EventBridge Example: 
User deletes a table in DynamoDB —> Event is recorded in CloudTrail —> This triggers EventBridge —> It triggers SNS and you receive an email about the deleted table.

AWS Config:
Tracks resource state and compliance using rules (managed or custom).
Config rules don’t deny/prevent actions from happening. They give you overview of the compliance of your resources.
Example: Assure that no security group allows SSH (port 22) to every one (0.0.0.0/0).
You can also record configuration and its changes over time to quickly be able to roll-back to figure out what happened with the infrastructure if you need to.
It also can be configured with EventBridge.

❗CloudWatch vs CloudTrail vs Config vs EventBridge:
- CloudWatch: Performance monitoring and alerting
- CloudTrail: Keep track of API calls
- Config: Record configuration changes and evaluate resources against compliance rules
- EventBridge: React to events and trigger actions automatically

25: IAM - Advanced


Organizations:
Lets you manage multiple AWS accounts centrally with one billing and unified policies.
There is 1 management account and many member accounts.

OU (Organisational Units): Groups of AWS accounts for more structured management.
Service Control Policy: Allows you to restrict what children accounts can do.
Tag Policy: Standardise tags across resources in Organisation.

IAM Conditions:
…
