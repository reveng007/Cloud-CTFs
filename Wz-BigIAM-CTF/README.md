# Full walkthrough of https://thebigiamchallenge.com/challenge/1

The challenges were totally based on scenarios where you would be given different IAM roles to assume and have to perform attack based on:

1. Misconfigured Simple Query Service (sqs)
2. Overly permissive S3 bucket Resource policy
3. Abusing greedy Expansion of AWS `sns:Endpoint`
4. Abusing AWS IDP (Cognito)
