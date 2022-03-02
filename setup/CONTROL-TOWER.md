# Control Tower

## Setup

It is easiest to start with a fresh AWS account and follow the instructions in Getting Started with AWS Control Tower.

**Note:** While not necessary, out of habit, I first followed the steps in, Security Best Practices in IAM to secure the fresh AWS account including creating an IAM Admin account.


AWS Control Tower sets up the multi-account environment, aka. landing zone, with an AWS Organization as shown:

**Note:** Going forward, we will use the term landing zone exclusively.



Observations:

* The Core OU contains two newly created Member accounts; each with their respective Root users. Comparing this to the recommended landing zone, this Core OU is equivalent to the Security OU with the two recommended accounts

* The original account becomes the AWS Organization Master account in the Root OU. In addition to the original Root and IAM Admin users, AWS Control Tower creates an SSO configuration with a single SSO Admin user (more on this later)
For good measure, we go ahead and create initial passwords for the Root user for each of the Log Archive and Audit accounts; we have to use the forgot password feature to do this.

AWS Control Tower uses AWS CloudFormation to create a number of AWS resources during the landing zone setup. 
We do not need to know much about these AWS CloudFormation resources other than not to touch them.


## Guardrails

While learning much of AWS Control Tower is about understanding how it configures other AWS services, e.g., AWS SSO, Guardrails are distinct to AWS Control Tower.
A guardrail is a high-level rule that provides ongoing governance for your overall AWS environment. A guardrail applies to an entire organizational unit (OU), and every AWS account within the OU is affected by the guardrail. Therefore, when users perform work in any AWS account in your landing zone, they’re always subject to the guardrails that are governing their account’s OU.

There are two types of Guardrails:

* Prevention — A preventive guardrail ensures that your accounts maintain compliance, because it disallows actions that lead to policy violations.

* Detection — A detective guardrail detects noncompliance of resources within your accounts, such as policy violations, and provides alerts through the dashboard.
  
There are 22 Mandatory, 11 Strongly Recommended, and 5 Elective Guardrails; a mix of both Prevention and Detection Guardrails. When you create a new landing zone, only the mandatory guardrails are enabled by default.

## Account Factory

The Account Factory is the second of two features that are distinct to AWS Control Tower:

An Account Factory is a configurable account template that helps to standardize the provisioning of new accounts with pre-approved account configurations. AWS Control Tower offers a built-in Account Factory that helps automate the account provisioning workflow in your organization.

