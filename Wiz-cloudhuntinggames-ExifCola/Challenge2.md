# 2.WIZ Cloud Hunting CTF:

## 2. Follow, follow the trail

----

So you have managed to validate FizzShadows' claim and track the IAM role that has exfiltrated ExfilCola's recipe. You've been granted access to the cloudtrail table.

Follow the trail of the S3Reader. Who used it?

----


1. Tried this, no result found:
```sql
SELECT 
	EventTIme,
	EventName,
	IP,
	requestParameters,
	userIdentity_ARN,
	EventSource
FROM cloudtrail where IP like '37.19.199.135' and requestParameters like '%soda-vault%' ORDER BY EventTime DESC
```

<img width="1957" height="410" alt="Clipboard_2025-09-08-02-15-06" src="https://github.com/user-attachments/assets/6b877796-d839-4f30-b51e-f67fdcbc33d4" />

2. Removed IP:
```sql
SELECT 
	EventTIme,
	EventName,
	requestParameters,
	userIdentity_ARN,
	EventSource
FROM cloudtrail where requestParameters like '%soda-vault%' ORDER BY EventTime DESC
```

<img width="2515" height="621" alt="Clipboard_2025-09-08-02-40-45" src="https://github.com/user-attachments/assets/31ce5124-aaec-4926-890e-bd0b27960d3b" />

> Working!

### From previous section the userIdentity_ARN we found: 
`arn:aws:sts::509843726190:assumed-role/S3Reader/drinks`
> `arn:aws:sts::<AWS Account ID>:assumed-role/<assumed role Name>/<assumed role session name>`

3. Now, we want to hunt for `userIdentity_userName` who assumed role named, `S3Reader` having assumed role session name `drinks`:
```sql
SELECT 
	EventTIme,
	EventName,
	userIdentity_userName,
	requestParameters,
	userIdentity_ARN,
	EventSource
FROM cloudtrail WHERE EventName like 'AssumeRole' ORDER BY EventTime DESC	
```

<img width="2221" height="862" alt="Clipboard_2025-09-08-02-58-11" src="https://github.com/user-attachments/assets/f4dd3e32-01a3-49a6-a174-872661930d14" />

Adding `requestParameters` pattern search => `{"roleArn":"arn:aws:iam::509843726190:role/S3Reader","roleSessionName":"drinks"}` (derived from above retrieved `userIdentity_ARN`)

```sql
SELECT 
	EventTIme,
	EventName,
	userIdentity_userName,
	requestParameters,
	userIdentity_ARN,
	EventSource
FROM cloudtrail WHERE EventName like 'AssumeRole' 
  AND requestParameters like '{"roleArn":"arn:aws:iam::509843726190:role/S3Reader","roleSessionName":"drinks"}' 
ORDER BY EventTime DESC

OR A bit loose version,

SELECT 
    EventTime,
    EventName,
    userIdentity_userName,
    requestParameters,
    userIdentity_ARN,
    EventSource
FROM cloudtrail 
WHERE EventName LIKE 'AssumeRole' 
    AND requestParameters LIKE '%S3Reader%' 
    AND requestParameters LIKE '%drinks%' 
ORDER BY EventTime DESC
```

<img width="1920" height="116" alt="Clipboard_2025-09-08-03-03-14" src="https://github.com/user-attachments/assets/7823ae55-0675-49a9-8212-69e48264ca93" />

Notice another thing:

| Query1 | Query2 | 
| ------ | ------ |
| `SELECT * FROM cloudtrail where requestParameters like '%soda-vault%' ORDER BY EventTime DESC` | `SELECT * FROM cloudtrail WHERE EventName like 'AssumeRole' ORDER BY EventTime DESC` |
| <img width="2124" height="534" alt="Clipboard_2025-09-08-02-53-41" src="https://github.com/user-attachments/assets/d4a5fc01-d6eb-463a-9ab6-6f7bf7c01360" /> Notice EventSource - `S3` |  <img width="2218" height="400" alt="Clipboard_2025-09-08-02-56-24" src="https://github.com/user-attachments/assets/7b7e4daf-03b4-42f7-96ed-d586ac5e2ca0" /> Notice EventSource - `sts` |

Ans: `Moe.Jito`
``
