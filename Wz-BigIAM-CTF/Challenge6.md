# Challenge 6

## Task
One final push
Anonymous access no more. Let's see what can you do now.

Now try it with the authenticated role: `arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role`

-----

We are provided with:
1. IAM role: `arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role`
2. Again with IAM assumed role as previously we had.
3. A Resource based policy

<img width="1679" height="731" alt="image" src="https://github.com/user-attachments/assets/dc64536f-007a-4d74-a1c6-f28b6b901d33" />

-------

It states that we have `sts:AssumeRoleWithWebIdentity` action permission on Principal `"Federated": "cognito-identity.amazonaws.com"` with a condition of `"cognito-identity.amazonaws.com:aud": "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"`.

Quick search on does this retieved id can mean:
<img width="1755" height="651" alt="image" src="https://github.com/user-attachments/assets/b9bcfd2b-e1b7-4649-a1a7-7f751d14bdc2" />

-----

### Meaning:
It states that we have `sts:AssumeRoleWithWebIdentity` action permission on Principal `"Federated": "cognito-identity.amazonaws.com"` with a condition of 
`--identity-pool-id` being `us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b`.

------

Let's proceed, similarly like Challenge 5:
```bash
$ aws cognito-identity get-id --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b
```

<img width="1900" height="248" alt="image" src="https://github.com/user-attachments/assets/b8e69338-332d-4bda-86a6-75cc0ff2e6a5" />

Just like previously what we did in Challenge 5:

<img width="2808" height="316" alt="image" src="https://github.com/user-attachments/assets/08562708-6167-484c-baad-001d3401e7ef" />

> Nope!

<img width="2804" height="1231" alt="image" src="https://github.com/user-attachments/assets/7ac9a9fc-c88f-470d-b67a-6254c249b719" />

> It worked but it gave us this...

<img width="1407" height="722" alt="image" src="https://github.com/user-attachments/assets/e21f3793-74b6-4b01-b9c2-4896c7410794" />

#### Not this one, we needed : `arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role` :(

-----

### 1. So, I went back to question, trying from start with a fresh mind :

<img width="1581" height="657" alt="image" src="https://github.com/user-attachments/assets/9f897e09-6704-450d-977c-5f5dda86114b" />

We have to perform `AssumeRoleWithWebIdentity` action. Aws cli command for this is:

<img width="1794" height="844" alt="image" src="https://github.com/user-attachments/assets/9b786779-dbb4-4006-a688-e80d4435273f" />

> `assume-role-with-web-identity`

Let's see what are the [commands](https://docs.aws.amazon.com/cli/latest/reference/sts/assume-role-with-web-identity.html) that are needed by it to work!

<img width="1272" height="1218" alt="image" src="https://github.com/user-attachments/assets/c5ea383f-ab71-486c-b021-30b84f05b0d0" />

```bash
$ aws sts assume-role-with-web-identity --role-arn <value> --role-session-name <value> --web-identity-token <value>
```
> We can use the `role-arn` which is already given. \
> We can also give `--role-session-name` by our own. 
> #### Only thing, we dont' have is `--web-identity-token`

<img width="1309" height="440" alt="image" src="https://github.com/user-attachments/assets/ce452b5e-07c1-4752-b8c4-55d4a50f7c73" />

Made a quick search on Google, I got [this](https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/get-open-id-token.html) hit :

<img width="1749" height="525" alt="image" src="https://github.com/user-attachments/assets/371bc373-4b10-46c5-a9f7-d688742bcb22" />

<img width="1290" height="1000" alt="image" src="https://github.com/user-attachments/assets/ed6d1b6c-43c2-4cbd-8ba7-eca331aad9a8" />

```bash
$ aws cognito-identity get-open-id-token --identity-id us-east-1:157d6171-eeba-ced0-c0e1-59df4c546333
```

<img width="2807" height="567" alt="image" src="https://github.com/user-attachments/assets/5d105833-6ace-4e80-b6e7-87a19bd87207" />

| From [assume-role-with-web-identity](https://docs.aws.amazon.com/cli/latest/reference/sts/assume-role-with-web-identity.html) | From [get-open-id-token](https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/get-open-id-token.html) | 
| ------ | ----- |
| <img width="1874" height="721" alt="image" src="https://github.com/user-attachments/assets/217994d7-ebf6-4745-9f18-9527733ae0d4" /> | <img width="1910" height="1336" alt="image" src="https://github.com/user-attachments/assets/0a89d64e-1b3b-4b6c-8215-93ee3882d62b" /> |

### 2. Now, this retrieved OpenID token can be used with `assume-role-with-web-identity` which can be passed via `--web-identity-token`

```bash
$ aws sts assume-role-with-web-identity --role-arn arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role --role-session-name anythingwewant --web-identity-token <OpenID token>
```

<img width="2820" height="1036" alt="image" src="https://github.com/user-attachments/assets/fc7ec9b0-f193-46c2-aa72-23b70a11d809" />

### 3. So, let's use this short term creds (used in kali machine)

<img width="1414" height="912" alt="image" src="https://github.com/user-attachments/assets/8d516537-25ee-44fe-b488-f8fc9b322812" />

> In the middle, I added aws session key of previous challenge so to avoid confusion added a block 😅

### 4. Now, we have assumed role of `arn:aws:iam::092297851374:role/Cognito_s3accessAuth_Role` which is mentioned in the question.

The name says S3 access, let's try listing S3:
```
$ aws s3 ls
```

<img width="728" height="263" alt="image" src="https://github.com/user-attachments/assets/0463fb55-8788-423b-b626-747a135dfdf8" />

<img width="1416" height="275" alt="image" src="https://github.com/user-attachments/assets/fef4a5b2-8e6e-40bd-9cae-bccb65a766a1" />

```bash
$ aws s3 cp s3://wiz-privatefiles-x1000/flag2.txt - | cat
```


