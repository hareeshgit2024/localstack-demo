# Local Stack #

## Overview ##

LocalStack is an open-source cloud service emulator that provides a fully functional local AWS cloud stack for testing and development purposes. It runs as a <ins>_single container_</ins> in our machine. It allows developers to simulate the AWS cloud services on our machines, enabling faster and cost-effective testing without using real AWS resources. LocalStack covers a wide range of AWS services like S3, Lambda, DynamoDB and many more. It is mainly useful for developers working on serverless applications and cloud-native architecture as it helps rapid iteration and debugging. By setting AWS environment locally, LocalStack helps ensure that applications behave consistently when deployed to the actual AWS cloud.

### References: ###
https://www.localstack.cloud/

https://docs.docker.com/guides/localstack/

### why should I use LocalStack.? ###

Testing AWS services locally allows developers to experiment with how their applications interact with services like S3, Lambda, and DynamoDB without relying on the actual AWS cloud. This approach facilitates rapid development cycles, eliminating the need to deploy to the cloud during early stages, thus reducing both cost and complexity. LocalStack replicates the functionality of these services on a local machine, enabling swift feedback and iteration. Applications can interact with simulated external APIs without the overhead of cloud provisioning or network delays. This setup simplifies the validation of integrations and testing of cloud-based scenarios, bypassing the need to configure IAM roles or policies in a live AWS environment, and allows for the simulation of intricate cloud architectures locally before deploying changes to AWS.

### Setup and Run LocalStack in my machine ###

To set up and run LocalStack on your development machine, you have several options depending on your preferences and environment. Here are the main methods:

- **Docker:** LocalStack is primarily distributed as a Docker image, which is the most common way to run it. You can pull the LocalStack Docker image from Docker Hub and run it using Docker commands. This method requires Docker to be installed on your machine.
  
  ```bash
  # Pull the LocalStack Docker Image
  docker pull localstack/localstack

  # Run the LocalStack Container  
  docker run -d --name localstack_main -p 4566:4566 -p 4571:4571 localstack/localstack
  
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


