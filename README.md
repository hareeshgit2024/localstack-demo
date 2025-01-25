# Local Stack #

## Overview - AWS ##

Amazon Web Services (AWS) is a comprehensive cloud computing platform offering over 200 fully featured services. It allows businesses to host their applications, scale workloads, and manage infrastructure without owning physical servers. AWS operates on a pay-as-you-go pricing model, which means you only pay for what you use.

**Challenges with AWS in Development Environments**

While AWS is powerful, it can be challenging to use in a development or testing environment:

- Cost: Even small-scale tests can accumulate costs.
- Access Restrictions: Developers may not always have permissions to use live AWS services.
- Complexity: Debugging in the AWS environment can be challenging.

## Overview - LocalStack ##

LocalStack is an open-source cloud service emulator that provides a fully functional local AWS cloud stack for testing and development purposes. It runs as a <ins>_single container_</ins> in our machine. It allows developers to simulate the AWS cloud services on our machines, enabling faster and cost-effective testing without using real AWS resources. LocalStack covers a wide range of AWS services like S3, Lambda, DynamoDB and many more. It is mainly useful for developers working on serverless applications and cloud-native architecture as it helps rapid iteration and debugging. By setting AWS environment locally, LocalStack helps ensure that applications behave consistently when deployed to the actual AWS cloud.

### why should I use LocalStack.? ###

Testing AWS services locally allows developers to experiment with how their applications interact with services like S3, Lambda, and DynamoDB without relying on the actual AWS cloud. This approach facilitates rapid development cycles, eliminating the need to deploy to the cloud during early stages, thus reducing both cost and complexity. LocalStack replicates the functionality of these services on a local machine, enabling swift feedback and iteration. Applications can interact with simulated external APIs without the overhead of cloud provisioning or network delays. This setup simplifies the validation of integrations and testing of cloud-based scenarios, bypassing the need to configure IAM roles or policies in a live AWS environment, and allows for the simulation of intricate cloud architectures locally before deploying changes to AWS.

**Benefits of LocalStack**

- Cost Savings: No need to pay for AWS services during development.
- Speed: No reliance on external network latency.
- Debugging: Test services in isolation.
- Simplicity: Easy to start and integrate with tools like Docker.

**When to Use AWS vs LocalStack**

| AWS                                  |	LocalStack                             |
|--------------------------------------| ------------------------------------    |
| Production-ready environments.       |	Local development and testing.         |
| Large-scale scalability.             |	Debugging and isolated service testing.|
| Real-world integration requirements. |	Cost-effective development workflows.  |


### References: ###
- https://www.localstack.cloud/
- https://docs.docker.com/guides/localstack/
  
### Setup and Run LocalStack in my machine ###

To set up and run LocalStack on your development machine, you have several options depending on your preferences and environment. Here are the main methods:

- **Docker:** LocalStack is primarily distributed as a Docker image, which is the most common way to run it. You can pull the LocalStack Docker image from Docker Hub and run it using Docker commands. This method requires Docker to be installed on your machine.
  
  ```bash
  # Pull the LocalStack Docker Image
  docker pull localstack/localstack

  # Run the LocalStack Container  
  docker run -d --name localstack_main -p 4566:4566 -p 4571:4571 localstack/localstack

  #Pull the LocalStack Pro Docker Image
  docker run --rm -it -e LOCALSTACK_AUTH_TOKEN=<<token>> -p 4566:4566 -p 4510-4559:4510-4559 localstack/localstack-pro:latest
  
  # Verify LocalStack is running  
  docker ps
  
  # Stop the container
  docker stop localstack_main

  # Remove the stopped container
  docker rm localstack_main
  
  ```

