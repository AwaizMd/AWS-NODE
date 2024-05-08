# AWS IAM

## Topics:-

1. IAM Introduction
2. IAM Best Practices
3. Different Types of Policies
4. Policy Evaluation
5. Identity Federation
6. STS API Methods

**IAM Introduction**

AWS IAM is a global service that you can use to manage access to AWS services and resources. Access can be granted to IAM users, groups and roles using permission policies.


**Long Term Credentials**

When you create an IAM user, you can assign him/her Access keys and User passwords which are considered to be long term credentials. Long credentials do not get expired. So never expose them to anybody else!

**Temporary Credentials**
IAM Role, on the other hand, provides short term temporary credentials to whoever assumes the role. These credentials get expired within 15mins to 12 hours and need to be refreshed.

IAM Role can be assumed by an IAM User, Group, Another Role in the same or different AWS Accounts. It can also be assumed by AWS services like EC2, Lambda, etc… or it could be federated users like Users in Active Directory of an on-premises organization, Web identities (Facebook users, Twitter Users, etc..)

**IAM Role**
An IAM Role has two main parts. Permission policy and Trust policy. (Theses policies are JSON objects) The permission policy describes the permission of the role. Trust policy describes who can assume that role. (E.g. EC2 service, Lambda service, A specific AWS Account, etc…)

Once the IAM Role is assumed by an allowed entity, AWS STS (Security Token Service) provides the temporary security credentials to the entity. The temporary security credentials contain the following information.

* Session Token
* Access Key ID
* Secret Access Key
* Expiration

When an IAM user from a different AWS Account assumes an IAM role of another account (E.g. Using the Switch Account feature in AWS Console or using the API) the temporary credentials from STS will replace his/her existing credentials of the trusted account — The account he is from.

**IAM Best Practices**
Following are the best practices for using IAM service. They must be thoroughly followed as IAM is the centralized service for Security in the AWS platform.

* Lock Away Your AWS Account Root User Access Keys
* Create Individual IAM Users
* Use Groups to Assign Permissions to IAM Users
* Grant Least Privilege
* Get Started Using Permissions with AWS Managed Policies
* Use Customer Managed Policies Instead of Inline Policies
* Use Access Levels to Review IAM Permissions
* Configure a Strong Password Policy for Your Users
* Enable MFA for Privileged Users
* Use Roles for Applications That Run on Amazon EC2 Instances
* Use Roles to Delegate Permissions
* Do Not Share Access Keys
* Rotate Credentials Regularly
* Remove Unnecessary Credentials
* Use Policy Conditions for Extra Security
* Monitor Activity in Your AWS Account
* For more information about the best practices, follow this link to the AWS documentation.

**Different Types of IAM Policies**
There are three major types of IAM policies used to control access in AWS.

* Service Control Policies
* Identity-Based Policies
* Resource-Based Policies
* Service Control Policies
* Service Control Policies (SCPs) are used to manage all the AWS Accounts in your AWS Organization. SCPs can be applied at individual AWS accounts level or at    
  Organizational Units (OUs) level inside your AWS Organization to control the maximum permission. (Applying the SCP to an OU means applying the same policy to all the AWS accounts under that OU).

In order to apply SCPs, you must enable “All Features” in the organization. SCPs aren’t available if your organization has enabled only the consolidated billing features.

If multiple SCPs affects an AWS Account, the maximum permission is determined by the overlap.

