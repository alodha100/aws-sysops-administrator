# EC2 for SysOps


## Placement Groups

* Cluster – only 7 instances in a cluster, shares the same hardware (rack)

* Spread – instances located on different hardware (different rack)

## Termination Protection

Command Line shutdown will terminate instance unless termination protection is enabled.

## Troubleshooting EC2 launch issues

### InstanceLimitExceeded

Maximum number of EC2s reached in that region. Open support ticket with AWS to increase this value. Max number by default is 20. This is a soft limit that can be increased by opening a support ticket.

Go to EC2 Dashboard -> Limits to view resource limits of EC2s.

### InsuffcientInstanceCapacity

AWS does not have the capacity to launch your request. Try again after a short amount of time. If trying to spin up multiple instances, try one at a time.

### Instance terminates immediately

State of EC2 goes from Pending -> terminated

•	Reached EBS volume limit
•	EBS snapshot corrupt
•	Root EBS volume is encrypted and you don’t have permissions ot access KMS key for decryption
•	Instance stored-backed AMI that you used to launch the instance is missing a required part (image.part.xx file)

Go to EC2 dashboard, View instance attributes (click the cog on the top right and enable **State Transition Reason**)

View the State Transition Reason in the table below.

## Troubleshooting EC2 SSh issues

### Private key permissions

* Ensure private key (pem) file has 400 permissions (via `chmod 400 key.pem`) - otherwise you will get "Unprotected Private Key File error".

### Username for OS

Ensure the login username is correct (depends on the AMI) else you will get "Host key not found" error.

| OS/Distro            | Official AMI |
| -------------------- | ------------ |
| ssh                  | Username     |
| Amazon Linux         | ec2-user     |
| Ubuntu               | ubuntu       |
| Debian               | admin        |
| RHEL 6.4 and later   | ec2-user     |
| RHEL 6.3 and earlier | root         |
| Fedora               | fedora       |
| Centos               | centos       |
| SUSE                 | ec2-user     |
| BitNami              | bitnami      |
| TurnKey              | root         |
| NanoStack            | ubuntu       |
| FreeBSD              | ec2-user     |
| OmniOS               | root         |

### Connection timeout

* Incorrectly configured security group.
* CPU load of instance is too high.

## Instance Launch Types

### On Demand Instances: short workload, predictable pricing.

* Pay for what you use (billing per second after first minute)
* Highest cost, no upfront payment
* No long term commitment.
* Recommended for short-term, un-interrupted workloads, where you cant predict how the application will behave.

### Reserved instance: long workloads (>= 1 year).

* Up to 75% discount compared to On-demand.
* Pay upfront for what you use with long term commitment.
* Reservation period can be 1 or 3 years.
* Reserve a specific instance type.
* Recommended for steady state usage applications(e.g. database).


### Convertible Reserved Instances: long workloads with flexible instances.

* Convert and change instance type
* up to 54% discount


### Scheduled Reserved Instances: launch within reserved time window.

* Launch within time window you reserve

### Spot Instances: short workloads for cheap, can lose instances.

* Discount of up to 90% compared to On-demand.
* Bid a price and get the instance as long as its under the bid price.
* If you are outbid you will lose the instance.
* Price varies based on offer and demand.
* Reclaimed with a 2 minute notification warning when the spot price goes above your bid.
* Used for batch jobs, Big Data analysis, or workloads that are resilient to failure.
* Not great for critical jobs or databases.

### Dedicated hosts: book an entire physical server, control instance placement.

* Physical dedicated EC2 server.
* Full control of EC2 instance placement.
* Visibility into the underlying sockets / physical cores of the hardware.
* Allocated for your account with a 3 year period reservation.
* More expensive, more control.
* Userful for software licensing model (Bring Your Own License)
* Or companies that have strong regulatory or compliance needs.

### Dedicated instance: no other customers share hardware.

* Instance running on hardware thats dedicated to you.
* May share hardware with other instances in same account.
* No control over instance placement (can move hardware after Stop / Start).
  
![](images/dedicated-instances-vs-hosts.png)