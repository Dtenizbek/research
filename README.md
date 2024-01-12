# Proposed Solution for Managing Secrets in AWS EKS with Terraform
Given the current setup of storing sensitive data like database passwords in a configuration file stored with the code in GitHub, it's important to improve the security and management of these secrets. Here are a couple of options for managing secrets in AWS EKS with Terraform:

###  AWS Secrets Manager Integration:

Description: AWS Secrets Manager is a service that helps you protect access to your applications, services, and IT resources without the upfront investment and on-going maintenance costs of operating your own infrastructure. It enables you to rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle.
Implementation: Integrate AWS Secrets Manager with the application to securely store and manage the database passwords and other sensitive data. Use the AWS provider in Terraform to interact with AWS Secrets Manager and retrieve the secrets during the deployment of the application pods in EKS.
Benefits: This approach provides a centralized and secure way to manage secrets, with features such as automatic rotation of credentials and fine-grained access control.
Citation: "The AWS provider supports both AWS Secrets Manager and AWS Parameter Store. It can also be configured to rotate secrets when they expire and can synchronize AWS Secrets Manager secrets to Kubernetes Secrets" .
##### AWS Secrets Manager Integration Example:
Step 1: Create a Secret in AWS Secrets Manager
```
aws secretsmanager create-secret --name my-db-credentials --secret-string '{"username":"myuser","password":"mypassword"}'
```
Step 2: Integrate with Application
In the application code, replace the hardcoded database credentials with code to retrieve the secrets programmatically using the Secrets Manager APIs. Here's an example in Python:
```
import boto3

# Create a Secrets Manager client
client = boto3.client('secretsmanager')

# Retrieve the secret
response = client.get_secret_value(SecretId='my-db-credentials')
secret = response['SecretString']
# Use the retrieved secret in the application
```
Step 3: IAM Policy for Access
Ensure that the application has an AWS Identity and Access Management (IAM) policy permitting access to the specific secrets in Secrets Manager.

### HashiCorp Vault Integration:

Description: HashiCorp Vault is a popular open-source tool for securely storing and accessing secrets. It provides a centralized place to manage access to tokens, passwords, certificates, and encryption keys.
Implementation: Utilize HashiCorp Vault to store and manage the sensitive data. Integrate Vault with the application to dynamically retrieve the secrets during runtime. Use Terraform to provision and manage the integration of Vault with the EKS cluster.
Benefits: Vault offers advanced features such as dynamic secrets, encryption as a service, and comprehensive access control policies, providing a robust solution for managing sensitive data.
Citation: "It is very common to have a single solution for secrets that would be nice to integrate with k8s. (Hashicorp vault or AWS services like param store/secrets manager)" .
 ##### HashiCorp Vault Integration Example:
Step 1: Provision HashiCorp Vault in AWS
Use Terraform to provision HashiCorp Vault in AWS. Here's an example Terraform configuration:
```
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "vault" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"
  # other instance configuration...
}
```
Step 2: Integrate with Application
In the application code, use the Vault API to dynamically retrieve the secrets during runtime. Here's an example in Go:
```
package main

import (
	"fmt"
	"github.com/hashicorp/vault/api"
)

func main() {
	// Create a new Vault client
	client, err := api.NewClient(&api.Config{
		Address: "http://vault.example.com:8200",
	})
	// Retrieve the secret
	secret, err := client.Logical().Read("secret/data/myapp")
	// Use the retrieved secret in the application
}
```
By following these examples, you can securely manage and retrieve your sensitive data using AWS Secrets Manager or HashiCorp Vault, enhancing the security of your applications running on AWS EKS.
