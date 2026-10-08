Question 3 - Required SCP

 Objective
The requirement is to restrict AWS workload activity to:
eu-west-1

The SCP uses the AWS condition key:
aws:RequestedRegion

The policy denies actions when the requested region is not eu-west-1.

Policy
The SCP is stored in:
region-restriction.json / scp.json

How It Works
The policy uses:
Effect: Deny
together with:
StringNotEquals for aws:RequestedRegion

The requested region must therefore be:
eu-west-1 for normal regional AWS API actions covered by the SCP.

Why NotAction Is Used
Some AWS services are global services and do not operate through a normal regional endpoint.

The policy therefore uses NotAction to avoid unintentionally blocking important global services.

The exception list should still be reviewed against the organisation's actual AWS environment. Only services that genuinely need to operate globally should be excluded.

Deployment Approach
I would deploy the SCP in stages.

Step 1 - Test: Apply to non-production account or test OU first.

Step 2 - Review Existing Activity: Review CloudTrail and identify workloads making API requests outside eu-west-1.

Step 3 - Check Dependencies: Confirm required AWS services and global services are not affected.

Step 4 - Deploy to Controlled Account: Attach to controlled test account and perform normal activities.

Step 5 - Production Deployment: After testing, attach to production OU.

Step 6 - Monitoring: Monitor CloudTrail for denied requests and investigate legitimate workloads affected.

Any required exception should go through a controlled change process.

## Important Security Note
An SCP is a guardrail. It does not grant permissions. An IAM user or role still needs an Allow permission for an operation to succeed. The SCP only limits the maximum permissions.

Testing
Before production deployment, test:
- EC2 deployment in eu-west-1
- S3 operations in eu-west-1
- RDS operations in eu-west-1
- Attempts to create resources in another region
- IAM administration
- Route 53 operations
- CloudFront operations
- STS operations

Expected result is that regional workload actions outside eu-west-1 are denied while required global services continue to operate.
