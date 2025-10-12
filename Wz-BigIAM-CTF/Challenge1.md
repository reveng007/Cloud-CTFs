# Challenge 1:

We are provided with :
1. We are provided with aws cli configured with assumed role.

<img width="2036" height="420" alt="image" src="https://github.com/user-attachments/assets/93ac9d54-0c29-4167-8412-539e054c5be2" />

2. We can also view IAM policy

<img width="1204" height="921" alt="image" src="https://github.com/user-attachments/assets/c841109c-f849-4d98-a162-f4bad3f013d5" />

-----

This shows us that:
1. Iam user has `s3:GetObject` Action permission on `thebigiamchallenge-storage-9979f4b/*` Resource.
2. Iam user has `s3:ListBucket` Action permission on `thebigiamchallenge-storage-9979f4b` Resource with a condition of `files/*` prefix of S3.

Combining all the above information to check S3 bucket:
```bash
$ aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/
```
<img width="1217" height="190" alt="image" src="https://github.com/user-attachments/assets/9f515829-2396-4c0a-9015-903aa8204299" />

Then download it via `s3 cp`

<img width="2799" height="643" alt="image" src="https://github.com/user-attachments/assets/dccc459b-6323-4333-b8b8-599590af9df4" />

> Flag1
