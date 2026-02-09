### Trade Processing Microservices Platform

I worked on a trade processing platform built using a microservices architecture, designed to handle high-volume, event-driven trade data.

Trade data was received from upstream systems and ingested into our platform through RabbitMQ. One of our core services acted as a consumer, which received trade events and initiated the processing pipeline.

#### Enrichment & Processing

The system applied multiple enrichment rules, transformation logic, and validations on incoming trades.

Different microservices were responsible for different enrichment rules, ensuring separation of concerns and scalability.

Each service focused on a specific domain or business rule related to trade enrichment and processing.

#### Trade Lifecycle Operations

The platform supported the complete trade lifecycle, including:

Trade acknowledgment

Match / unmatch

Settlement

Cancellation

Commenting and updates

All possible trade state transitions and edge cases were handled to ensure data consistency and correctness.

#### Acknowledgment Handling

There were two types of trade acknowledgments:

SIG_ACK – Generated internally by our system.

BOX_ACK – Received from an external downstream system.

Based on the type of acknowledgment:

Trades were routed to the appropriate downstream services.

Additional processing and state updates were applied accordingly.

#### Communication & Data Flow

RabbitMQ was used for asynchronous, event-driven communication between services.

HTTP/REST calls were used for synchronous interactions where immediate responses were required.

The final processed and enriched trade data was sent to a dedicated service that indexed the data into Elasticsearch, enabling fast search and analytics.

### Key Characteristics

Event-driven and scalable microservice design

Asynchronous processing using RabbitMQ

Clear separation of enrichment responsibilities

Robust handling of trade lifecycle states

Real-time visibility via Elasticsearch



### 🛡️ Security Areas You MUST Cover

I’ll explain this in layers, exactly how architects think.

#### 1️⃣ API Security (HTTP / REST calls)
What’s needed?

✔ Authentication
✔ Authorization
✔ Secure transport

How?

OAuth2 + JWT

mTLS for internal services

HTTPS everywhere

Example

Only authorized services can:

Acknowledge trades

Perform cancel / settlement

External systems (BOX_ACK) must authenticate using:

OAuth2 client credentials

OR mutual TLS

📌 Interview line:

“All synchronous REST APIs are secured using OAuth2 with JWT, and internal service-to-service communication uses mTLS.”

### 2️⃣ RabbitMQ Security (VERY IMPORTANT)

Since RabbitMQ is your backbone, it needs strong protection.

Required Security Measures
🔹 Authentication

Username/password

Preferably cert-based auth

🔹 Authorization

Exchange-level permissions

Queue-level access control

Producer ≠ Consumer permissions

🔹 Transport Security

TLS encryption for RabbitMQ connections

🔹 Message Integrity

Signed messages

Message headers validation

Idempotency keys (prevent replay)

📌 Example:

Only the ack service can publish SIG_ACK

Only the external gateway can accept BOX_ACK

📌 Interview line:

“RabbitMQ channels are secured using TLS, fine-grained permissions, and message validation to prevent unauthorized publishing or replay attacks.”

### 3️⃣ Trade Lifecycle & State Transition Security

This is business security, not just technical.

Why important?

You must prevent:

Cancelling a settled trade

Double settlement

Invalid state transitions

How?

State machine validation

Role-based authorization

Optimistic locking

Example
NEW → ACKED → MATCHED → SETTLED
❌ SETTLED → CANCELLED (blocked)


📌 Techniques:

Versioned records

Database-level constraints

Event versioning

📌 Interview line:

“Trade state transitions are strictly validated using a state machine approach with optimistic locking to prevent illegal or concurrent updates.”

### 4️⃣ Acknowledgment Security (SIG_ACK vs BOX_ACK)

This is a BIG interview differentiator.

Risks

Fake BOX_ACK

Duplicate acknowledgments

Out-of-order ACKs

Controls

✔ Source validation
✔ Signature verification
✔ Correlation IDs
✔ Idempotent processing

Example

BOX_ACK must:

Come from whitelisted client

Have valid signature

Match trade correlation ID

📌 Interview line:

“External acknowledgments are validated using source authentication, digital signatures, and idempotent processing to prevent spoofing or duplicates.”

### 5️⃣ Data Security (DB + Elasticsearch)
At Rest

Encrypted disks

Encrypted indices (Elasticsearch)

Secure credentials (Vault / KMS)

In Transit

TLS between services and Elasticsearch

Access Control

Read-only access for analytics

No direct write access from UI

📌 Interview line:

“Trade data is encrypted at rest and in transit, and Elasticsearch access is restricted using role-based access control.”

### 6️⃣ Secrets & Configuration Security
NEVER:

❌ Hardcode credentials
❌ Store secrets in Git

