# Control Tower

## Setup

It is easiest to start with a [fresh AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) and follow the instructions in [Getting Started with AWS Control Tower](https://docs.aws.amazon.com/controltower/latest/userguide/getting-started-with-control-tower.html).

**Note:** While not necessary, out of habit, I first followed the steps in, [Security Best Practices in IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) to secure the fresh AWS account including creating an IAM Admin account.

AWS Control Tower sets up the multi-account environment, aka. landing zone, with an AWS Organization as shown:

**Note:** Going forward, we will use the term landing zone exclusively.

Key of Control Tower:
* With AWS, using multiple AWS Accounts is the best way to create comprehensive security/resource boundaries between teams, i.e., we should think of AWS Accounts as resource containers
* The AWS Organization master account should only contain resources related to setting up AWS Organizations itself
* Your AWS Organization should include a Security Organizational Unit (OU) with a Log Archive account aggregating all the logs from all the accounts in the AWS Organization; acting as a single source of truth. It should also have a Security Tooling account to house automated auditing tools
* Your AWS Organization should include an Infrastructure OU with a Network account to hold shared networking resources, e.g,. shared VPCs
* Your AWS Organization should include a Sandbox OU to hold individual developer accounts with fixed spending limits to be used for experimentation
* You AWS Organization should include a Workloads OU to hold accounts that mirror your development life cycle, e.g., Dev, Pre-Prod (aka Staging), and Prod
* While one can build a properly configured multi-account environment using a number of AWS services centered around AWS Organizations, AWS Control Tower is a no-cost* service that accomplishes this through a managed set of CloudFormation templates tied together with a user interface

***Note:** There is no additional charge to use AWS Control Tower. However, when you set up AWS Control Tower, you will begin to incur costs for AWS services configured to set up your landing zone and mandatory guardrails.

### Observations:

* The Core OU contains two newly created Member accounts; each with their respective Root users. Comparing this to the recommended landing zone, this Core OU is equivalent to the Security OU with the two recommended accounts

* The original account becomes the AWS Organization Master account in the Root OU. In addition to the original Root and IAM Admin users, AWS Control Tower creates an SSO configuration with a single SSO Admin user (more on this later)
For good measure, we go ahead and create initial passwords for the Root user for each of the Log Archive and Audit accounts; we have to use the forgot password feature to do this.

AWS Control Tower uses AWS CloudFormation to create a number of AWS resources during the landing zone setup. 
We do not need to know much about these AWS CloudFormation resources other than not to touch them.

## Concepts

### Guardrails

While learning much of AWS Control Tower is about understanding how it configures other AWS services, e.g., AWS SSO, Guardrails are distinct to AWS Control Tower.
A guardrail is a high-level rule that provides ongoing governance for your overall AWS environment. A guardrail applies to an entire organizational unit (OU), and every AWS account within the OU is affected by the guardrail. Therefore, when users perform work in any AWS account in your landing zone, they’re always subject to the guardrails that are governing their account’s OU.

There are two types of Guardrails:

* Prevention — A preventive guardrail ensures that your accounts maintain compliance, because it disallows actions that lead to policy violations.

* Detection — A detective guardrail detects noncompliance of resources within your accounts, such as policy violations, and provides alerts through the dashboard.
  
There are 22 Mandatory, 11 Strongly Recommended, and 5 Elective Guardrails; a mix of both Prevention and Detection Guardrails. When you create a new landing zone, only the mandatory guardrails are enabled by default.

### Account Factory

The Account Factory is the second of two features that are distinct to AWS Control Tower:

An Account Factory is a configurable account template that helps to standardize the provisioning of new accounts with pre-approved account configurations. AWS Control Tower offers a built-in Account Factory that helps automate the account provisioning workflow in your organization.

