# **WORK IN PROGRESS** AWS Solutions Architect Associate Certification - SAA-C02

Just some personal notes for the certification AWS SAA-C02

## Course Introduction

Measured exams skills and syllabus [here](doc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf).

Exams sample questions [here](doc/AWS-Certified-Solutions-Architect-Associate_Sample-Questions.pdf).

## AWS Infrastructure

A **region** is a cluster of data centers. AWS has regions all around the world. They are connected through an AWS private network.

A common question can be: **"How to choose a Region?"**

You need to consider multiple aspects:

- Compliance: data never eaves a region without your explicit permission
- Proximity: proximity to customer to reduce latency
- Available services: some services/features may not be available in every Region
- Pricing: pricing varies region to region and it is transparent

An **AWS Availability Zones (AZ)** is a set of one or more data centers with redundant power, networking and connectivity.
Each region has *many* AZs usually 3, min is 2, max is 6. They are not known but they are separate from each other, thus isolated from disasters.
AZs in the same Regions are connected through high performance network.

For example in the *ap-southeast-2* AWS Region we can have the following three different AZs: *ap-southeast-2a*, *ap-southeast-2b*, *ap-southeast-2c*.

Finally, AWS has hundreds **Point of Presence** all around the world to provide lower latency to users.

## IAM Services

IAM is a Global service providing Identity and Access Management. A root account is created by default for each Organization (AWS account), this should not be shared or even used. First thing to do is to create **Users** which are people within your organization. Users can be grouped together into different **groups** (e.g.: Developers, Operations, etc.).

Users can belong to 0 (not a best practice) or multiple groups.

You can create Users and Groups to assign them permissions - attaching them JSON documents called **IAM Policies**. The suggested approach is to apply the **least privilege principle**: don't give more permissions than a user needs. Moreover, you should be aware that users inherit all the permissions of their groups.

You can find an example of IAM Policies in the picture below.

![Alt text](img/IAM-Policies-example.png "IAM Policies")

In order to protect account's users you can enable:

- Password Policy
- Multi-Factor Authentication (MFA)

Always protect root account.

Finally, IAM security tools enable you to revise and manage your IAM policies:

- IAM Credentials Report (Account-level) generates a report of all IAM users and the status of their various credentials.
- IAM Access Advisor (user-level) shows the service permissions granted to a user and when those services were last used.

To access AWS, you have 3 options:

- Management Console - protected by password & MFA
- CLI - protected by access keys
- Software Developer Kit (SDK) - for code: protected by access keys

Users manage their own access keys which are secret. Don't share Access Keys.

### AWS STS

[AWS Security Token Service (STS)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) is a web-service that enables to grant limited and temporary access to AWS resources. Token is valid for up to one hour. It provides temporary security credentials that are not stored with the user but generated dynamically.

Federation lets users outside of AWS to assume temporary credentials for accessing AWS resource. Federation can have many flavors:

- SAML 2.0 (for enterprise identity federation)
- Custom Identity broker
- Web identity federation with or without Cognito
- SSO
- Non-SAML with AWS Microsoft AD.

### AWS Organizations

**AWS Organizations** is a global service that enables you to manage multiple AWS accounts. The main account is the master account and it can not be changed while other accounts are member accounts. Each member account can only be part of one organization.
This method provides you a consolidated billing across all accounts.

Multi account strategies can be applied to create one account per department, cost center, or project based on regulatory requirements, better resource isolation, to have separate per-account service limits.

You can leverage **Service Control Policies (SCP)** to control IAM actions. SCPs can be applied at the OU level or account level and automatically does not apply to Master account.

## Amazon EC2

Elastic Cloud Computing EC2 is one of the most popular of AWS' offering. AWS offers different types of instances:

- General Purpose: great for a diversity of workloads such as web servers or code repositories; great balance between compute, memory and network resources.
- Memory optimized: fast performance for workloads that process large data sets in memory, such as in-memory databases, real time processing and distributed web scale cache stores.
- Storage Optimized: great for workloads that process large data sets in storage, such as data warehouses, distributed file systems and OLTP systems.
- Compute optimized: great for workloads which require high performance processors.

