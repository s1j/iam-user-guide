# Example Scenarios for Using Access Data<a name="access_policies_access-advisor-example-scenarios"></a>

You can use service last accessed data to make decisions about the permissions that you grant to your IAM entities \(users or roles\)\. For more information, see [Reducing Permissions Using Service Last Accessed Data](access_policies_access-advisor.md)\. 

**Note**  
Before you view the access data for a resource in IAM, make sure you understand the reporting period, reported entities, and the evaluated policy types for your data\. For more details, see [Things to Know](access_policies_access-advisor.md#access_policies_access-advisor-know)

It’s up to you as an IAM administrator to balance the accessibility and least privilege that’s appropriate for your organization\. 

## Using Data to Reduce Permissions for a Group<a name="access-advisor-sample-reduce-permissions-group"></a>

You can use service last accessed data to reduce group permissions to include only those services that your users need\. This method is an important step in [granting least privilege](best-practices.md#grant-least-privilege) at a service level\.

For example, Paulo Santos is the administrator in charge of defining AWS user permissions for Example Corp\. This company just started using AWS, and the software development team has not yet defined what AWS services they will use\. Paulo wants to give the team permission to access only the services they need, but since that is not yet defined, he temporarily gives them power\-user permissions\. Then he uses service last accessed data to reduce the group's permissions\.

Paulo creates a managed policy named `ExampleDevelopment` using the following JSON text\. He then attaches it to a group named `Development` and adds all of the developers to the group\.

```
{

    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "FullAccessToAllServicesExceptPeopleManagement",
            "Effect": "Allow",
            "NotAction": [
                "iam:*",
                "organizations:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "RequiredIamAndOrgsActions",
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceLinkedRole",
                "iam:DeleteServiceLinkedRole",
                "iam:ListRoles",
                "organizations:DescribeOrganization"
            ],
            "Resource": "*"
        }
    ]
}
```

Paulo decides to wait for 90 days before he [views the service last accessed data](access_policies_access-advisor-view-data.md#access_policies_access-advisor-viewing) for the `Development` group using the AWS Management Console\. He views the list of services that the group members accessed\. He learns that the users accessed five services within the last week: AWS CloudTrail, Amazon CloudWatch Logs, Amazon EC2, AWS KMS, and Amazon S3\. They accessed a few other services when they were first evaluating AWS, but not since then\.

Paulo decides to reduce the policy permissions to include only those five services and the required IAM and Organizations actions\. He edits `ExampleDevelopment` policy using the following JSON text\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "FullAccessToListedServices",
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "kms:*",
                "cloudtrail:*",
                "logs:*",
                "ec2:*"
            ],
            "Resource": "*"
        },
        {
            "Sid": "RequiredIamAndOrgsActions",
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceLinkedRole",
                "iam:DeleteServiceLinkedRole",
                "iam:ListRoles",
                "organizations:DescribeOrganization"
            ],
            "Resource": "*"
        }
    ]
}
```

To further reduce permissions, Paulo can view the account's events in AWS CloudTrail **Event history**\. There he can view detailed event information that he can use to reduce the policy's permissions to include only the actions and resources that the developers need\. For more information, see [Viewing CloudTrail Events in the CloudTrail Console](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events-console.html) in the *AWS CloudTrail User Guide*\.

## Using Data to Reduce Permissions for an IAM User<a name="access_policies_access-advisor-reduce-permissions-users"></a>

You can use service last accessed data to reduce the permissions for an individual IAM user\.

For example, Martha Rivera is an IT administrator responsible for ensuring that people in her company do not have excess AWS permissions\. As part of a periodic security check, she reviews the permissions of all IAM users\. One of these users is an application developer named Nikhil Jayashankar, who previously filled the role of a security engineer\. Because of the change in job requirements, Nikhil is a member of both the `app-dev` group and the `security-team` group\. The `app-dev` group for his new job grants permissions to multiple services including Amazon EC2, Amazon EBS, Auto Scaling, Route 53, and Elastic Transcoder\. The `security-team` group for his old job grants permissions to IAM and CloudTrail\.

As an administrator, Martha signs into the IAM console and chooses **Users**, chooses the name `nikhilj`, and then chooses the **Access Advisor** tab\.

Martha reviews the **Last Accessed** column and notices that Nikhil has not recently accessed IAM, CloudTrail, Route 53, Amazon Elastic Transcoder, and a number of other AWS services\. Within her company, Martha confirms that Nikhil has no business need to access IAM and CloudTrail anymore because he is no longer a member of the internal security team\. 

Martha is now ready to act on the service last accessed data\. However, unlike the group in the previous example, an IAM user like `nikhilj` might be subject to multiple policies and be a member of multiple groups\. Martha must proceed with caution to avoid inadvertently disrupting access for `nikhilj` or other group members\. In addition to learning what access Nikhil should have, she must determine *how* he is receiving these permissions\. 

Martha chooses the **Permissions** tab, where she views which policies are attached directly to `nikhilj` and those attached from a group\. She expands each policy and views the policy summary to learn which policy allows access to the services that Nikhil is not using:
+ IAM – The `IAMFullAccess` AWS managed policy is attached directly to `nikhilj` and attached to the `security-team` group\.
+ CloudTrail – The `AWSCloudTrailReadOnlyAccess` AWS managed policy is attached to the `security-team` group\.
+ Route 53 – The `App-Dev-Route53` customer managed policy is attached to the `app-dev` group\.
+ Elastic Transcoder – The `App-Dev-ElasticTranscoder` customer managed policy is attached to the `app-dev` group\.

Martha decides to remove the `IAMFullAccess` AWS managed policy that is attached directly to `nikhilj`\. She also removes Nikhil’s membership to the `security-team` group\. These two actions remove the unnecessary access to IAM and CloudTrail\. 

Nikhil's permissions to access to Route 53 and Elastic Transcoder are granted by the `app-dev` group\. Although Nikhil isn't using those services, other members of the group might be\. Martha reviews the service last accessed data for the `app-dev` group and learns that several members recently accessed Route 53, but no group members have accessed Elastic Transcoder in the last year\. She removes the `App-Dev-ElasticTranscoder` customer managed policy from the group\.

Martha then reviews the service last accessed data for the `App-Dev-ElasticTranscoder` customer managed policy\. She learns that the policy is not attached to any other IAM identities\. She investigates within her company to make sure that the policy will not be needed in the future, and then she deletes it\.

## Using Data Before Deleting Resources<a name="access-advisor-sample-delete-resources"></a>

You can use service last accessed data before you delete an IAM resource to make sure that a certain amount of time has passed since someone last used the resource\. This applies to users, groups, roles, and policies\.

## Using Data Before Editing Policies<a name="access-advisor-sample-edit-policies"></a>

You can review the service last accessed data for an IAM identity \(user, group, or role\), or for a policy before editing a policy that affects that resource\. This is important because you don't want to remove access for someone that is using it\.

For example, Arnav Desai is a developer and AWS administrator for Example Corp\. When his team started using AWS, they gave all developers power\-user access that allowed them full access to all services except IAM and Organizations\. As a first step towards [granting least privilege](best-practices.md#grant-least-privilege), Arnav wants to use the AWS CLI to review the managed policies in his account\. 

To do this, Arnav first lists the customer managed permissions policies in his account that are attached to an identity, using the following command:

```
aws iam list-policies --scope Local --only-attached --policy-usage-filter PermissionsPolicy
```

From the response, he captures the ARN for each policy\. Arnav then generates a service last accessed data report for each policy using the following command\.

```
aws iam generate-service-last-accessed-details --arn arn:aws:iam::123456789012:policy/ExamplePolicy1
```

From that response, he captures the ID of the generated report from the `JobId` field\. Arnav then polls the following command until the `JobStatus` field returns a value of `COMPLETED` or `FAILED`\. If the job failed, he captures the error\.

```
aws iam get-service-last-accessed-details --job-id 98a765b4-3cde-2101-2345-example678f9
```

When the job has a status of `COMPLETED`, Arnav parses the contents of the JSON\-formatted `ServicesLastAccessed` array\.

```
 "ServicesLastAccessed": [
        {
            "TotalAuthenticatedEntities": 1,
            "LastAuthenticated": 2018-11-01T21:24:33.222Z,
            "ServiceNamespace": "dynamodb",
            "LastAuthenticatedEntity": "arn:aws:iam::123456789012:user/IAMExampleUser",
            "ServiceName": "Amazon DynamoDB"
        },

        {
            "TotalAuthenticatedEntities": 0,
            "ServiceNamespace": "ec2",
            "ServiceName": "Amazon EC2"
        },

        {
            "TotalAuthenticatedEntities": 3,
            "LastAuthenticated": 2018-08-25T15:29:51.156Z,
            "ServiceNamespace": "s3",
            "LastAuthenticatedEntity": "arn:aws:iam::123456789012:role/IAMExampleRole",
            "ServiceName": "Amazon S3"
        }
    ]
