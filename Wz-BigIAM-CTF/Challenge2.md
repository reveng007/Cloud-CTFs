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

We have `sqs:SendMessage` and `sqs:ReceiveMessage` Action 