EC2 instances' has the following naming convention: *instance-class*_*generation*.*size-within-the-instance-class*; e.g.: m5.2xlarge.

### Introduction to Security Groups

Security Groups are the fundamental of network security in AWS: they are acting as a *firewall* on controlling how traffic is allowed into or out EC2 instances. Sec Groups **only contain allow rules** and can reference by IP addresses or by security group.

Security Groups can be attached to multiple instances and a single instance can have multiple Security Groups. Each Security Group is region/VPC scoped.

### Elastic IPs

By default, an EC2 instance come with:

- A private IP address for the internal AWS network;
- a public IP for the Internet.

When you stop and start an EC2 instance, you lose the IP address. If you need to have a fixed public IP for your EC2 instance, you can use an **Elastic IP address**. You can assign an Elastic IP address to one EC2 instance at a time.
You can only have **5 Elastic IP** addresses in your account; even if you can ask AWS to increase that, it is not recommended. You should prefer DNS or Load Balancers services over Elastic IPs.

### Placement Groups

Sometimes you want control over EC2 Instance placement strategy, for example, you want to place your EC2 instances in different Availability Zones. That strategy can be defined by a **Placement Group**.

When you create a placement group, you can specify the following:

- **Cluster**: same rack and same AZ. Increase network performance.
- **Spread**: spread instances across different hardware and different AZs. Increase availability. Reduced network performance. Limited to 7 instances per AZ per placement group.
- **Partition**: you deploy instances in different partitions (rack). You can use up to 7 partitions per AZ and span across different AZs in the same region. Different partitions do not share hardware.

### Elastic Network Interfaces (ENI)

