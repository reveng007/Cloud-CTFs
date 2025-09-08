# 4.WIZ Cloud Hunting CTF:

## 4. Ain't no mountain high enough to keep me away from my logs

-------

Great progress! As you continue your investigation, you discover that once the attacker compromised the EC2 machine, they were able to manipulate a Lambda function to gain access to multiple IAM users. ExfilCola has granted you root access to the EC2 machine where this activity originated from.

But the question remains - how did the attacker gain access to this machine?

Your task is to find the IP address of another ExfilCola workload that was used as the initial entry point into the organization.

------

1. Normally for Endpoint Analysis on linux, log file analysis is the way to go.

<img width="757" height="276" alt="Clipboard_2025-09-08-06-23-49" src="https://github.com/user-attachments/assets/345e43e7-0fcc-48ab-987d-06b1866f7676" />

> It seems deleted.

2. Went up to Claude and asked:
> What are the ways folders can be deleted in linux and what are the ways those can be retrieved ?

> link: https://claude.ai/public/artifacts/e3f3a5b4-5e19-4da0-83ec-75bffce01662

Started from: Best Practice for Recovery:
```
umount /dev/sdX

# 3. Mark filesystem as read-only
mount -o ro /dev/sdX /mnt/readonly
```

```
# Used findmnt to check:
$ findmnt
```

<img width="1722" height="263" alt="Clipboard_2025-09-08-05-44-30" src="https://github.com/user-attachments/assets/e3affd1e-5762-49ae-8fe6-1a9fce4a4452" />

```
$ umount /var/log
```

<img width="814" height="371" alt="Clipboard_2025-09-08-06-02-48" src="https://github.com/user-attachments/assets/5fdeea94-5f1b-4a2a-8bd3-50b060da7850" />

> nothing!

3. Tried to list down files to search for other vectors:

<img width="1244" height="618" alt="Clipboard_2025-09-08-06-06-47" src="https://github.com/user-attachments/assets/d9155282-281c-4c4d-ab33-0538c9635093" />

> We can see 2 python files.

4. 

> lambda function file: probably the one file which attacker used to exploit.
> Checking logger.py

<img width="918" height="298" alt="Clipboard_2025-09-08-06-24-52" src="https://github.com/user-attachments/assets/aa0637d5-38f5-42c7-9985-1fa2d02f7ece" />

Resolving the above host:

<img width="712" height="275" alt="Clipboard_2025-09-08-06-30-19" src="https://github.com/user-attachments/assets/48e4aa15-7ae0-4de7-ba7f-3d557bbec77b" />

> tried this ip, but it didn't work!

### NOTE:
I again checked back `/var/log` directory, surprisingly `umount` worked!:

<img width="949" height="65" alt="Clipboard_2025-09-08-06-14-18" src="https://github.com/user-attachments/assets/820c0d97-5ae0-4ce6-bac8-0a916577d2ca" />

Within `auth.log`, we can see many ssh connections from a specific ip.

<img width="2828" height="1186" alt="Clipboard_2025-09-08-06-16-33" src="https://github.com/user-attachments/assets/c9a31f3e-2437-44f1-a314-23e8cc91587f" />

Ans: `102.54.197.238`
