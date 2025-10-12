# Challenge 1:

## Task
We all know that public buckets are risky. But can you find the flag?

---

We are provided with :
1. `aws cli` configured with assumed role.

<img width="2036" height="420" alt="image" src="https://github.com/user-attachments/assets/93ac9d54-0c29-4167-8412-539e054c5be2" />

2. We can also view Resource policy

<img width="1204" height="921" alt="image" src="https://github.com/user-attachments/assets/c841109c-f849-4d98-a162-f4bad3f013d5" />

-----

This shows us that:
1. 1st policy shows anyone (`Principal : "*"`) has `s3:GetObject` Action permission on `thebigiamchallenge-storage-9979f4b/*` Resource.
2. 2nd policy shows anyone (`Principal : "*"`) has  `s3:ListBucket` Action permission on `thebigiamchallenge-storage-9979f4b` Resource with a condition of `files/*` prefix of S3.

Combining all the above information to check S3 bucket:
```bash
$ aws s3 ls s3://thebigiamchallenge-storage-9979f4b/files/
```

Then download it via `s3 cp`

<img width="2798" height="682" alt="image" src="https://github.com/user-attachments/assets/c9533d98-ef44-4c32-87ed-664d61548574" />

> used `--no-sign-request` to perform request anonymously, without signing it with AWS credentials. \
> Flag1.