- **Docker Compose:** For more complex setups or to manage dependencies, you can use Docker Compose. This involves creating a docker-compose.yml file where you can define LocalStack and other services you might need to run alongside it.

  ```groovy
  version: '3.8'

  services:
    localstack:
      image: localstack/localstack:latest
      ports:
        - "4566:4566"            # LocalStack Gateway
        - "4571:4571"            # Internal services
      environment:
        - AWS_DEFAULT_REGION=us-east-1
        - EDGE_PORT=4566
        - SERVICES=s3,ec2,lambda # Specify the AWS services you want to emulate
        - DEBUG=1                # Enable debug mode
        - DATA_DIR=/tmp/localstack_data # Persistent data
      volumes:
        - "./localstack:/tmp/localstack_volume" # Persist data on the host

  ```

- **LocalStack CLI:** You can use the LocalStack Command Line Interface (CLI) to manage and run LocalStack. This can be installed via Python's package manager, pip, and provides commands to start and stop LocalStack easily.
  
  LocalStack CLI Commands
  
  The LocalStack CLI provides a convenient way to manage and interact with your LocalStack instance.

  Below are some of the key commands available:

  ```
  
  # Installation (use pip)
  pip install localstack-client

  #Start LocalStack
  localstack start

  #Stop LocalStack
  localstack stop

  #Checking LocalStack status
  localstack status

  #Listing all running services
  localstack services

  #Configuring LocalStack
  localstack config set <key> <value>

  #Viewing Configuration
  localstack config show

  #Help commands
  localstack --help
  
  ```
  
- **AWS CLI and SDKs:** Once LocalStack is running, you can interact with it using the AWS CLI or AWS SDKs by configuring them to point to the LocalStack endpoints instead of the real AWS endpoints.
  When using the AWS CLI with LocalStack, you can use the same commands as you would with the real AWS cloud. However, you need to configure the AWS CLI to point to your LocalStack instance.

  You can install the `AWS CLI` using the Homebrew package manager with the following command:

  ```
  brew install awscli
  ```

  Below is a sample command used with AWS CLI commands that can be utilized with LocalStack,

  Create a bucket in S3

  ```
  aws s3api create-bucket --bucket my-bucket --endpoint-url=http://localhost:4566
  ```

```
To convert the LocalStack command for listing S3 buckets into a command that works with AWS Cloud,
you simply need to remove the **--endpoint-url** parameter, as it is specific to LocalStack.

Here’s the equivalent command for AWS Cloud:

    aws s3api list-buckets

--endpoint-url: This is used to point the AWS CLI to a custom endpoint, such as LocalStack, for testing purposes.

Removing it defaults the CLI to connect to the actual AWS Cloud.

Make sure your AWS CLI is configured with valid AWS credentials and the correct region to run this command successfully

```

`When you create an S3 bucket, SQS queue, or SNS topic in AWS versus LocalStack, the underlying mechanisms and storage processes differ significantly due to the nature of their environments. Here’s a breakdown of how each service operates in both AWS and LocalStack:`

## Amazon Web Services (AWS) ##

**S3 Bucket:**

Creation: When you create an S3 bucket in AWS, the request is processed by the AWS backend, which allocates storage resources across its global infrastructure. Metadata about the bucket, such as its name, region, and settings, is stored in AWS's distributed storage systems.

Data Storage: Data stored in S3 is replicated across multiple data centers within the chosen region to ensure durability and availability.

Access: AWS uses IAM policies, bucket policies, and ACLs to manage access control.

NB : When we say AWS backend, it will be dealt by AWS Config and AWS Control Plane.

**SQS Queue:**

Creation: Creating an SQS queue involves configuring attributes such as visibility timeout and message retention. AWS backend services handle the setup and management of these configurations.

Message Storage: Messages are stored redundantly across multiple servers and data centers for durability.

Access: Access is controlled via IAM policies, and permissions can be set on the queue itself.

**SNS Topic:**

Creation: Creating an SNS topic involves setting up configurations like delivery protocols and policies. AWS manages these configurations in its backend systems.

Message Delivery: Messages published to an SNS topic are delivered to subscribed endpoints using the configured protocols (e.g., HTTP, email, SQS).

Access: Access control is managed through IAM policies and topic policies.

## LocalStack ##

**S3 Bucket:**