[ENI](https://aws.amazon.com/it/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/) is a network interface that is attached to an EC2 instance. It is a virtual network interface that is attached to an EC2 instance.
The ENI can have the following attributes:

- Primary private IP address (v4), 1+ secondary private IP addresses (v4)
- One Elastic IP address per IPv4
- One Public IP
- One or more security groups
- A MAC address

You can create ENI independently or you can move on the fly existing ENI to a different instance.
Note that any ENIs are AZ scoped.

### EC2 Nitro

EC2 Nitro is the underlying platform for the next generation of EC2 instances. It is a new virtualization technology that provides a new level of performance and security:

- Better networking options (IPv6, HPC.)
- Higher speed EBS (max 32k EBS IOPS on non-nitro instances).

### AMI Overview

**Amazon Machine Image (AMI)** is a customization of an EC2 instance. You can add your own software, data, and configuration to an AMI. AMI ensures faster boot/configuration time because is all pre-packaged. AMI are region scoped and can be copied across regions.

You can launch EC2 instances from:

- a public AMI - AWS provided
- your own AMI - make and maintain yourself
- marketplace AMI - third party.

## AWS Databases and Storage services

### Elastic Block Storage (EBS)

[**Amazon Elastic Block Store (EBS)**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonEBS.html#ebs-features) provides block level storage volumes for use with EC2 instances. EBS volumes behave like raw, unformatted block devices. You can mount these volumes as devices on your instances. EBS volumes that are attached to an instance are exposed as storage volumes that persist independently of the life of the instance. You can create a file system on top of these volumes, or use them in any way you would use a block device (such as a hard drive). You can dynamically change the configuration of a volume attached to an instance.

The following is a summary of the use cases and characteristics of SSD-backed volumes. For information about the maximum IOPS and throughput per instance:

![Alt text](img/EBS_Volume_types.jpg "EBS Types")

**EBS Multi-Attach** is a feature that allows you to attach multiple instances to a single EBS volumes. Multi-attach is supported only from io1/io2 family of volumes. Each instance has full read&write access to all attached volumes. Best use case is to achieve higher application availability in clustered environments. Applications must manage concurrent write operations on the volumes. Volumes need to be formatted in a cluster-aware file system (not XFS, EXT4, etc.).

When you create an encrypted volume, you get the following:

- Data at rest is encrypted inside the volume
- Data in transit is encrypted between the volume and the instance
- All snapshot are encrypted

Encryption has a minimal impact on latency. EBS encryption leverages keys from KMS (AES-256).

You can encrypt an unencrypted EBS volume with taking a snapshot, then encrypt the snapshot and then creating a new volume from the snapshot.

### Elastic File System (EFS)

EFS is a managed NFSv4 (network file system) that provides a file system that can be mounted on many EC2 across multi-AZ. It is highly available, scalable but is more expensive than EBS. However, it is pay per use.
You can attach a security group to an EFS instance and then **all Linux based AMI (not Windows)** can access it.

### Amazon EC2 instance store

The last option to attach storage to an EC2 instance is the [**Amazon EC2 instance store**](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/InstanceStorage.html). This is a local disk that is attached to the instance. An instance store provides temporary block-level storage for your instance. This storage is located on disks that are physically attached to the host computer. Instance store is ideal for temporary storage of information that changes frequently, such as buffers, caches, scratch data, and other temporary content, or for data that is replicated across a fleet of instances, such as a load-balanced pool of web servers.

### Amazon S3

Amazon S3 allows you to store objects (files) in **buckets** (directories). Buckets must have a globally unique name and they are defined at the regional level.

Each object has a key (name) and a value (content) is the **full path** to the file, e.g.: s3://my-bucket/myfile.txt. Object values are the content of the body. Object max size is **5 TB** but to upload more than 5 GB you must use the **multipart** upload feature. Each object can have metadata, tags and version ID (if enabled).

Versioning is enabled at the **bucket level**. Enabling versioning is a best practice and it protects you against unintended deletes and provides you easy roll back to previous versions. Note that any file not versioned prior to enabling versioning will have version `null`. Suspending versioning on a bucket does not delete the previous versions.

S3 is **strong consistent**: after a successful write or overwrite or delete operation any subsequent read or list request will return the latest version of the object.

#### S3 Encryption

There are 4 methods to encrypt objects in S3:

- SSE-S3: Server Side Encryption with S3-Managed Keys - HTTP/HTTPS with header `x-amz-server-side-encryption:AES256`.
- SSE-KMS: server side encryption with KMS - HTTP/HTTPS with header `x-amz-server-side-encryption:aes:kms`.
- SSE-C: server side encryption with customer keys not stored on AWS - only HTTPS is supported.
- CSE: Client side encryption - client store encrypted data to S3 buckets, also decryption is client side. You can leverage AWS provided library to encrypt/decrypt data.

SSE-S3 and SSE-KMS can be thought as the following:

![Alt text](img/SSEncryption.jpg "S3andKMS-encryption")

and SSE-C as follows:

![Alt text](img/SSCEncryption.jpg "SSEC-encryption").

One way to force encryption is to use bucket policy and refuse any API call to PUT an S3 object without encryption headers. Another way is to enable **default encryption** option in S3. Note that bucket policies are evaluated before default encryption.

#### S3 Security and CORS

When an S3 bucket is published as website you may need to enable [CORS](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cors.html) on the *destination* bucket.

#### S3 Replication

You can enable Cross Region Replication (**CRR**) and Same Region Replication (**SRR**) on S3 buckets. This allows you to replicate data from one region to another no matter the account owner. Copy is async. Replication is **not retroactive**, only new objects will be replicated. You can not chain replication.

Note that versioning needs to be enabled on the source and destination bucket to enable replication.

You can use CRR to meet compliance standard, gain lower latency access or replication across accounts.
You can use SRR for example for log aggregation or for live replication between production and test accounts.

You can use S3 **[pre-signed URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html)** to give temporary access to an object.

#### S3 Storage Classes and Glacier

S3 provides different storage classes:

- **Standard**
  - default storage class
  - high durability per year of objects across multiple AZ
  - 99,99% availability
  - Sustain 2 concurrent facility failure
- **Standard-Infrequent Access (IA)**
  - Suitable for data that is accessed infrequently but requires rapid access when needed
  - same durability
  - 99,9% availability
  - Sustain 2 concurrent facility failure
  - Low cost compared to standard
- **One Zone Infrequent Access (IA-1)**
  - Same as IA but in a single AZ
  - high durability
  - 99,95% availability
  - 20% lower cost than IA
- **Intelligent Tiering**
  - small monthly monitoring and auto-tiering fee
  - automatically moves objects to the most cost-effective storage class between general and IA
  - High durability
  - 99,9% availability
  - resilient against events that impact an entire AZ
- **Glacier**
  - Low cost storage for long-term archival (10s of years)
  - Alternative to on-premise magnetic tape storage
  - High annual durability
  - You are charged for storage and retrieval fees
  - Each item is called Archive and archives are stored in Vaults
  - retrieval is not immediate - different retrieval options (expedited, standard or bulk)
  - minimum storage duration of 90 days
- **Glacier Deep Archive**
  - long term storage with longer retrieval time
  - cheaper
  - minimum storage duration of 180 days

Further details [here](https://aws.amazon.com/s3/storage-classes/?nc1=h_ls)

#### S3 Lifecycle rules

You can enable actions to manage your objects based on tags or prefixes:

- **Expiration**: delete object or older version after a period of time
- **Transition**: change the storage class of an object after a period of time

### Databases in AWS

**Relational Database Service RDS** is a managed relational database service that provides high-performance, highly available, and scalable relational databases. It allows creating databases that are managed by AWS. It offers the following engines:

- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora (AWS proprietary DB)

RDS provides you:

- Provisioning and OS patching
- Continuous backups and Point in Time restore - daily full backup and transaction logs are backed-up every 5 minutes (retention between 7 and 35 days)
- DB Snapshot (different from backups, snapshots are triggered by the user and retention is arbitrary)
- Monitoring dashboard
- Read replicas for improved performance
- Multi AZ setup for Disaster Recovery
- scaling capabilities (vertical and horizontal)

However, you can not SSH the instance.

#### RDS Read Replicas

In order to improve performance you can enable **RDS Read Replicas**. It allows you to create up to 5 read replicas of an RDS instance within AZ, Cross AZ or Cross Region. Replication is ASYN so reads are **eventually consistent**. You can promote replicas to their own DB.

Usually in AWS there is a network cost when data goes from one AZ to another but, for RDS Read Replicas **within the same region** you are not overcharged. If you are using a cross region replicas you are charged for the transfer of data between the two regions.

Note that applications must update the connection string to leverage read replicas.

#### RDS Multi AZ

To provide disaster recovery you can use RDS Multi AZ. It allows you to create a single RDS instance in multiple AZs. This is useful when you need to provide high availability for your RDS instance. In this scenario RDS leverages a SYNC replication model. Using a single DNS name to access the RDS instance provide automatic app failover to standby.

This is not intended for scaling as the standby instance are not accessible.

#### RDS Encryption

You can enable at rest and in-flight encryption for RDS.

At rest encryption has to be defined at launch time. Note **that if the master is not encrypted, the read replicas cannot be encrypted**. TDE available only for SQLs server and Oracle.

Moreover, snapshots of (un-)encrypted RDS are (not) encrypted. You can copy an unencrypted snapshot into an encrypted one and vice-versa. To encrypt an unencrypted snapshot you need to create a new snapshot, copy the snapshot and enable encryption for the new snapshot, then restore the database from the snapshot and finally migrate applications to the new database.

#### RDS Security

RDS databases are usually deployed within a **private subnet**, not in a public one. You can use **security groups** to control access to the database.

IAM helps to control who can manage AWS RDS.

To sum up:

Your responsibility in AWS RDS:

- Check the ports / IP / security group inbound rules in DB’s SG
- In-database user creation and permissions or manage through IAM
- Creating a database with or without public access
- Ensure parameter groups or DB is configured to only allow SSL connections

AWS responsibility:

- No SSH access
- No manual DB patching
- No manual OS patching
- No way to audit the underlying instance

#### Amazon Aurora

Aurora is a proprietary technology from AWS (not open sourced) but Postgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL database). Aurora is **AWS cloud optimized** and claims **5x** performance improvement over MySQL on RDS, over **3x** the performance of Postgres on RDS. Aurora storage automatically grows in increments of 10 GB, up to 64 TB.  Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10 ms replica lag). Failover in Aurora is instantaneous. It’s **HA native**. Aurora costs more than RDS (20% more) – but is more efficient. Aurora stores **6 copies** of your data across **3 AZs**:

- 4 copies out of 6 needed for writes
- 3 copies out of 6 needed for reads
- Self healing with p2p replication
- storage is striped across 100s of volumes
- autoscaling

Remember the picture below:

![Alt text](img/AuroraHA.jpg "Aurora HA")

At each point in time, Aurora has a primary and 1+ secondary. The primary is the primary copy of the data. The secondary is the copy that is used for reads. The primary is the only copy that is written to. You have a DNS name for writes on the primary and a different one (connection load balancing) for reads on the secondary.
You can create custom endpoints pointing to a subset of the replicas.

Aurora security is pretty the same of RDS.

Aurora supports **Multi-Master** topology in case you want immediate failover for write node. Every node does RW instead promoting a RR (read replica) as the new master.

#### Amazon ElastiCache

ElastiCache is a managed cache service that provides high-performance, high-capacity, and cost-effective caching for Memcached and Redis.
Caches are in-memory databases with really high performance and low latency. It is used to store data that is frequently accessed, but is not stored in a long-term consistent storage. It helps to make your applications stateless.

AWS takes care of OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery and backups.

Main differences between RDS and ElastiCache important to know are the following.
Redis:

- Multi AZ with Auto-Failover
- Read Replicas to scale reads and have high availability
- Data Durability using AOF persistence
- Backup and restore features

Memcached:

- Multi-node for partitioning of data (sharding)
- No high availability (replication)
- Non-persistent
- No backup and restore
- Multi-threaded architecture

#### DynamoDB

DynamoDB is a serverless **NoSQL** database with replication across multiple AZs. DynamoDB is made of **tables** which are collections items (rows) and each item has attributes (columns). Each table can have an infinite number of items and the maximum size of an item is 400 KB.

Supported data types are:

- scalar types (string, number, boolean, null)
- Document types (List, Map)
- Set Types (Set)

You have two different options to provide DynamoDB:

- Provisioned Mode
  It requires you to specify beforehand the read and write capacity units (RCU and WCU) you want to provision. You pay for the provisioned capacity units and you can auto-scale the provisioned CU.
- On Demand Mode
  You pay for the on-demand capacity units requested. More expensive but it is more flexible for unpredictable usage.

DynamoDB provides a fully-managed, highly available and seamless in-memory cache database: DynamoDB Accelerator (**DAX**).
This feature helps to solve read congestion by caching data in memory with no impacts on the application. Default TTL for cache is 5 minutes.
It is important for the exam to understand the difference between the Elasticache and DAX.

Another important feature is **DynamoDB Streams**. It allows you to create an ordered stream of item-level modification (create, update and delete) in a table.
Stream records can be sent to Lambda and Kinesis.

DynamoDB provides **Global Tables**: a multi-region active-active replication. This replication leverages DynamoDB Streams.

## High Availability and Scalability: ELB & ASG

[**Amazon Elastic Load Balancer (ELB)**](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-elb.html) is a load balancer that provides high availability and scalability. It is a network service that provides a stable, reliable, and scalable way to distribute traffic across multiple EC2 instances.

[**Amazon Elastic Compute Cloud (EC2) Auto Scaling Groups (ASG)**](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-asg.html) is a service that provides automatic scaling for Amazon EC2 instances.

## Amazon Route 53

Amazon Route 53 is a 100% available, scalable, fully managed and **authoritative** DNS service. Route 53 is a domain registrar as well.

See [official documentation](https://aws.amazon.com/it/route53/features/) for details

## Integration and Messaging

When you deploy multiple applications they will need to communicate each other. There are two patterns of application communication:

- synchronous: application to application
- asynchronous: application to queue to application

Synchronous communication between application can be problematic, it's better to decouple your applications:

- SQS: queue model
- SNS: pub/sub model
- Kinesis: real-time streaming model

### SQS

SQS it's a queue model. It's a simple queue that can be used to communicate between applications.

You can have multiple **producers** that create messages and send them to the queue and then you can have different **consumers** that consume/poll messages from the queue. Message is persisted in SQS until a consumer deletes it.

SQS is a fully managed service with the following features:

- unlimited throughput: unlimited number of messages in queue
- default retention of messages: 4 days, minimum is 1 minute and maximum is 14 days
- low latency
- maximum message size is 256kb but you can decrease it
- Can have duplicate messages
- Can have out of order messages

SQS provides **in-flight encryption** using HTTPS API and **at-rest encryption** using KMS and client-side encryption. Moreover you can control accesses through **IAM Policies** and SQS Access Policies (similar to S3 bucket policies).

After a message is polled by a consumer it becomes invisible to other consumers. By default, the **message visibility timeout** is set to 30 seconds. This means that if you don't delete the message within 30 seconds, it will become visible again. You can adjust visibility timeout between 0 seconds and 12 hours - a consumer can change the visibility timeout using the `ChangeMessageVisibility` API.

You can enable **Dead Letter Queue (DLQ)** to send messages that is not being properly processed more than a pre-defined number of times, called `MaximumReceives`. DLQ is useful for debugging and analysis purposes.

**SQS Temporary Queue Client** allows you to implement a **Request - Response System**. This architecture is useful to scale queues. It leverages virtual queues instead of creating/deleting SQS queues.

**SQS - Delay Queue** allows you to delay a message up to 15 minutes - default is 0. This delay can be set at queue level. A producer can overwrite a delay for a specific message using the `DelaySeconds` API.

**SQS - FIFO Queue** is useful for ordering messages when you need to guarantee that messages are processed in a specific order.

### SNS

SNS is a pub/sub service to send one message to many receivers instead of deploy multiple direct integrations. With SNS multiple subscribers can leverage the same topic and receive the same message. Messages can be grouped in different **topics** and each topic can have multiple **subscribers**.
Subscribers can be SQS, HTTP/HTTPS, Lambda, SMS messages or email, Mobile notifications.

SNS integrates with many AWS services, e.g.: CloudWatch, Auto Scaling Group notifications, S3, etc.

SNS provides the same security features as SQS. You can enable in-flight encryption using HTTPS API and at-rest encryption using KMS and client-side encryption. You can control accesses through **IAM Policies** and SNS Access Policies (similar to S3 bucket policies).

As SQS, also SNS provides **FIFO model** but with the only limitation is that this use case can have only SQS queues as subscribers.

#### SNS + SQS: Fan Out

Fan out is a way to send messages to multiple subscribers, you can push once in SNS and receive in all SQS queues that are subscribers to the same topic. This is a fully decoupled model that guarantees no data loss. SQS allows for data persistence, delayed processing and retries of work as we saw in the previous section. You can scale and add new queues as needed.

SNS support **message filtering** through JSON policies.

### Kinesis

AWS Kinesis **Data Streams** is a service to collect, store, and analyze data from a variety of sources. It is a fully managed service that provides a reliable, scalable, and highly available data streaming service. You can enable multiple shards to increase throughput.

**Firehose** is a fully managed data-streaming serverless service to deliver data in **near real-time** to AWS destinations (e.g. Redshift, Elasticsearch, S3, etc.), 3rd party partner and custom destinations (e.g. HTTP Endpoint). You can enable data processing through AWS Lambda functions. Firehose does not include or require any storage.

## Container

To manage containers you have three different choices:

- ECS: AWS' own container platform
- Fargate: AWS' serverless container platform
- EKS: managed Kubernetes

### ECS

ECS stands for Elastic Container Service. You must provision and maintain the EC2 instances that will run your containers. If you use AMI to provision your instances, you will automatically have the **ECS agent** configured to interact with ECS service. In order to connect to task deployed on the cluster, the ALB will use **dynamic port mapping**: this means that ALB supports finding the right port on your EC2 instances to reach the tasks on the cluster, therefore you must allow **any port** on the EC2 instance's security group from the ALB security group.
You can mount EFS file systems to your ECS cluster, this can also be shared with Fargate Tasks.

### Fargate

Fargate is a serverless platform that allows you to launch Docker containers on AWS with no need to provision and manage any EC2 instances. AWS runs the containers based on CPU/RAM you need. In Fargate networking to your tasks is slightly different from ECS. Each task has a unique ENI (unique IP address) and you must allow on the ENI's security group the task port from the ALB security group.

### EKS

Elastic Kubernetes Service is a fully managed Kubernetes cluster on AWS. EKS supports EC2 launch mode or Fargate launch mode to deploy serverless containers.

### ECR

Elastic Container Registry (ECR) is a managed container registry service that allows you to host and manage Docker images. You can use ECR to push and pull Docker images to and from your own private Docker registry. ECR is fully integrated with ECS & IAM for security, it is backed by Amazon S3. It supports image vulnerability scanning, image lifecycle management, and image tagging.

## Security

### AWS KMS

KMS (Key Management Service) is a managed service that provides a key management system to encrypt data at rest and to encrypt data in transit. It is seamlessly integrated with many other AWS services.

KMS is **bound** to a specific region.

It offers two different Customer Master Keys (**CMK**) type:

- Symmetric (AES-256) keys: a single encryption key is used to encrypt and decrypt data and you never get access to the key unencrypted (must call KMS API to use it);
- Asymmetric (RSA and ECC) key pairs: public and private keys are used to encrypt and decrypt data or to sign/verify operations, the public is downloadable, and the private is never accessible (must call KMS API to use it).

KMS allows you to fully manage the key and policies lifecycle: you can create, delete, enable, disable, update, and rotate keys/policies. The automatic key rotation managed by KMS is a good security practice happens every 1 year. You can also create a custom key rotation schedule.

KMS offers three different CMK:

- AWS managed Service Default CMK: this is the default CMK used by AWS services (Free of charge);
- User Keys Created in KMS ($1/month);
- User Keys imported ($1/month).

Then you pay for API call to KMS ($0.03/10kcalls).

You can use KMS to encrypt all sensitive data in your application (a.g.: password, secrets).

### Secrets Manager and SSM Parameter Store

SSM is a secure storage for storing private configuration and secrets seamlessly integrated with KMS.

It allows you to assign a TTL to a parameter to force updating or deleting sensitive data.

AWS Secrets Manager provides the same functionalities but is more integrated with other AWS services and it allows easily rotation and management of secrets.

### CloudHSM

CloudHSM is a managed service that provides a **hardware** security module (HSM) to encrypt data at rest and to encrypt data in transit. You can use HSM to manage your own symmetric/asymmetric encryption keys entirely. It is tamper resistant and FIPS 140-2 L3 compliant. CloudHSM is a good option to use with SSE-C encryption.

CloudHSM infrastructure can be spread across Multi AZs.

### AWS Shield

AWS offers two different **AntiDDoS** services:

- AWS Shield Standard: a free service activated by default for every AWS customers, and it protects you from attacks such as SYN/UDP floods, reflection attacks, and other layer three or layer four attacks.

- AWS Shield Advanced: charged service ($3,000 per month per organization) DDoS mitigation service which protects you against more sophisticated attacks on your Amazon EC2, ELB, CloudFront, Global Accelerator, and Route 53 services. Also you get access 24/7 to a DDoS **response team** (DRP). Finally you can have fees waived in case of DDoS attacks.

### WAF

AWS WAF protects your web applications from attacks such as SQL injection, cross-site scripting (XSS), and other common web exploits at **Layer 7**.

It just protects: ALB, API Gateway, CloudFront.