```

From this information, Arnav learns that the `ExamplePolicy1` policy allows access to three services, Amazon DynamoDB, Amazon S3, and Amazon EC2\. The IAM user named `IAMExampleUser` last attempted to access DynamoDB on November 1, and someone used the `IAMExampleRole` role to attempt to access Amazon S3 on August 25\. There are also two more entities that attempted to access Amazon S3 in the last year\. However, nobody has attempted to access Amazon EC2 in the last year\.

This means that Arnav can safely remove the Amazon EC2 actions from the policy\. Arnav wants to review the current JSON document for the policy\. First, he must determine the version number of the policy using the following command\.

```
aws iam list-policy-versions --policy-arn arn:aws:iam::123456789012:policy/ExamplePolicy1
```

From the response, Arnav collects the current default version number from the `Versions` array\. He then uses that version number \(`v2`\) to request the JSON policy document using the following command\.

```
aws iam get-policy-version --policy-arn arn:aws:iam::123456789012:policy/ExamplePolicy1 --version-id v2
```

Arnav stores the JSON policy document returned in the `Document` field of the `PolicyVersion` array\. Within the policy document, Arnav searches for actions with in the `ec2` namespace\. If there are no actions from other namespaces remaining in the policy, then he detaches the policy from the affected identities \(users, groups, and roles\) and then deletes the policy\. In this case, the policy does include the Amazon DynamoDB and Amazon S3 services, so Arnav removes the Amazon EC2 actions from the document and saves his changes\. He then uses the following command to update the policy using the new version of the document and to set that version as the default policy version\.

```
aws iam create-policy-version --policy-arn arn:aws:iam::123456789012:policy/ExamplePolicy1 --policy-document file://UpdatedPolicy.json --set-as-default
```

The `ExamplePolicy1` policy is now updated to remove access to the unnecessary Amazon EC2 service\.

## Other Scenarios<a name="access-advisor-scenarios-other"></a>

Information about when an IAM resource \(user, group, role, or policy\) last attempted to access a service can help you when you complete any of the following tasks:
+ **Policies** – [Editing an existing customer\-managed or inline policy to remove permissions](access_policies_manage-edit.md)
+ **Policies** – [Converting an inline policy to a managed policy and then deleting it](best-practices.md#best-practice-managed-vs-inline)
+ **Policies** – [Adding an explicit deny to an existing policy](reference_policies_evaluation-logic.md#AccessPolicyLanguage_Interplay)
+ **Policies** – [Detaching a managed policy from an identity \(user, group, or role\)](access_policies_manage-attach-detach.md#detach-managed-policy-console)
+ **Policies** – [Deleting a managed policy \(this also detaches the policy from identities\)](access_policies_manage-delete.md)
+ **Entities** – [Set a permissions boundary to control the maximum permissions that an entity \(user or role\) can have](access_policies_manage-attach-detach.md)
+ **Groups** – [Removing users from a group](id_groups_manage_add-remove-users.md)
+ **Groups** – [Deleting a group](id_groups_manage_delete.md)
+ **Users** – [Deleting a user](id_users_manage.md#id_users_deleting)
+ **Roles** – [Deleting a role](id_roles_manage_delete.md)