DO:

✔ Use Vault / AWS Secrets Manager
✔ Rotate credentials
✔ Environment-based configs

📌 Interview line:

“All secrets such as RabbitMQ credentials and OAuth tokens are stored in a secure secrets manager and injected at runtime.”

### 7️⃣ Audit, Monitoring & Compliance

This is mandatory for trade systems.

What to log?

Trade state changes

Acknowledgment events

Who/what triggered changes

Tools

Centralized logging

Audit trails

Alerting on anomalies

📌 Interview line:

“We maintain full audit trails for all trade lifecycle events to support compliance and investigation.”

### 8️⃣ Denial of Service & Resilience Security
Protections

Rate limiting

Circuit breakers

Dead-letter queues (DLQ)

Back-pressure handling

📌 Interview line:

“Rate limiting and circuit breakers protect the platform from overload or malicious traffic.”

🧠 Security Summary (INTERVIEW GOLD)

You can say this confidently 👇

“Yes, security is critical in our trade processing platform. We secure APIs using OAuth2 and mTLS, protect RabbitMQ with TLS and fine-grained permissions, validate trade state transitions, authenticate external acknowledgments, encrypt data at rest and in transit, and maintain full audit trails for compliance.”




## 🔹 How OAuth 2.0 Works?
OAuth 2.0 does not share passwords; instead, it uses access tokens to grant limited access.

✅ Step-by-Step OAuth 2.0 Flow
###1️⃣ User Initiates Login

The user tries to log in using a third-party app (e.g., "Login with Google").

###2️⃣ Authorization Request

The app (client) redirects the user to the authorization server (Google).

Example URL:

objectivec
Copy code
https://accounts.google.com/o/oauth2/auth?
    client_id=CLIENT_ID
    &redirect_uri=CALLBACK_URL
    &response_type=code
    &scope=email profile
The user sees a consent screen and approves access.

### 3️⃣ Authorization Server Issues Code

After approval, the authorization server redirects the user to the app with an authorization code.

### 4️⃣ Client Requests Access Token

The app exchanges the authorization code for an access token by making a secure request:

bash
Copy code
POST https://oauth2.googleapis.com/token
{
  "client_id": "CLIENT_ID",
  "client_secret": "CLIENT_SECRET",
  "code": "AUTHORIZATION_CODE",
  "redirect_uri": "CALLBACK_URL",
  "grant_type": "authorization_code"
}
### 5️⃣ Authorization Server Issues Access Token

The server verifies the code and responds with an access token (and sometimes a refresh token).

### 6️⃣ Client Accesses Resource Server

The app uses the access token to make API calls and fetch user data:

bash
Copy code
GET https://www.googleapis.com/oauth2/v1/userinfo
Authorization: Bearer ACCESS_TOKEN



Example: Level 0 processor calls Level 1 processor via REST/Kafka.

Use JWT to authenticate service calls:

Each service includes a JWT in headers.

The receiver validates the JWT to ensure the request is from a trusted service.

✅ Benefit: Secures internal service communication in microservices setup.








## 🔹 Steps to Implement Secure File Transfer & Processing with Business Logic

### Step 1: Generate the Key Pair on Your Spring Boot Server
Use ssh-keygen to generate a public-private key pair.

Store the private key securely on your Spring Boot server.

Share the public key with the vendor for authentication.

### Step 2: Vendor Configures Public Key on Their FTP Server
Vendor adds the public key to the ~/.ssh/authorized_keys file.

Configures role-based authentication to control file access.

### Step 3: Configure SFTP Authentication in Spring Boot
Use JSch (Java Secure Channel) or Spring Integration SFTP to connect.

Reference the private key for authentication instead of a password.

### Step 4: Implement Polling Mechanism to Check for Files
Use Spring Scheduler (@Scheduled) or a Cron Job to poll for new files between 3-4 AM.

If a new file is detected, send a Kafka event to trigger further processing.

### Step 5: Validate and Pre-Process the File
Perform initial checks:

File format validation (CSV, JSON, XML).

Schema validation (column names, required fields).

File integrity checks (checksum, size validation).

Use Apache Commons CSV or Apache Tika for format validation.

If the file fails validation, send an error event to Kafka for alerting.

### Step 6: Apply Business Rules & Data Transformations
Process each record based on complex business rules:

Transform data formats (e.g., date conversion, currency formatting).

Apply lookup rules (e.g., enrich data with reference tables).

Validate transactional limits (e.g., max invoice amount).

Send a Kafka event after applying business rules to notify downstream services.

### Step 7: Process Large Files Using Spring Batch
Use Spring Batch for efficient chunk-based processing.

Implement retry, skip, and restart policies to handle errors.

