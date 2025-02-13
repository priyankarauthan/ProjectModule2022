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
