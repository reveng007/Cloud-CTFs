# 1.WIZ Cloud Hunting CTF:

## 1. Ain't no data when she's gone:

-----

FizzShadows claim that they were able to exfiltrate ExfilCola's secret recipes. You have to validate this claim before considering any further steps.

All ExfilCola's secret recipes are stored in a secured S3 bucket. Luckily, their security team was responsible enough to make sure that S3 data events were collected. You've been granted access to these logs, which are available in the s3_data_events table.

Go ahead and see if there are any traces of exfiltration: find the IAM role involved in the attack.

----

1. 
```sql
SELECT * FROM s3_data_events ORDER BY EventTime DESC
```
> To look at the logs and get familiar with it about parameters and such.


<img width="2858" height="465" alt="Clipboard_2025-09-08-01-22-21" src="https://github.com/user-attachments/assets/ff1964b5-02e9-4ba8-b7f5-da4837990dc0" />

<img width="1277" height="811" alt="Clipboard_2025-09-08-01-50-13" src="https://github.com/user-attachments/assets/e14b8449-4844-4c9b-b8c7-a26201fa5b4b" />

<img width="382" height="112" alt="Clipboard_2025-09-08-01-53-34" src="https://github.com/user-attachments/assets/aba82135-b153-4e70-803d-e9d23b1b624c" />

> From here:
> We can narrow down our search into:
> 1. Searching for: `path like '%recipe%'` (we can see in above logs and also asked in the question)
> 2. Searching for: `requestParameters like '%soda-vault%'`
> 3. And Keeping an eye on uncommon ip, if can be identified.

2. Now, we are looking for the ARN of any events that downloaded the recipe with `GetObject`.
```sql
SELECT useridentity_ARN FROM s3_data_events where path like '%recipe%' and eventname like 'GetObject' ORDER BY EventTime DESC
OR,
SELECT useridentity_ARN FROM s3_data_events where requestParameters like '%soda-vault%' and eventname like 'GetObject' ORDER BY EventTime DESC
OR, 
SELECT * FROM s3_data_events where path like '%recipe%' and requestParameters like '%soda-vault%' and eventname like 'GetObject' ORDER BY EventTime DESC
```

<img width="2844" height="202" alt="new" src="https://github.com/user-attachments/assets/4d66c90e-88f9-4035-ad63-f60660a6d7a3" />


> This ip address: `37.19.199.135` stands out of all!

3. Just Confirming our suspicion:
```sql
SELECT 
	EventTIme,
	EventName,
	IP,
	requestParameters,
	Path,
	userIdentity_ARN,
	EventSource
FROM s3_data_events where IP like '%37.19.199.135%' and path like '%recipe%' and requestParameters like '%soda-vault%' ORDER BY EventTime DESC
```

<img width="2580" height="866" alt="Clipboard_2025-09-08-02-04-18" src="https://github.com/user-attachments/assets/023ada9b-fb85-4b1b-91bc-97e9c629acc1" />

> Various GET Operations on multiple files along with a put request from same ip, which is sus!

4. A more thorough query:
```sql
SELECT * FROM s3_data_events where IP like '%37.19.199.135%' and path like '%recipe%' and requestParameters like '%soda-vault%' ORDER BY EventTime DESC
```

<img width="2334" height="102" alt="Clipboard_2025-09-08-02-04-49" src="https://github.com/user-attachments/assets/e7a8851d-08df-40e5-b869-b73950be1d06" />

----

> What path LIKE `'%recipe%'` means ?
> > `%recipe%` means: any string that contains the word "recipe" anywhere within it
> > The `% before "recipe"` matches any characters (or no characters) that come before "recipe"
> > The `% after "recipe"` matches any characters (or no characters) that come after "recipe"

---

Ans: `arn:aws:sts::509843726190:assumed-role/S3Reader/drinks`
> `arn:aws:sts::<AWS Account ID>:assumed-role/<assumed role Name>/<assumed role session name>`