Creation: When you create an S3 bucket in LocalStack, it simulates the creation process by storing metadata and data locally on your machine. This is typically done using a local file system or an in-memory database.

Data Storage: The data is stored locally, and there is no replication across multiple locations, as LocalStack is designed for local testing.

Access: LocalStack does not enforce IAM policies or bucket policies in the same way AWS does, as it is primarily for development and testing purposes.

**SQS Queue:**

Creation: LocalStack simulates the creation of an SQS queue by storing queue configurations locally.

Message Storage: Messages are stored in a local database or in memory, depending on the LocalStack configuration.

Access: Access control is minimal, focusing on functionality rather than security, since LocalStack is not intended for production use.

**SNS Topic:**

Creation: Creating an SNS topic in LocalStack involves setting up local configurations that mimic AWS behavior.

Message Delivery: Messages are delivered to local endpoints or other simulated services within LocalStack.

Access: Similar to SQS, access control is simplified, and IAM policies are not enforced.

## Key Differences ##

Infrastructure: AWS uses a global, distributed infrastructure, while LocalStack runs locally on your machine.

Data Durability and Availability: AWS provides high durability and availability through data replication, while LocalStack does not replicate data.

Access Control: AWS enforces strict access control via IAM, while LocalStack simplifies access control for development purposes.

Purpose: AWS is designed for production environments, offering scalability, security, and reliability. LocalStack is intended for local development and testing, providing a lightweight simulation of AWS services.


These differences highlight the distinct purposes and capabilities of AWS and LocalStack, with AWS focusing on production-grade services and LocalStack providing a convenient local testing environment.

## Localstack Commands ##

Before getting into the Service details, let us see the health of service in Localstack.

http://localhost:4566/_localstack/health


### Simple Storage Service - S3 ###

| Operation                     |	Command                                 |
|-------------------------------| ------------------------------------    |
| Create Bucket                 |	aws s3api create-bucket --bucket trainingbucket --endpoint-url=http://localhost:4566      |
| List Buckets                  |	aws s3api list-buckets --endpoint-url=http://localhost:4566                              |
| Get Bucket Information        |	aws s3api get-bucket-location --bucket trainingbucket --endpoint-url=http://localhost:4566|
| List Objects                  |	aws s3api list-objects --bucket trainingbucket --endpoint-url=http://localhost:4566       |
| Upload File                   |	aws s3 cp << file-path >> s3://trainingbucket/ --endpoint-url=http://localhost:4566           |
|                               | aws s3 cp testfile.txt s3://trainingbucket/ --endpoint-url=http://localhost:4566 |
| Download File                 |	aws s3 cp s3://trainingbucket/<< object-key >> << local-path >> --endpoint-url=http://localhost:4566      |
|                               | aws s3 cp s3://trainingbucket/testfile.txt ./download --endpoint-url=http://localhost:4566 |
| Delete Object                 |	aws s3api delete-object --bucket trainingbucket --key <object-key> --endpoint-url=http://localhost:4566      |
|                               | aws s3api delete-object --bucket trainingbucket --key testfile.txt --endpoint-url=http://localhost:4566 |
| Delete Bucket                 |	aws s3api delete-bucket --bucket trainingbucket --endpoint-url=http://localhost:4566      |


### Simple Queue Service - SQS ###

| Operation                     |	Command                                 |
|-------------------------------| ------------------------------------    |
| Create Queue                  |	aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name trainingqueue     |
| List Queues                   |	aws sqs list-queues --endpoint-url=http://localhost:4566   |
| Send Message to Queue         |	aws sqs send-message --queue-url http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/trainingqueue --message-body "Welcome to LocalStack today" --endpoint-url=http://localhost:4566     |
| View message from a Queue     |	aws sqs receive-message --queue-url http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/trainingqueue --max-number-of-messages 5 --wait-time-seconds 5 --endpoint-url=http://localhost:4566     |
| Delete Queue                  | aws --endpoint-url=http://localhost:4566 sqs delete-queue --queue-url http://sqs.us-east-1.localhost.localstack.cloud:4566/000000000000/trainingqueue |


