# 3.WIZ Cloud Hunting CTF:

## 3. Deeper into the trail

-------

Bingo — you've tracked down the compromised IAM user: Moe.Jito. Keep digging through the CloudTrail logs.

Follow the attacker's footsteps and find the machine that was compromised and leveraged for lateral movement.

-------

Things we know till now:
```
1. assumed role Name: S3Reader and roleSessionName: drinks
2. IAM role Name: S3Reader
3. userIdentity_ARN: arn:aws:iam::509843726190:user/Moe.Jito
```

1. 
```sql
SELECT 
    EventTime,
    EventName,
    userIdentity_userName,
    requestParameters,
    userIdentity_ARN,
    EventSource
FROM cloudtrail 
WHERE userIdentity_ARN like 'arn:aws:iam::509843726190:user/Moe.Jito'
ORDER BY EventTime DESC
```

<img width="1940" height="377" alt="Clipboard_2025-09-08-03-35-19" src="https://github.com/user-attachments/assets/0d43d339-eecf-42ec-8915-302e72786fc2" />

2. 
```sql
SELECT 
    EventTime,
    EventName,
    userIdentity_userName,
    requestParameters,
    userIdentity_ARN,
    EventSource
FROM cloudtrail 
ORDER BY EventTime DESC
```

<img width="2707" height="308" alt="Clipboard_2025-09-08-04-29-04" src="https://github.com/user-attachments/assets/f5b1ca76-bd32-4c1e-ac1c-b18ac17fcc30" />

- After scrolling down, I was able to see the instance with EventName, "UpdateFunctionCode20150331v2" and the EventSource being "lambda.amazonaws.com".
- Sometimes, Lambda function is a great way to inject malicious code and cause :
  - persistence (when lambda has periodic triggers)
  - privilege Escalation
  - and perform lateral movement (when having overly permissive role and can attack lambda functions or specifically lambda authorizers to target endpoints behind an API Gateway and access resources).

#### Privilege Escalation Examples via exploiting lambda functions:
- https://github.com/RhinoSecurityLabs/cloudgoat/blob/master/cloudgoat/scenarios/aws/vulnerable_lambda/README.md

#### Persistence via exploiting lambda functions:
- https://docs.aws.amazon.com/lambda/latest/api/API_UpdateFunctionCode.html
- https://research.splunk.com/cloud/211b80d3-6340-4345-11ad-212bf3d0d111/

#### Lateral Movement Examples via exploiting lambda functions:
- https://github.com/RhinoSecurityLabs/cloudgoat/blob/master/cloudgoat/scenarios/aws/ec2_ssrf/README.md
- https://www.tenchisecurity.com/en/insights-news/the-fault-in-our-stars

3. Tried to narrow down the hunt:
```sql
SELECT
    EventTime,
    EventName,
    userIdentity_userName,
    requestParameters,
    userIdentity_ARN,
    EventSource
FROM cloudtrail 
WHERE userIdentity_ARN LIKE '%i-%'  -- EC2 instances
ORDER BY EventTime DESC;
 
Or,
 
SELECT
    EventTime,
    EventName,
    userIdentity_userName,
    requestParameters,
    userIdentity_ARN,
    EventSource
FROM cloudtrail 
WHERE userIdentity_ARN LIKE '%i-%'  -- EC2 instances
   OR EventSource = 'rds.amazonaws.com'  -- RDS
   OR EventSource = 'lambda.amazonaws.com'  -- Lambda
   OR EventSource = 'ecs.amazonaws.com'  -- ECS containers
   OR EventSource = 'eks.amazonaws.com'  -- Kubernetes ; More can be added for more complex scenarios`
ORDER BY EventTime DESC;
```

<img width="2530" height="530" alt="Clipboard_2025-09-08-07-44-55" src="https://github.com/user-attachments/assets/36a35acc-5045-4fca-9e58-91abde7ac1d2" />

> Ans: `i-0a44002eec2f16c25`



