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

Meaning:
It states that we have `sts:AssumeRoleWithWebIdentity` action permission on Principal `"Federated": "cognito-identity.amazonaws.com"` with a condition of 
`--identity-pool-id` being `us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b`.

<img width="1755" height="651" alt="image" src="https://github.com/user-attachments/assets/b9bcfd2b-e1b7-4649-a1a7-7f751d14bdc2" />

Let's proceed, similarly like Challenge 5:
```
$ aws cognito-identity get-id --identity-pool-id us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b
```

<img width="1900" height="248" alt="image" src="https://github.com/user-attachments/assets/b8e69338-332d-4bda-86a6-75cc0ff2e6a5" />

```
$ 
```