### Simple Notification Service - SNS ###

| Operation                     |	Command                                 |
|-------------------------------| ------------------------------------    |
| Create Topic                  |	aws sns create-topic --name << topic-name >> --endpoint-url=http://localhost:4566      |
|                               | aws --endpoint-url=http://localhost:4566 sns create-topic --name trainingtopic |
| List Topics                   |	aws --endpoint-url=http://localhost:4566 sns list-topics      |
| Subscribe                     |	aws --endpoint-url=http://localhost:4566 sns subscribe --topic-arn <topic-arn> --protocol <protocol> --notification-endpoint <endpoint> |
|                               | aws --endpoint-url=http://localhost:4566 sns subscribe --topic-arn arn:aws:sns:eu-central-1:000000000000:trainingtopic --protocol sqs --notification-endpoint arn:aws:sqs:us-east-1:000000000000:trainingqueue      |
| List Subscriptions            |	aws --endpoint-url=http://localhost:4566 sns list-subscriptions     |
| List Subscriptions by Topic   | ## aws sns list-subscriptions-by-topic --topic-arn <topic-arn> --endpoint-url=http://localhost:4566 |
| Unsubscribe                   | ## aws sns unsubscribe --subscription-arn <subscription-arn> --endpoint-url=http://localhost:4566 |
| Publish message               | ## aws sns publish --topic-arn <topic-arn> --message "<message>" --endpoint-url=http://localhost:4566 |
| Delete Topic                  |	aws --endpoint-url=http://localhost:4566 sns delete-topic --topic-arn <topic-arn>       |



## Scenario based development ##

**Scenario**: Automated Employee Data Processing

**Background**:

The HR department at Company-X is responsible for maintaining an up-to-date record of all employees. They periodically receive updates to employee information in the form of JSON files. These files include new hires, updates to existing employee details, and information on employees who have left the company. To streamline the process of updating the employee database, Company-X has implemented an automated data processing pipeline using Localstack to simulate AWS services.

**Flow**:

- Data Upload:

A member of the HR team uploads a JSON file named ` employee_updates.json ` to the designated S3 bucket. This file contains the latest employee data changes, including new hires and updates to existing records.

- Event Notification:

The upload triggers an ` S3 event notification `, which sends a message to an ` SNS topic `. This message contains metadata about the uploaded file, such as its location in the S3 bucket.

- Message Distribution:

The SNS topic immediately publishes the message to an ` SQS queue `. The queue acts as a buffer, ensuring that messages are processed sequentially and reliably.

- Data Processing:

A ` Lambda function ` is set up to trigger when a new message arrives in the SQS queue. The function retrieves the message and reads the employee_updates.json file from the S3 bucket.
The Lambda function parses the JSON data and prepares it for CRUD operations. It identifies new employee records, updates to existing employees, and any deletions that need to be made.

- Database Update:

The Lambda function communicates with the Spring Boot application, which serves as the interface for handling CRUD operations on the DynamoDB database.
The Spring Boot application processes the data, performing the necessary create, update, or delete operations on the employee records stored in DynamoDB.

- Monitoring and Logging:

Throughout the process, ` CloudWatch ` collects logs and metrics from each component. It tracks the number of files processed, the status of Lambda executions, and the health of the DynamoDB updates.
CloudWatch is configured to send alerts to the IT team if any anomalies or errors occur, ensuring that issues are addressed promptly.

**Outcome:**

By the end of this automated process, the employee records in DynamoDB are updated to reflect the latest changes from the HR department. This efficient pipeline reduces manual effort, minimizes errors, and ensures that employee data is always current.

## Solution ##

**Component Diagram**

![Component_Diagram](https://github.com/user-attachments/assets/243560ef-8b0e-4e80-b13b-475a905e6435)

**Sequence Diagram**

![SequenceDiagram](https://github.com/user-attachments/assets/48d99298-b56c-48c6-bb54-efd6e6c1b357)


