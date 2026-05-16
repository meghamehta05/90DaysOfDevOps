# Day 10 Challenge

## Files Created

- devops.txt
- notes.txt
- script.sh
- project/

---

# Task 1: Create Files

## Create Empty File

```bash
touch devops.txt
Task 2: Read Files
Read notes.txt
cat notes.txt
Open script.sh in Read Only Mode
vim -R script.sh
Display First 5 Lines of /etc/passwd
head -n 5 /etc/passwd
Display Last 5 Lines of /etc/passwd
tail -n 5 /etc/passwd
Task 3: Understand Permissions
Check Permissions
ls -l devops.txt notes.txt script.sh
Current Permissions
rw-r--r-- → owner can read/write
group can read
others can read

script.sh was not executable initially.

Task 4: Modify Permissions
Make script.sh Executable
chmod +x script.sh
./script.sh

Verified script executed successfully.

Make devops.txt Read Only
chmod a-w devops.txt
Set notes.txt Permission to 640
chmod 640 notes.txt

Meaning:

Owner → read/write
Group → read
Others → no permission
Create project Directory with 755 Permission
mkdir project
chmod 755 project
Verify Updated Permissions
ls -l
ls -ld project

Verified all permission changes successfully.

Task 5: Test Permissions
Try Writing to Read-Only File
echo "test" >> devops.txt

Error:
Permission denied

Try Executing Non-Executable File
./notes.txt

Error:
Permission denied

Commands Used
touch
echo
cat
vim
head
tail
chmod
ls
mkdir