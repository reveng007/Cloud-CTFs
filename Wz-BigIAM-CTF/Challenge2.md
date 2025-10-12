# Challenge 2:

## Task:
We created our own analytics system specifically for this challenge. We think it's so good that we even used it on this page. What could go wrong?
Join our queue and get the secret flag.

-----

We are provided with :
1. `aws cli` configured with assumed role.

<img width="1529" height="328" alt="image" src="https://github.com/user-attachments/assets/0b9e1f0c-8497-4773-a307-3812e10c159b" />

2. We can also view IAM policy

<img width="1543" height="612" alt="image" src="https://github.com/user-attachments/assets/bf35ec03-d5a6-4619-b6c2-4a15696c89ca" />

------

We have `sqs:SendMessage` and `sqs:ReceiveMessage` action to perform.

With quick google search, i found this:
<img width="1384" height="1304" alt="image" src="https://github.com/user-attachments/assets/886843e7-110c-444f-909d-1b035f285284" />

> TLDR: We can send and receive messages.

Let's try sending one.

Structure of sending and Receiving message:

| [receive-message](https://docs.aws.amazon.com/cli/latest/reference/sqs/receive-message.html) | [send-message](https://docs.aws.amazon.com/cli/latest/reference/sqs/send-message.html) |
| ---- | ---- |
| <img width="1276" height="1240" alt="image" src="https://github.com/user-attachments/assets/0a26a40b-4b7b-4be6-9335-62b3640e327d" /> | <img width="1302" height="1217" alt="image" src="https://github.com/user-attachments/assets/6e7f3ab3-27b5-4c0b-9bc4-b8e07bc037c0" /> |

```bash
$ aws sqs send-message --queue-url https://sqs.us-east-1/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2 --message-body "Message sent via
AWS CLI"
```
> Crafted the queue url from: `"arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"`

<img width="2783" height="1080" alt="image" src="https://github.com/user-attachments/assets/dbbd2686-7baa-4b51-9249-9247de80303d" />

### No use!!

Let's try Receiving one:
```bash
$ aws sqs receive-message --queue-url https://sqs.us-east-1/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2
```

<img width="2823" height="862" alt="image" src="https://github.com/user-attachments/assets/bcba6ecc-e397-4460-9467-a3ac7c4bd413" />

> Now, we got something!

> If we simply go to the url, we can find our flag.