### Step 8: Store Processed Data in a Database or Kafka
Insert cleaned and transformed data into a database (PostgreSQL, MySQL, Elasticsearch).

Alternatively, send processed data to a Kafka topic for further streaming analytics.

### Step 9: Archive or Move Processed Files
Move successfully processed files to an archive directory to prevent duplicate processing.

Use S3, HDFS, or NFS for long-term storage.

### Step 10: Implement Monitoring & Alerts
Use Splunk, ELK, or Prometheus to track job execution.

Send alerts via Kafka, email, or Slack if processing fails or takes too long.

















### 🔹 How It Works in Your Vendor File Processing System

1️⃣ Generate the Key Pair on Your Spring Boot Server

Run the following command on your Spring Boot server (or wherever the file download job runs):

```
ssh-keygen -t rsa -b 4096 -f /opt/keys/vendor_ftp_key

```
This creates:
```
Private Key: /opt/keys/vendor_ftp_key (🔒 Keep this secret!)

Public Key: /opt/keys/vendor_ftp_key.pub (📤 Share this with the vendor)
```
2️⃣ Share the Public Key with the Vendor

Send vendor_ftp_key.pub to the vendor's FTP/SFTP administrator.

The vendor will add this public key to the ~/.ssh/authorized_keys file on their FTP/SFTP server.

For example, if the vendor provides SSH/SFTP access, they will do:

```
cat vendor_ftp_key.pub >> ~/.ssh/authorized_keys
```

3️⃣ Use the Private Key in Your Spring Boot Job

Now, when your Spring Boot job connects to the vendor’s FTP/SFTP, it will use the private key instead of a password.

In your Spring Boot application, configure the job to use JSch (Java Secure Channel) for SFTP authentication with the private key




## 🔹 How to Set Time with @Scheduled in Spring Boot?

Spring Boot provides the @Scheduled annotation to run tasks at specific times, including polling an FTP server.
We can set the time between 3-4 AM using cron expressions or fixed delays.

### 🔹 Using @Scheduled with a Cron Expression (Preferred)

✅ Best when polling needs to happen at a specific time (e.g., between 3-4 AM every 10 minutes).
✅ More flexible than fixed delays.

## 📌 Poll FTP Every 10 Minutes Between 3-4 AM

```
@Scheduled(cron = "0 0/10 3-4 * * ?") // Runs every 10 minutes from 3:00 to 3:50 AM
public void checkForNewFiles() {
    List<String> newFiles = ftpService.getNewFiles("/vendor/incoming/");
    for (String file : newFiles) {
        kafkaTemplate.send("file-events", new FileEvent(file, "/vendor/incoming/"));
    }
}
```
### 🔹 Cron Breakdown: "0 0/10 3-4 * * ?"

0 → Start at second 0

0/10 → Run every 10 minutes

3-4 → Run between 3 AM - 4 AM

* * ? → Run every day










### Architecture of the Automated Data Pipeline
The automated data pipeline I implemented follows a modular, event-driven architecture, ensuring high reliability, scalability, and security. Below is a detailed breakdown of its components:

# 1. File Ingestion & Secure Transfer

Source: The pipeline ingests files from various client sources via FTP/SFTP.

# Secure Transfer:

Files are downloaded using Apache Commons Net (for FTP) or JSCH (for SFTP).

Transfers are secured using TLS/SSL encryption.

A checksum validation (e.g., SHA-256) ensures file integrity.

# Job Scheduling:

Spring Boot's @Scheduled annotation or Spring Batch jobs handle file polling.

For dynamic scheduling, Quartz Scheduler is used.

# 2. File Processing & Transformation

# Parsing & Preprocessing:

Files (CSV, JSON, XML, Parquet) are parsed using:

Apache Commons CSV (for CSVs).

Jackson / Gson (for JSONs).

Apache POI (for Excel).

# Data Sanitization:

Invalid characters, missing values, or incorrect formats are handled.

# Transformation:

Data is mapped to domain objects using DTOs (Data Transfer Objects).

Business rules are applied using Spring Batch processing steps.

# 3. Staging & Temporary Storage

Staging Tables (DB2/MySQL/PostgreSQL):
Data is temporarily stored in a staging table using Spring Data JPA.
Duplicate detection is handled using hashing (MD5, SHA-256).
In-Memory Processing (Redis):
Frequently accessed data (e.g., lookup values) is cached in Redis to reduce database load.

# 4. Data Validation & Business Rules
Validation Steps:
Ensures that mandatory fields exist.
Validates data formats (e.g., email, phone, date).
Cross-checks foreign keys against master data tables.
Error Handling & Logging:
Invalid records are logged into an error table.
Notifications (via Kafka, email, or Slack) alert the support team.