![image](https://github.com/AwaizMd/AWS-NODE/assets/72355688/8631be1f-f551-4f9c-9ca1-d9040d5b7653)


You can either use Whitelisting or Blacklisting of permissions when using SCPs to control maximum permissions. Blacklisting permissions provides less admin overhead when managing many AWS accounts.

SCPs are not a replacement to AWS IAM policies. They are only used to control what a specific AWS account can or cannot do. IAM policies are still required to manage access for different entities within the AWS account.

If you blacklist a resource/service for an AWS account using SCPs, that cannot be accessed even to the root user of that account.

**Identity-Based Policies**

Identity-based policies are attached to an IAM user/group or a role. It specifies what the identity (user/group/role) can do. For example, you can allow an IAM user “John” to read from a certain S3 bucket (E.g. mybucket) and Deny spinning up any EC2 instances.

```js
{
   "Version": "2012-10-17",
   "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": ["arn:aws:s3:::mybucket/*"]
    },
   {
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*"   
   }]
}
```

We can set up quite granular access controls using Identity-Based Policies. For Identity-based policies, it’s not mandatory to mention the “Principal” — (to whom the policy gets applied to), as it is implied from the attached identity. Identity-based policies can be managed( Managed by AWS or Customer. These can be reused among many identities) or inlinepolicies (Applied only to a certain identity).

**Resource-based Polices**
Resource-based policies are applied to a resource rather than to an identity. You can attach resource-based policies to S3 buckets, SQS queues, etc… With resource-based policies, you can specify who has access to the resource and what actions they can perform on it. Resource-based policies are inline only, not managed.

```js
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::mybucket/*"]
    }
  ]
}
```
Above policy can be applied to the “mybucket” s3 bucket to Allow only read-only access to the content of the s3 bucket to anybody. Note that we must specify “Principal” — to whom the policy gets applied to. In the above policy, it’s anyone.

**Policy Evaluation**
When a principal tries to access an AWS resource, there could be multiple types of policies applied. (SCPs, Identity-based policies, resource-based policies). AWS evaluates all these policies before allow or deny access to the resource for the principle. The main logic behind policy evaluation is as follows.

* The decision starts at Deny
* Evaluate all the applicable policies to the resource and to the principal
* Explicit “DENY” policies override any Explicit “Allow” policies
* If there is no explicit “DENY” polices defined but only Explicit “Allow” polices, the access is granted to the resource
* If there are no explicit policies defined for the resource (ALLOW or DENY) implicit “DENY” is applied by default. So the access to the resource is denied.



![image](https://github.com/AwaizMd/AWS-NODE/assets/72355688/4c04a167-b71d-4131-a535-2e947d94aadd)



**Evaluating Policies Within a Single Account**
Following steps take place when an IAM User/Role try to access a resource within a single AWS account.

* Check if Service Control Policies (SCPs) permits the action. If not Deny access immediately.
* If SCP allows, then check the Identity-based policy(IAM policy) attached to the identity and the resource-based policy attached to the resource. For example, if IAM user, John is accessing an S3 bucket, check IAM policy attached to John and also check resource-based policy attached to that S3 bucket.
* If either Identity-based policy OR resource-based policy allows the action, Allow John accessing the S3 bucket.

**Evaluating Cross Account Polices**
When a user from a different AWS account wants to access a resource of another AWS account then the policies are evaluated as follows.

* Check if Service Control Policies (SCPs) permits the action. If not Deny access immediately.
* If SCP allows, then check the Identity-based policy(IAM policy) attached to the identity and the resource-based policy attached to the resource.
* If BOTH Identity-based policy AND resource-based policy allows the action, Allow the user to access the resource.

**Identity Federation**
You can create up to 5,000 IAM users per AWS account. Imagine your organization has 10,000 users who need access to an AWS account. First of all, it is not allowed to create 10,000 IAM users. Even if it was possible, administrating 10,000 IAM users can be a nightmare. This is where the identity federation comes into the play.

Identity federation means outsourcing identity management to an external party. It could be google, facebook, twitter, on-premises active directory. Google, Facebook, Twitter, etc.. are examples of Web Identities. Active Directory, on the other hand, is an example of a Co-operate identity manager.

Identity federation is based on the trust between AWS and the External Identity provider(Web or Co-operate). The process of AWS identity federation is as follows.

* Create an AWS IAM role that can be assumed by a federated identity
* Provide required permission to that IAM Role.
* Configure AWS and the external IDP (Identity Provider)
* If it is web identity like facebook or twitter, create an app on facebook or twitter and configure app ids and secrets in AWS
* If it is corporate identity like Active Directory, upload metadata document or link metadata URL in AWS. Do the necessary configuration at the on-premises side as well.
* When a user wants to access a resource in AWS, direct him/her to the login page of web identity (login with facebook, etc…) or the login page of Active Directory.
* Once the user has successfully logged in, send a confirmation token to AWS. In the case of web identity, it could be id_token, If it is Active Directory it could be SAML assertion.
* AWS will trust the assertion and allow the user to assume the IAM Role thereby access the authorized resources in AWS.
* STS API Methods
* AWS IAM Roles provides temporary credentials to whoever authorized to assume an IAM Role. Temporary credentials are supplied by AWS Security Token Service (STS) by evaluating the permission polices attached to the role.

There are Five main API methods provided by AWS STS.

* AssumeRole
* AssumeRoleWithWebIdentity
* AssumeRoleWithSAML
* GetFederationToken
* GetSessionToken

**AssumeRole**
This is typically used for cross-account access. The AWS account who wants to share a resource with another AWS account can create an IAM Role. Add the permission policy and the trust policy to the role. We can refer the other AWS account id in the trust policy. So the other account can use “Switch Role” feature in the AWS Console to enter the role name and account id of the resource sharing account and get access to the resource

**AssumeRoleWithSAML**
This API call returns a set of temporary security credentials for users who have been authenticated via a SAML authentication response. Typically used for Active Directory federation.

**AssumeRoleWithWebIdentity**
This API call returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with a web identity provider. If you have a mobile app where the users need to access an AWS resource (E.g. Upload profile photo to a S3 bucket) you can set up the IAM role, configure web identity and allow all authenticated (With facebook, google, etc..) federated users to assume that role.

**GetFederationToken**
GetFederationToken API call needs Long Term Credentials of an IAM User instead of IAM Role. Because of that, use this API method only in a safe environment where the long term credentials can be stored.

It returns a set of temporary security credentials (consisting of an access key ID, a secret access key, and a security token) for a federated user. A typical use is in a proxy application that gets temporary security credentials on behalf of distributed applications inside a corporate network.

**GetSessionToken**
Typically used to receive temporary credentials for Untrusted environments. It returns a set of temporary credentials for an AWS account or IAM user. The user must already be an IAM user. When he wants to access AWS resource in untrusted environments, he can use MFA to protect calls to AWS with GetSessionToken call.

