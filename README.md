# Deploying ChromaDB on AWS
⚙️ Code example for [Deploying ChromaDB on AWS](https://medium.com/@ricoledan/deploying-chromadb-on-aws-37c448331911)

This AWS CloudFormation template creates a stack that runs Chroma on a single EC2 instance.
The instance is configured with Docker and Docker Compose, which are used to run Chroma and ClickHouse services.

## Components

These parameters can be provided when creating the stack:
* KeyName: The name of an existing EC2 KeyPair for SSH access to the instance. (Optional)
* InstanceType: The type of EC2 instance to be created. (Default: "m5.4xlarge")
* ChromaVersion: The Chroma version to install. (Default: "0.3.21")
* Conditions: Defines a condition named HasKeyName to check if a KeyName is provided.

### Resources

ChromaInstance:
* AMI ID
* InstanceType
* UserData
* SecurityGroupIds
* KeyName (if provided)
* BlockDeviceMappings (with a volume size of 24 GB).
* ChromaInstanceSecurityGroup: An EC2 security group with ingress rules for SSH (port 22) and Chroma server (port 8000).

### Outputs: 

The public IP address of the Chroma server is provided as an output named ServerIp.

### Mappings:

The template includes a mapping named Region2AMI with Amazon Linux 2 AMI IDs and root device names for each AWS region.

### UserData: 

The UserData section in the template installs Docker and Docker Compose, creates the necessary configuration files, and starts Chroma and ClickHouse services using Docker Compose.

## Commands

### Launch the AWS CloudFormation template

```bash
aws cloudformation create-stack --stack-name chroma-stack --template-body file://./chroma.cf.json \
--parameters ParameterKey=InstanceType,ParameterValue=m5.4xlarge
```

### Get the public IP address of your new Chroma server

```bash
aws cloudformation describe-stacks --stack-name chroma-stack --query 'Stacks[0].Outputs'
```

### Set the environmental variables within the newly created EC2 in the AWS Console

```bash
export CHROMA_API_IMPL=rest
export CHROMA_SERVER_HOST=<server IP address>
export CHROMA_SERVER_HTTP_PORT=8000
```

### Code Example: Add the server details to your application

```python
import chromadb
from chromadb.config import Settings

chroma = chromadb.Client(Settings(chroma_api_impl="rest",
chroma_server_host="<server IP address>",
    chroma_server_http_port=8000))
```

### Delete Resources

```bash
aws cloudformation delete-stack --stack-name chroma-stack
```