# 5. Event-Driven Processing (Kafka Integration)

Kafka for Asynchronous Processing:
Once data is validated, an event is published to a Kafka topic.
Kafka ensures real-time data streaming to downstream systems.
Consumers for Processing:
Level 0, Level 1, Level 2 processing stages consume Kafka events to process and enrich the data.
Each level transforms, validates, and stores the data progressively.
Fault Tolerance:
Consumer groups ensure that messages are processed even if a node fails.
Failed messages are stored in a dead-letter queue (DLQ) for reprocessing.
# 6. Storage & Indexing

Final Storage:
After processing, data is stored in a relational database (DB2, MySQL, PostgreSQL).
Large files are stored in NFS or HDFS, and converted into Parquet format.
Search & Analytics:
Elasticsearch is used for indexing and fast retrieval.
Logs and metrics are forwarded to Splunk for monitoring.
# 7. Authentication & Security
Single Sign-On (SSO) with OAuth2/OIDC:
Authentication is managed using OAuth2 and OpenID Connect.
Token-based authentication ensures secure API access.
Encryption & Access Control:
AES encryption secures sensitive data.
Role-based access controls (RBAC) enforce authorization.
# 8. Monitoring & CI/CD

Logging & Monitoring:
Logs are stored in Splunk or ELK Stack (Elasticsearch, Logstash, Kibana).
Prometheus + Grafana is used for real-time performance monitoring.
CI/CD Pipeline:
GitHub Actions / Jenkins automates deployment.
SonarQube ensures code quality.
# 9. Error Handling & Retries

Retry Mechanism:
Spring Retry automatically retries failed file downloads.
Kafka retry strategies handle transient failures.
Alerting System:
Failure alerts are sent via email, Slack, or PagerDuty.
High-Level Workflow:
File Download (FTP/SFTP) →
File Parsing & Preprocessing (Apache Commons CSV, Jackson) →
Data Validation & Staging (JPA, Redis) →
Event Triggering (Kafka) →
Multi-Level Processing (Level 0 → Level 1 → Level 2) →
Final Storage (DB, NFS, Elasticsearch) →
Monitoring & Alerts (Splunk, Prometheus)

# Why This Architecture?

✅ Scalability – Kafka ensures real-time processing at scale.
✅ Fault Tolerance – Redis caching and DLQ prevent data loss.
✅ Security – OAuth2/OIDC for authentication, AES encryption for data.
✅ Performance Optimization – Staging tables, Redis caching, and Parquet format reduce latency.



### Q- Large files are stored in NFS or HDFS, and converted into Parquet format. What do we use to convert files to parquet format
✅ Apache Spark – If you're working with big data on HDFS/NFS and need distributed processing.
Handling Large Files from a Vendor Using Distributed Processing

#### When receiving a large file from a vendor, the main challenges are efficient file distribution, parallel processing, and format conversion. Here's how a distributed system processes such a large file efficiently:

# 1. File Download & Storage
Secure File Transfer
Files are received via FTP/SFTP and downloaded using:
Apache Commons Net (FTP)
JSCH (SFTP)
Secure transmission using TLS/SSL.
Checksum validation (SHA-256) ensures data integrity.
Storage Location
The file is stored in a distributed file system (NFS, HDFS, S3, or HDFS-based storage like HBase) for further processing.

# 2. File Splitting for Parallel Processing
Once stored, the large file needs to be split into smaller chunks so that multiple nodes can process it in parallel.

Approach 1: Using HDFS Block Storage
If stored in HDFS, the file is automatically split into blocks (128MB or 256MB each).
Each block is replicated across multiple data nodes for reliability.

Approach 2: Manually Splitting the File
If the file is a CSV or JSON, we split it using:
Apache Hadoop FileSystem API
Apache Spark’s textFile().repartition(N)
Linux split command (for traditional file systems)

# 3. Distributed Processing Across Multiple Nodes
After splitting, multiple nodes process the file in parallel.

Processing Frameworks
Apache Spark (Best for Large-Scale Processing)

Uses RDDs (Resilient Distributed Datasets) to process data in parallel.
Supports in-memory computation for fast processing.
Example: Spark reads Parquet, transforms data, and stores it.


Dataset<Row> df = spark.read().format("csv").option("header", "true").load("hdfs://path/to/file.csv");
df.write().parquet("hdfs://path/to/output.parquet");
Apache Flink (For Streaming & Batch Processing)

Provides real-time and batch processing.
Works well with Kafka for streaming large files.
Kafka + Microservices

Kafka producers push split file chunks into Kafka topics.
Multiple Kafka consumers (Microservices) read and process chunks concurrently.

# 4. File Conversion to Parquet Format

# Why Convert to Parquet?

