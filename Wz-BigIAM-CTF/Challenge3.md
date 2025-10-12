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
```
$ aws subscribe --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications --protocol <something> --notification-endpoint  <from given policy>
```
> we choose to add `--notification-endpoint` as flag as in the policy we have been given with `"sns:Endpoint": "*@tbic.wiz.io"`

Now, the question comes:
1. which protocol to use ?
2. what would be the `--notification-endpoint` ?

