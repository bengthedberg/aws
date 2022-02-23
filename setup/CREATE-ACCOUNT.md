# Create AWS Account

1. **Decide what email address will be used to register for the account**
   
   This seems very basic, but a little forethought can save you a headache down the road. 
The email used to create the account will be associated with the root user, who has full control of all AWS resources. 
Consider creating an email alias/distribution list that forwards mail to the group of people in the organization 
who should have root access. This is typically only a couple of trusted people. Doing so prevents the root account 
from being tied specifically to one person and gives you more flexibility if someone is not reachable during an emergency. 
Note also that the root user should [rarely](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#create-iam-users) 
be used as there are only a [few situations](https://docs.aws.amazon.com/general/latest/gr/aws_tasks-that-require-root.html)
when a root account is needed and it’s best to use it only when needed.

   Since many companies end up pursuing a multi-account strategy, consider using a naming convention for the email address that can be applied to other accounts. 
A good email address might be something like aws-prod@company.com or even something that includes the account alias (aws-[account-alias]@company.com).
   
   A little planning in this area can help you avoid situations where the root user is associated with some random email address that was used by someone who is no longer at the company.

2. **Create the account**
   
   Go through the process of [signing up](https://portal.aws.amazon.com/billing/signup) for an AWS account and enter any necessary billing information. 
   
   Note that if you are using multiple AWS accounts—a very common strategy—make use of [AWS Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_org_create.html) 
in order to create a hierarchy of accounts that are all linked together. With Organizations, you create 
one master account and then create child accounts underneath it without having to re-enter billing info. 
You benefit from consolidated billing, better reserved instance utilization, volume discounts, and policies 
that can be applied across multiple accounts.

3. **Turn on MFA for the root user**

   We strongly recommend turning on [Multi-Factor Authentication](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_mfa_enable_virtual.html) (MFA) 
   for the root user in all accounts as a best practice.

   It may be helpful to have all those on the distribution list present to scan the QR code shown in IAM to store it in 
their virtual MFA devices (e.g. Google Authenticator). Alternatively, you can save a screenshot of the QR code or put it 
on a mobile device, and store it in a safe place such as an office vault.

4. **Choose an AWS support plan**

   Make sure to look at what [support](https://aws.amazon.com/premiumsupport/) you may need from AWS and get it set up. If you have multiple accounts, 
   you will still need a support plan for each one. Less critical accounts, such as development accounts, 
   may not require the same plan as something like production.

5. **Activate IAM user access to billing information and billing alerts**

   By default, billing information is only made available to the root user. Since the root user should rarely be used, 
   you’ll need to [activate Identity and Access Management (IAM) user access](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/grantaccess.html)
   so that admins or billing groups can access
   what they need. Your IAM policies will dictate who has access to it. You’ll also want to 
   [enable billing alerts](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) so that 
   billing data is sent to CloudWatch and can later be used to send notifications if needed.
   
6. **Create a password policy**

   Before creating IAM users, it’s a good idea to create a [password policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_passwords_account-policy.html) in IAM. 
   You can specify criteria such as minimum password length, whether to allow users to change their own password, 
   and password expiration. Note that if you’re using a single sign-on service/identity provider, 
   this won’t apply to the users managed there.
   
7. **Create an Admin group**

   An IAM group should be created for those who will be AWS admins with full access to AWS functionality. 
   These users are frequently the same as those who have access to the root credentials, but may include a few others. 
   These users will be responsible for granting restricted access to others. Create the group and attach the 
   AdministratorAccess managed policy to it. Next, create the users that belong in this group and add them to it. 
   During the user creation process, choose the “AWS Management Console access” access type. In other words, grant 
   them access to the console and let them create access keys for programmatic access later if they need it. 
   A lot of people create access keys that aren’t needed until much later and it’s one more access mechanism floating 
   out there that doesn’t need to be. They frequently end up getting regenerated anyway because people lose track of them.
   
   You can now logout of the root account and start using your admin user that was just created.
   
8. **Choose an account alias**

   On the IAM dashboard, you can create an [account alias](https://docs.aws.amazon.com/IAM/latest/UserGuide/console_account-alias.html). 
   It becomes a label for your account and provides a memorable 
   login URL for the console. It’ll appear in Organizations and at the top of the console, which helps you to know which account you’re in.
   
9. **Turn on CloudTrail**

   Doing so ensures that all API activity—including use of the AWS console, CLI, SDKs, etc.—is tracked and stored in S3 and optionally 
   CloudWatch Logs too. It provides a full audit trail and should be turned on in every account. AWS now turns it on by default, 
   but you’ll only get 90 days of activity. Create a custom “trail” and configure it to track management events in all regions and enable log file validation.
   
10. **Decide which regions you’ll use first**

    AWS resources are placed in regions. Decide which [regions](https://aws.amazon.com/about-aws/global-infrastructure/) you’ll be using at first. 
    You’ll want to take into account service availability, pricing, and data sovereignty requirements.
    
11. **Turn on AWS Config**
   
    For each region you use, turn on [AWS Config](https://aws.amazon.com/config/). It will keep an inventory of resources you create and changes made to them. 
   This helps you diagnose problems by finding out what happened when something stops working as expected. It has many other features, 
   but simply turning it on is a good start.
   
12. **Creating a billing alarm**

    If you need to keep an eye on AWS spend, consider creating a [billing alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html).
   
13. **Create the remaining groups and users**

    Create the remaining groups and users for everyone else who will be using AWS. Grant [least privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege)
    access so that people only have access to what is needed. These groups may include developers, DBAs, network engineers, etc. 
    If these users exist in a single sign-on service/identity provider or if you will have multiple AWS accounts, you’ll
    want to setup the necessary roles to enable access and to allow cross-account switching.

14. **Implement additional security mechanisms**

    Depending on your needs and compliance requirements, you might want to consider adding additional security measures by setting up 
[GuardDuty](https://aws.amazon.com/guardduty/) and [Macie](https://aws.amazon.com/macie/), 
requiring MFA to be used by all users, creating KMS keys for encryption, deploying security benchmarks such as
[CIS](https://aws.amazon.com/quickstart/architecture/accelerator-cis-benchmark/), creating custom AWS Config rules, and leveraging CloudWatch Events in order to be notified of important events happening in your account.

15. **Decide how you’ll be creating resources**

    Decide whether resources will be managed via the console or defined as code with something like [CloudFormation](https://aws.amazon.com/cloudformation/). Creating infrastructure as code has some significant advantages. Making this decision up front will govern how things are created and modified as you move forward. Everyone in your organization needs to be on the same page here.

16. **Start building!**

    With this initial foundation in place, you should be ready to start building on AWS. With an ever-growing list of services, the sky’s the limit. Enjoy!