Columnar format → Fast reads and efficient storage.
Compression → Reduces storage footprint.
Optimized for analytics.
Tools for Conversion
Apache Spark

Dataset<Row> df = spark.read().format("csv").load("hdfs://path/to/split-file.csv");
df.write().parquet("hdfs://path/to/output.parquet");
Apache Parquet Library (Java)

Configuration conf = new Configuration();
ParquetWriter<Group> writer = ExampleParquetWriter.builder(new Path("output.parquet")).withConf(conf).build();
Pandas (Python)
python

import pandas as pd
df = pd.read_csv("split-file.csv")
df.to_parquet("output.parquet")

5. Storing & Indexing
   
Final converted files are stored in HDFS, AWS S3, or a distributed file system.

Elasticsearch indexing is used for fast retrieval.

# 7. Monitoring & Fault Tolerance
   
Failure Handling: If a node fails, Kafka’s consumer group rebalances the load.
Logging: Logs are sent to Splunk / ELK Stack.
Retries: Spring Retry, Kafka retries, or Spark checkpointing ensures processing resumes from the last successful stage.

### End-to-End Workflow

Vendor sends a file via FTP/SFTP → Downloaded & stored in HDFS/NFS/S3.

File is split into chunks for parallel processing.

Distributed processing using Apache Spark, Kafka, or Flink.

Each node processes its chunk & converts it into Parquet.

Processed data is stored in HDFS/S3/DB and indexed in Elasticsearch.

Monitoring & fault tolerance ensure a resilient system.

































# To know when a vendor file is ready for download from the vendor’s location to your system, you can use one of the following mechanisms depending on how the vendor provides file availability notifications:

## 🔹 Approach 1: Polling Mechanism (Scheduled Check)
If the vendor does not provide any event notification, you can periodically check if the file is available at their location.

✅ Steps
Schedule a job in Spring Boot that checks for new files at regular intervals.
If the file exists, download it.
📌 Implementation (Using SFTP)

import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;
import java.io.File;
import java.io.FileOutputStream;
import java.util.Properties;

@Service
public class VendorFileDownloader {

    private static final String SFTP_HOST = "vendor.sftp.com";
    private static final int SFTP_PORT = 22;
    private static final String SFTP_USER = "username";
    private static final String SFTP_PASSWORD = "password";
    private static final String VENDOR_DIRECTORY = "/vendor/files/";
    private static final String LOCAL_DIRECTORY = "/local/vendor/files/";

