# News-feed-from-RSS
Data Engineering Project is an implementation of the data pipeline which consumes the latest news from RSS Feeds and makes them available for users via handy API. The pipeline infrastructure is built using popular, open-source projects.

Access the latest news and headlines in one place. 💪

Table of Contents
Architecture diagram
How it works
Data scraping
Data flow
Data access
Prerequisites
Running project
Testing
API service
References
Contributions


Architecture diagram

![image](https://user-images.githubusercontent.com/96236642/158231562-8567830e-fe59-4aba-98c6-c503f37430bf.png)

Architecture diagram

MVP Architecture

How it works

Data Scraping
Airflow DAG is responsible for the execution of Python scraping modules. It runs periodically every X minutes producing micro-batches.

First task updates proxypool. Using proxies in combination with rotating user agents can help get scrapers past most of the anti-scraping measures and prevent being detected as a scraper.

Second task extracts news from RSS feeds provided in the configuration file, validates the quality and sends data into Kafka topic A. The extraction process is using validated proxies from proxypool.

Data flow

Kafka Connect Mongo Sink consumes data from Kafka topic A and stores news in MongoDB using upsert functionality based on _id field.
Debezium MongoDB Source tracks a MongoDB replica set for document changes in databases and collections, recording those changes as events in Kafka topic B.
Kafka Connect Elasticsearch Sink consumes data from Kafka topic B and upserts news in Elasticsearch. Data replicated between topics A and B ensures MongoDB and ElasticSearch synchronization. Command Query Responsibility Segregation (CQRS) pattern allows the use of separate models for updating and reading information.
Kafka Connect S3-Minio Sink consumes records from Kafka topic B and stores them in MinIO (high-performance object storage) to ensure data persistency.
Data access
Data gathered by previous steps can be easily accessed in API service using public endpoints.
Prerequisites
Software required to run the project. Install:

Docker - You must allocate a minimum of 8 GB of Docker memory resource.
Python 3.8+ (pip)
docker-compose
Running project
Script manage.sh - wrapper for docker-compose works as a managing tool.

Build project infrastructure
./manage.sh up
Stop project infrastructure
./manage.sh stop
Delete project infrastructure
./manage.sh down
Testing
Script run_tests.sh executes unit tests against Airflow scraping modules and Django Rest Framework applications.

./run_tests.sh
API service
Read detailed documentation on how to interact with data collected by pipeline using search endpoints.

Example searches:

see all news

http://127.0.0.1:5000/api/v1/news/ 
add search_fields title and description, see all of the news containing the Robert Lewandowski phrase
http://127.0.0.1:5000/api/v1/news/?search=Robert%20Lewandowski 
find news containing the Lewandowski phrase in their titles
http://127.0.0.1:5000/api/v1/news/?search=title|Lewandowski 
see all of the polish news containing the Lewandowski phrase
http://127.0.0.1:5000/api/v1/news/?search=lewandowski&language=pl

References
Inspired by following codes, articles and videos:

How we launched a data product in 60 days with AWS

Toruń JUG #55 - "Kafka Connect - szwajcarski scyzoryk w rękach inżyniera?" - Mariusz Strzelecki
Kafka Elasticsearch Sink Connector and the Power of Single Message Transformations
Docker Tips and Tricks with Kafka Connect, ksqlDB, and Kafka

Contributions
Contributions are what makes the open-source community such an amazing place to learn, inspire, and create. Any contributions you make are greatly appreciated.

Fork the Project

Create your Feature Branch (git checkout -b feature/AmazingFeature)
Commit your Changes (git commit -m 'Add some AmazingFeature')
Push to the Branch (git push origin feature/AmazingFeature)
Open a Pull Request
