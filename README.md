├── vendor-file-processing
│   ├── src
│   │   ├── main
│   │   │   ├── java
│   │   │   │   └── com.vendor.processing
│   │   │   │       ├── VendorFileProcessingApplication.java
│   │   │   │       ├── config
│   │   │   │       │   ├── KafkaConfig.java
│   │   │   │       │   ├── ElasticsearchConfig.java
│   │   │   │       │   ├── HDFSConfig.java
│   │   │   │       ├── controller
│   │   │   │       │   ├── FileUploadController.java
│   │   │   │       ├── service
│   │   │   │       │   ├── FileIngestionService.java
│   │   │   │       │   ├── FileProcessingService.java
│   │   │   │       │   ├── DataProcessingService.java
│   │   │   │       │   ├── ElasticsearchIndexingService.java
│   │   │   │       │   ├── ParquetFileService.java
│   │   │   │       │   ├── KafkaEventService.java
│   │   │   │       ├── repository
│   │   │   │       │   ├── VendorDataRepository.java
│   │   │   │       ├── model
│   │   │   │       │   ├── VendorData.java
│   │   │   │       ├── events
│   │   │   │       │   ├── FileUploadedEvent.java
│   │   │   │       │   ├── FileParsedEvent.java
│   │   │   │       │   ├── DataProcessedEvent.java
│   │   │   │       │   ├── DataStoredEvent.java
│   │   ├── resources
│   │   │   ├── application.yml
│   ├── Dockerfile
│   ├── README.md
│   ├── pom.xml
