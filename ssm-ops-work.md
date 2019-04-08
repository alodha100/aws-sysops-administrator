# Management of EC2s at scale: Systems Manager

* Helps you manage EC2 and On-Premise systems at scale
* Gain operational insights about state of infrastructure
* Easily detect problems
* Patching automation for enhanced compliance
* Works for both Windows and Linux OS
* Integrated with CloudWatch, AWS Config & its free!

## Features

* Resource Groups
* Insights
  * Insights Dashboard
  * Inventory: discover and audit software installed
  * compliance
* Parameter Store

* Action:
   * Automation
   * **Run Command**
   * **Session Manager**
   * **Patch Manager**
   * Maintenance Windows
   * State Manager: define  and maintain configuration of OS and applications.

## How SSM works

* We need to install the SSM agent on to the systems we control.
* Installed by default on Amazon Linux AMI and some Ubuntu AMI.
* Can install on OnPrem VMs.
* If system cannot be controlled with SSM, it's probably an issue with SSM agent.
* SSM doesn't work out of the box. It must be configured.
* Make sure the EC2 instances have the correct IAM role to allow SSM actions. ( AmazonEC2RoleForSSM policy) 
  
## Tags

You can add text Key:Value pairs called Tags to many AWS resources, the naming convention is free (name it what you like).

They're used for:
   * Resource Grouping
   * Automation
   * Cost Allocation

In general, its better to have too many tags than too few.


## Resource Groups

Create, view or mangage logical groups of resources with tags.

Allows creation of logical groups of resources such as
   * Applications
   * Different layers of an application stack
   * Production versus development environments
   * Regional service
   * Works with EC2, S3, DynamoDB and Lambda

## Documents

* SSM documents can be written in JSON or YAML
* You define parameters and actions
* Many documents already exist in AWS.
* A document describes a stream of actions.

Examples:

```json
{
	"schemaVersion": "2.2",
	"description": "Install or uninstall the latest version or specified version of an AWS package. 
	                Available packages include the following: AWSPVDriver, AwsEnaNetworkDriver, IntelSriovDriver, 
	                AwsVssComponents, and AmazonCloudWatchAgent, and AWSSupport-EC2Rescue.",
	"parameters": {
		"action": {
			"description": "(Required) Specify whether or not to install or uninstall the package.",
			"type": "String",
			"allowedValues": [
				"Install",
				"Uninstall"
			]
		},
		"name": {
			"description": "(Required) The package to install/uninstall.",
			"type": "String",
			"allowedPattern": "^arn:[a-z0-9][-.a-z0-9]{0,62}:[a-z0-9][-.a-z0-9]{0,62}:([a-z0-9]
			                   [-.a-z0-9]{0,62})?:([a-z0-9][-.a-z0-9]{0,62})?:package\\/[a-zA-Z][a-zA-Z0-9\\-_]{0,39}$|^
			                   [a-zA-Z][a-zA-Z0-9\\-_]{0,39}$"
		},
		"version": {
			"description": "(Optional) A specific version of the package to install or uninstall. If installing, 
			the system installs the latest published version, by default. If uninstalling, the system uninstalls 
			the currently installed version, by default. If no installed version is found, the latest published 
			version is downloaded, and the uninstall action is run.",
			     "type": "String",
			     "default": "latest"
		}
	},
	"mainSteps": [{
		"action": "aws:configurePackage",
		"name": "configurePackage",
		"inputs": {
			"name": "{{ name }}",
			"action": "{{ action }}",
			"version": "{{ version }}"
		}
	}]
}
```

```yaml
---
schemaVersion: '2.2'
description: Sample document
mainSteps:
- action: aws:runPowerShellScript
  name: runPowerShellScript
  inputs:
    runCommand:
    - hostname
```    

## Run Command

Can be asked on the exam.

Want to run a command on ALL of our EC2s:
  * Execute a document (script) or just run a command
  * Run command across multiple instances (via resource groups)
  * Rate control / error control
  * Integrated with IAM & CloudTrail
  * No need for SSH
  * Results available in the console.

## Patching

There are different ways to use SSM to patch software or an OS:

* Inventory -> List software on an instance.
* Inventory + run command -> Patch software.
* Patch manager + maintenance window -> patch OS.
* Patch manager -> gives you compliance.
* State manager -> ensures instances are in a consistent state (compliance).

## Session Manager

* Allows you to start a secure shell on your VM
* Does not use SSH access and bastion hosts
* Only supported for EC2 right now.
* Logs actions performed through secure shells to S3 and CloudWatch.
* IAM Permissions: access SSM + write to S3 + write to CloudWatch
* CloudTrail can intercept StartSession events
* AWS Secure Shell vs SSH:
  * No need to open port 22
  * No need for bastion hosts
  * All commands are logged to S3 / CloudWatch (auditing)
  * Access to Secure Shell done through User IAM, not SSH keys.

## What if I lost my SSH key for an EC2?

Traditional, EBS backed:
* Stop the instance, detach root volume
* Attach the root volume to another instance as a data volume.
* Modify the `~/.ssh/authorized_keys` file with your new ley
* Move the volume back to the stopped instance
* Start the instance and you can SSH into it again.

New method, EBS backed:
* Run the AWSSupport-ResetAccess automation document in SSM

Instance Store backed EC2:
* You can't stop the instance, otherwise data is lost. AWS recommends termination.
* HOWEVER, You can use Session Manager access and edit the `~/.ssh/authorized_keys` directly.

## Parameter Store

Accessible via 
`EC2 -> Systems Manager Shared Resources -> Parameter Store.`
`Console -> Search "Paramter Store"`

* Secure storage for configuration and secrets.
* Optional Seemless Encryption using KMS.
* Serverless, scalable, durable, easy SDK, free.
* Version tracking of configurations / secrets.
* Configuration management using path & IAM.
* Notifications with CloudWatch Events.
* Integration with CloudFormation.

Hierarchy:

* /my-department/
  * my-app/
    * dev/
      * db-url
      * db-passwd
    * prod/
      * db-url
      * db-passwd
* /other-department/


Get parameters using the AWS cli
`aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password`

Unencrypted strings are returned in plaintex. SecureStrings are returned encrypted.

Use the `--with-decryption` flag to decrypt secure strings:

`aws ssm get-parameters --names /my-app/dev/db-url /my-app/dev/db-password --with-decryption`

You can get parameters by path

`aws ssm get-parameters-by-path --path /my-app/dev/`

You can do this recursively for all parameters under `my-app` using the `--recursive` flag:

``aws ssm get-parameters-by-path --path /my-app/dev/ --recursive`

## Opsworks

* Chef & Puppet help you perform server configuration automatically
* They work great with EC2 & On PRemise VM
* Opsworks = Managed Chef & Puppet
* It's an alternative to AWS SSM.
* **In the exam: Chef & Puppet needed --> AWS Opsworks**
* They help managing configuration as code.
* Helps in having consistent deployments.
* Works with Linux/Windows.
* Can automate: user accounts, cron, ntp, packages, services.
* Code snippets in chef are `recipes`, in puppet `manifests`.
* Many similaratities with SSM / Beanstalk / CloudFormation. They're opens-rouce and work cross-cloud.

EXAM TIP
**In the exam: Chef & Puppet needed think AWS Opsworks**s