    @Scheduled(fixedRate = 60000) // Runs every 60 seconds
    public void checkAndDownloadFiles() {
        try {
            JSch jsch = new JSch();
            Session session = jsch.getSession(SFTP_USER, SFTP_HOST, SFTP_PORT);
            session.setPassword(SFTP_PASSWORD);

            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            session.setConfig(config);
            session.connect();

            ChannelSftp channelSftp = (ChannelSftp) session.openChannel("sftp");
            channelSftp.connect();

            channelSftp.cd(VENDOR_DIRECTORY);
            for (Object obj : channelSftp.ls(VENDOR_DIRECTORY)) {
                ChannelSftp.LsEntry entry = (ChannelSftp.LsEntry) obj;
                String fileName = entry.getFilename();

                if (!fileName.equals(".") && !fileName.equals("..")) {
                    System.out.println("Downloading file: " + fileName);
                    try (FileOutputStream fos = new FileOutputStream(new File(LOCAL_DIRECTORY + fileName))) {
                        channelSftp.get(fileName, fos);
                    }
                    System.out.println("File downloaded: " + fileName);
                }
            }

            channelSftp.disconnect();
            session.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
🔹 Why Use This?
✅ Works for SFTP, FTP, and cloud storage
✅ Simple to implement
✅ Good if the vendor does not provide real-time notifications

## 🔹 Approach 2: Vendor Sends an Event Notification (Best for APIs or Webhooks)
If the vendor provides an API or webhook notification, you can configure a listener to receive a message when the file is ready.

✅ Steps
Vendor sends an API call/webhook when the file is ready.
Your Spring Boot application receives the event and downloads the file.
📌 Implementation (Webhook Listener for File Ready Event)


import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/vendor")
public class VendorFileWebhookController {

    @PostMapping("/file-ready")
    public String handleVendorFileReady(@RequestBody VendorFileEvent event) {
        System.out.println("File ready event received: " + event.getFileName());
        downloadFile(event.getFileName());
        return "File download started";
    }

    private void downloadFile(String fileName) {
        System.out.println("Downloading file: " + fileName);
        // Add logic to download file from vendor's location
    }

    static class VendorFileEvent {
        private String fileName;
        public String getFileName() { return fileName; }
        public void setFileName(String fileName) { this.fileName = fileName; }
    }
}
🔹 Why Use This?
✅ Real-time notification
✅ No need for constant polling
✅ Efficient for cloud APIs

## 🔹 Approach 3: Kafka Event Streaming (Best for Enterprise Systems)
If the vendor publishes file availability events via Kafka, you can consume the event and start downloading the file.

✅ Steps
Vendor publishes a Kafka message with file details.
Your Spring Boot app listens to Kafka topic and downloads the file.
📌 Implementation (Kafka Consumer for File Events)
java
Copy
Edit
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class FileEventConsumer {

    @KafkaListener(topics = "vendor-file-events", groupId = "vendor-group")
    public void consumeFileEvent(ConsumerRecord<String, String> record) {
        System.out.println("File ready event received: " + record.value());
        downloadFile(record.value());
    }

    private void downloadFile(String filePath) {
        System.out.println("Downloading file: " + filePath);
        // File download logic
    }
}
🔹 Why Use This?
✅ Real-time file tracking
✅ Scales well in microservices
✅ Decouples vendor system from your system

## 🔹 Approach 4: Checking for a Ready Flag File (Used for Batch Processing)
If the vendor drops files in a location, they may provide a flag file (e.g., .done, .ready) when the file is fully uploaded.

✅ Steps
Wait for file.ready or file.done to appear.
Once detected, download the corresponding file.
📌 Implementation (Using Java Polling)

import java.io.File;
import java.nio.file.*;

public class ReadyFileWatcher {

    private static final String VENDOR_DIRECTORY = "/vendor/files/";
    private static final String LOCAL_DIRECTORY = "/local/vendor/files/";

    public static void main(String[] args) {
        watchDirectory(VENDOR_DIRECTORY);
    }

    public static void watchDirectory(String dirPath) {
        Path path = Paths.get(dirPath);

        try (WatchService watchService = FileSystems.getDefault().newWatchService()) {
            path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE);

            while (true) {
                WatchKey key = watchService.take();
                for (WatchEvent<?> event : key.pollEvents()) {
                    Path filePath = ((WatchEvent<Path>) event).context();
                    if (filePath.toString().endsWith(".ready")) {
                        String fileName = filePath.toString().replace(".ready", "");
                        System.out.println("File is ready: " + fileName);
                        downloadFile(fileName);
                    }
                }
                key.reset();
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void downloadFile(String fileName) {
        System.out.println("Downloading file: " + fileName);
        // Add logic to download file
    }
}
🔹 Why Use This?
✅ Works well for batch-based vendor systems
✅ Ensures files are fully uploaded before downloading

🎯 Which One Should You Choose?
Approach	Best For	Pros	Cons
Polling (Scheduled Check)	Vendors with no notifications	Simple, works with FTP/SFTP/HDFS	Not real-time
Webhook Notification	API-based vendor services	Real-time, efficient	Vendor must support webhooks
Kafka Event Streaming	Event-driven vendors	Scalable, real-time	Requires Kafka setup
Ready Flag File	Batch file uploads	Ensures file completion	Requires vendor cooperation
🚀 Recommended Approach for Your Vendor File Processing System
If vendor provides an API notification → Use Webhook
If vendor supports Kafka → Use Kafka Event Processing
If no real-time notification is available → Use Polling Mechanism
If vendor drops a flag file → Use File Watcher






## Level 0: Raw File Processing with Spark
Objective: Load and validate the raw .txt file, perform initial transformations or checks, and store the result in NFS.
Example Spark Job for Level 0:

import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

public class Level0Processing {

    public static void main(String[] args) {
        // Initialize Spark session
        SparkSession spark = SparkSession.builder()
                .appName("Level 0 Raw File Processing")
                .getOrCreate();

        // Load raw .txt file into Spark
        String rawFilePath = "/nfs/raw_data/raw_file.txt";
        Dataset<Row> rawData = spark.read().text(rawFilePath);

        // Process or validate the raw file (e.g., check for empty lines, malformed data, etc.)
        Dataset<Row> validatedData = rawData.filter("length(value) > 0");

        // Write validated data to NFS in a structured format
        String level0OutputPath = "/nfs/processed_data/level0/";
        validatedData.write().parquet(level0OutputPath);

        // Stop Spark session
        spark.stop();
    }
}
## Level 1: Cleansing and Parquet Conversion with Spark
Objective: Cleanse and process the data from the raw file and convert it to Parquet format.
Example Spark Job for Level 1:

public class Level1Processing {

    public static void main(String[] args) {
        // Initialize Spark session
        SparkSession spark = SparkSession.builder()
                .appName("Level 1 Data Cleansing and Parquet Conversion")
                .getOrCreate();

        // Load validated data from Level 0 (stored as Parquet)
        String level0DataPath = "/nfs/processed_data/level0/";
        Dataset<Row> dataFromLevel0 = spark.read().parquet(level0DataPath);

        // Perform cleansing and transformations (e.g., remove duplicates, handle missing data)
        Dataset<Row> cleansedData = dataFromLevel0.dropDuplicates();

        // Write the cleansed data back to NFS in Parquet format
        String level1OutputPath = "/nfs/processed_data/level1/";
        cleansedData.write().parquet(level1OutputPath);

        // Stop Spark session
        spark.stop();
    }
}
## Level 2: Final Processing and Parquet Generation with Spark
Objective: Apply the final business logic or transformations to the data and produce the final Parquet file.
Example Spark Job for Level 2:

public class Level2Processing {

    public static void main(String[] args) {
        // Initialize Spark session
        SparkSession spark = SparkSession.builder()
                .appName("Level 2 Business Logic and Parquet Finalization")
                .getOrCreate();

        // Load cleansed data from Level 1
        String level1DataPath = "/nfs/processed_data/level1/";
        Dataset<Row> dataFromLevel1 = spark.read().parquet(level1DataPath);

        // Apply business logic (e.g., calculate new columns, enrich data)
        Dataset<Row> finalData = dataFromLevel1.withColumn("new_column", functions.lit("processed"));

        // Write the final data to NFS in Parquet format
        String finalOutputPath = "/nfs/processed_data/final/";
        finalData.write().parquet(finalOutputPath);

        // Stop Spark session
        spark.stop();
    }
}

## Kafka Integration (Event Triggering)
At each level, after the processing is done, you can trigger a Kafka event to signal the completion of the task and move on to the next level.

For example, after Level 0 processing:

// After processing, publish a Kafka event to trigger Level 1
kafkaTemplate.send("level0_complete", "fileName.parquet");

## Summary Workflow
Level 0 (Raw File Processing):

Load the raw .txt file using Spark.
Perform validation/initial checks.
Store processed data in Parquet format in NFS.
Level 1 (Cleansing and Parquet Conversion):

Load the Parquet file from Level 0.
Cleanse the data (e.g., drop duplicates, handle missing values).
Convert it to a new Parquet file and store it in NFS.
Level 2 (Business Logic and Final Parquet Generation):

Apply business logic on the cleansed Parquet data.
Generate the final Parquet file and store it in NFS.
Kafka Event-Driven Pipeline:

After each level of processing, a Kafka event is published to signal the completion of that stage, triggering the next stage.
 ## The architecture you've designed for the vendor file processing system is highly scalable due to several key elements:

# 1. Distributed Processing with Apache Spark
Horizontal Scaling: Spark can scale across many nodes in a cluster, enabling you to process large datasets in parallel. As your data grows, you can add more Spark workers to the cluster to handle increased workloads.
Efficient Data Processing: By using distributed in-memory computation, Spark is optimized for fast data processing. It allows for processing large files (like your raw .txt files) in a highly efficient manner.
Data Partitioning: Spark supports data partitioning, meaning it can split datasets across multiple nodes to process them in parallel, increasing throughput and performance.
# 2. Kafka for Event-Driven Architecture
Asynchronous Processing: Kafka enables asynchronous communication between the different stages of your pipeline. This means each level can process data independently and trigger events when they are done. It decouples the stages and prevents bottlenecks, ensuring that the system can handle high throughput.
Scalable Event Handling: Kafka is designed to handle high volumes of events and messages. By adding more Kafka brokers, you can horizontally scale your messaging infrastructure to handle increased load without significant performance degradation.
Consumer Parallelism: Kafka consumers can be scaled out to multiple instances, allowing you to process events in parallel. For instance, you can have multiple consumers for the download_ready or level0_complete events, ensuring fast processing.
# 3. Storage in NFS
Shared Storage: Using NFS ensures that all nodes (e.g., Spark workers) can access the same data, making it easy to scale the system without worrying about managing data replication or synchronization across different nodes.
Storage Scaling: As your dataset grows, you can scale the NFS storage capacity to store larger files, and it's easy to expand the storage without affecting the core processing logic.
# 4. Kafka-Driven Pipeline for Flexibility and Extensibility
Event Triggering Between Stages: The use of Kafka events between stages (Level 0, Level 1, Level 2) allows you to scale each stage independently. Each stage can be processed as an independent microservice or worker, and you can scale them up based on demand.
Loose Coupling of Components: With Kafka, each part of the pipeline (file checking, downloading, processing, etc.) can work independently and be scaled separately. This loose coupling allows for easier scaling, maintenance, and updates without affecting other parts of the system.
# 5. Spark’s Parquet File Storage for Efficiency
Efficient File Format: Parquet is a columnar file format that is optimized for reading large datasets efficiently. By converting your raw data to Parquet format at each level, you improve I/O performance and compression, reducing the amount of storage needed and the time required to process the data.
Compatibility with Big Data Tools: Parquet files are compatible with various big data tools and platforms, which allows you to integrate other systems (like Hadoop or more Spark clusters) if needed, making it easier to scale your system for future use cases.
# 6. Scalability of Each Level
Level 0 (File Checking/Downloading): This level involves checking for the file and downloading it. As you scale, you can introduce multiple downloaders (Kafka consumers) to handle more files concurrently.
Level 1 (Cleansing and Conversion): With Spark, you can scale this processing step by running multiple jobs across different worker nodes. As data grows, Spark can distribute the cleansing process over more nodes.
Level 2 (Business Logic/Final Processing): Similarly, in the final processing stage, you can scale out by running multiple Spark jobs, each handling different parts of the data or running concurrently.
# 7. Microservices and Independent Scaling
Microservice Architecture: If your pipeline is built using microservices, each stage (file checking, downloading, Level 0, Level 1, Level 2) can be deployed as a separate microservice that can be scaled independently based on the load.
Auto-scaling with Kubernetes: If you're deploying the system on Kubernetes (K8s), you can use Kubernetes' auto-scaling capabilities to scale up/down the number of pods (instances) of each microservice based on the number of events in Kafka, the file load, or the amount of data being processed.
# 8. Horizontal Scaling for Spark
Scale Spark Cluster: You can increase the size of your Spark cluster as your data grows. This means more nodes can join the cluster, allowing you to process larger datasets more efficiently and quickly.
Dynamic Resource Allocation: Spark can dynamically allocate resources (memory, CPU) based on the workload, making it easy to scale processing efficiently.
# 9. Fault Tolerance and Reliability
Spark Fault Tolerance: Spark provides built-in fault tolerance by replicating data across nodes in the cluster. If a node fails, Spark can recover the lost data from other nodes, ensuring the system continues to run smoothly.
Kafka’s Durability: Kafka ensures that messages (events) are durable and can be reprocessed if necessary, so even if a stage fails, you can re-trigger the processing without losing data.
## How Scalability Works Together:
Parallelism in both Kafka (events) and Spark (data processing) ensures that as the volume of files increases, your system can handle more tasks concurrently.
The decoupling between the stages and the use of Kafka enables each stage to scale independently, ensuring that heavy traffic or processing at one level does not slow down the others.
As the system grows, you can horizontally scale both the Kafka brokers and Spark workers to meet increasing demand, ensuring the system remains responsive and efficient.
Future Scalability Considerations:
If your data volume grows significantly or your use cases expand (e.g., adding more vendors), you can introduce additional Kafka topics or Spark clusters to manage the increased load.
With cloud infrastructure, you can take advantage of on-demand scaling for both Kafka and Spark, ensuring cost-efficiency and resource optimization.
Would you like more detailed examples on any specific aspect of scaling, or guidance on setting up these components in a cloud environment like AWS or Azure?




























├── vendor-file-processing
│   ├── sftp-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.sftp
│   │   │   │   │       ├── SftpApplication.java
│   │   │   │   │       ├── config
│   │   │   │   │       │   ├── SftpConfig.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── SftpService.java
│   ├── storage-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.storage
│   │   │   │   │       ├── StorageApplication.java
│   │   │   │   │       ├── config
│   │   │   │   │       │   ├── S3Config.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── StorageService.java
│   ├── processing-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.processing
│   │   │   │   │       ├── ProcessingApplication.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── FileProcessingService.java
│   ├── indexing-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.indexing
│   │   │   │   │       ├── IndexingApplication.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── ElasticsearchIndexingService.java
│   ├── auth-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.auth
│   │   │   │   │       ├── AuthApplication.java
│   │   │   │   │       ├── config
│   │   │   │   │       │   ├── SecurityConfig.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── AuthService.java
│   ├── event-service
│   │   ├── src
│   │   │   ├── main
│   │   │   │   ├── java
│   │   │   │   │   └── com.vendor.event
│   │   │   │   │       ├── EventApplication.java
│   │   │   │   │       ├── config
│   │   │   │   │       │   ├── KafkaConfig.java
│   │   │   │   │       ├── service
│   │   │   │   │       │   ├── KafkaEventService.java
│   ├── k8s-deployment
│   │   ├── sftp-deployment.yaml
│   │   ├── storage-deployment.yaml
│   │   ├── processing-deployment.yaml
│   │   ├── indexing-deployment.yaml
│   │   ├── auth-deployment.yaml
│   │   ├── event-deployment.yaml
│   ├── README.md
