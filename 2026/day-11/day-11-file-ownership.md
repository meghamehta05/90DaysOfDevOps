# Day 11 Challenge

## Files & Directories Created

- devops-file.txt
- team-notes.txt
- project-config.yaml
- app-logs/
- heist-project/
- bank-heist/

---

# Task 1: Understanding Ownership

## Check File Ownership

```bash
ls -l
Task 2: Basic chown Operations
Create File
touch devops-file.txt
Check Current Owner
ls -l devops-file.txt
Change Owner to tokyo
sudo chown tokyo devops-file.txt
Change Owner to berlin
sudo chown berlin devops-file.txt
Verify Changes
ls -l devops-file.txt
Task 3: Basic chgrp Operations
Create File
touch team-notes.txt
Check Current Group
ls -l team-notes.txt
Create Group
sudo groupadd heist-team
Change File Group
sudo chgrp heist-team team-notes.txt
Verify Group Change
ls -l team-notes.txt
Task 4: Combined Owner & Group Change
Create File
touch project-config.yaml
Change Owner and Group Together
sudo chown professor:heist-team project-config.yaml
Create Directory
mkdir app-logs
Change Directory Ownership
sudo chown berlin:heist-team app-logs
Verify Changes
ls -l
Task 5: Recursive Ownership
Create Directory Structure
mkdir -p heist-project/vault
mkdir -p heist-project/plans

touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf
Create Group
sudo groupadd planners
Recursive Ownership Change
sudo chown -R professor:planners heist-project/
Verify Recursive Changes
ls -lR heist-project/
Task 6: Practice Challenge
Create Groups
sudo groupadd vault-team
sudo groupadd tech-team
Create Directory
mkdir bank-heist
Create Files
touch bank-heist/access-codes.txt
touch bank-heist/blueprints.pdf
touch bank-heist/escape-plan.txt
Set Ownership
sudo chown tokyo:vault-team bank-heist/access-codes.txt

sudo chown berlin:tech-team bank-heist/blueprints.pdf

sudo chown nairobi:vault-team bank-heist/escape-plan.txt
Verify Ownership
ls -l bank-heist/
Commands Used
ls -l
touch
mkdir
chown
chgrp
groupadd
useradd
What I Learned
Difference between file owner and group
How to change file ownership using chown
How to change group ownership using chgrp
How recursive ownership works using -R
Importance of ownership management in DevOps systems