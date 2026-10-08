 Question 2 - Risk Review

1. SSM Automation Policy (Annexure A & B)

What the policy is trying to do
Allow Azure DevOps pipeline to use AWS Systems Manager to run approved commands on EC2 instances in eu-west-1 for deployment.
Main Risks

 1. SSM SendCommand is sensitive
The policy allows:
ssm:SendCommand
AWS-RunShellScript
AWS-RunPowerShellScript

These can execute arbitrary commands on EC2. If pipeline credentials are compromised, attacker can run commands on approved servers.

2. EC2 tag conditions
Tags used:
AutomationAllowed=true
Application=Automated-Deployment-Dev

Tags limit targets but should not be the only control. If another identity can modify tags, it could make unintended instances eligible.

3. S3 permissions too broad
Allows:
s3:GetObject, s3:PutObject on arn:aws:s3:::*/*

This is broader than necessary. It allows access to all buckets, not just deployment bucket.

I would restrict to specific deployment bucket and prefix.

 4. General-purpose shell documents
AWS-RunShellScript and AWS-RunPowerShellScript provide high flexibility. If only controlled commands are needed, a restricted custom SSM document would reduce risk.

 5. Explicit deny statements
Deny list is useful as defence in depth but should not replace least privilege. Role should first be given only required permissions.

 6. Resource and account validation
ARNS, account IDs, regions should be validated against actual AWS environment before deployment.
Recommended Controls
- Restrict ssm:SendCommand to approved SSM documents
- Limit EC2 targets to required instances or tightly controlled tags
- Protect automation tags from unauthorized modification
- Restrict S3 to exact deployment bucket and prefix
- Use dedicated IAM role for deployment pipeline
- Avoid shared admin credentials
- Use short-lived credentials or OIDC instead of long-lived keys
- Require approval for production deployments
- Monitor SSM execution and CloudTrail API activity
- Keep explicit deny controls as defence in depth

 Possible Outcome if As-Is
Deployment may work but permissions are broader than necessary. If pipeline credentials were compromised, attacker could execute commands on approved instances and access S3 objects outside intended bucket.

 2. RDS Policies (Annexure C & D)

 Admin Role
Policy gives:
rds:* on Resource: *

Very broad - full RDS admin everywhere. Can delete, create, modify databases. Additional permissions for CloudWatch, SNS, Performance Insights increase scope.

Risk: Compromise of this role allows significant changes to RDS infrastructure.

Recommendation: Replace wildcard with only required RDS actions for admin tasks, scoped to specific DB ARNs.

 CRUB Role
Also provides:
rds:* on Resource: *

Too broad for read/write access role.

Important distinction:
AWS IAM permissions like rds:* control AWS RDS API operations. They do NOT give SQL permissions inside the database. SQL permissions like SELECT, INSERT, UPDATE, DELETE should be managed through database-native roles.

Recommendation: Use IAM for infrastructure permissions and database-native roles for data access.

 ReadOnly Role
Narrower - includes:
rds:Describe*, rds:ListTagsForResource
EC2 permissions for VPCs, Subnets, Security Groups

This gives management-plane visibility. But should not be described as having SQL SELECT access. AWS RDS read-only API access is different from read access to data inside database.

Other Risks
- Broad RDS Permissions: rds:* with Resource "*" makes least privilege hard to enforce
- SNS Publishing: sns:Publish should be restricted to specific topics
- Service-linked Roles: iam:CreateServiceLinkedRole should be restricted and monitored
- Performance Insights: limit to required actions and resources

 Recommended Controls
- Replace rds:* with specific RDS actions
- Restrict resources where AWS supports resource-level permissions
- Separate database admin from AWS infrastructure admin
- Use database-native roles for SQL permissions
- Use MFA for privileged access
- Keep break-glass admin account
- Monitor RDS create, modify, delete with CloudTrail
- Restrict SNS to named topics
- Use permission boundaries or SCPs as additional controls

Possible Outcome
Policies may work but provide more access than required. Reducing permissions lowers impact if account or credential is compromised. Main improvement is moving from wildcard to specific permissions based on actual tasks.
