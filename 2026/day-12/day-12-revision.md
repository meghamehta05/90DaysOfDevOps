# Day 12 Revision

# Mindset & Learning Plan Review

- Reviewed my Day 01 learning goals
- Goal remains to become confident in Linux and DevOps fundamentals
- Improved consistency in daily GitHub uploads
- Need more practice in Linux troubleshooting and permissions

---

# Processes & Services Revision

## Process Check

```bash
ps aux
Observation:
Displayed all running processes successfully.

Service Status Check
systemctl status nginx

Observation:
Nginx service was active and running properly.

Log Check
journalctl -u nginx -n 20

Observation:
Recent logs showed successful service activity without errors.

File Skills Practice
Append Text into File
echo "DevOps Practice" >> notes.txt
Change File Permission
chmod 755 script.sh
Change File Ownership
sudo chown tokyo notes.txt
Verify Permissions
ls -l

Observation:
Permissions and ownership changes reflected correctly.

Cheat Sheet Refresh
5 Important Commands for Troubleshooting
Command	Purpose
ps aux	View running processes
top	Monitor CPU and memory
systemctl status	Check service health
journalctl	View service logs
ls -l	Check file permissions
User & Group Practice
Create User
sudo useradd -m reviser
Verify User
id reviser

Observation:
User created successfully with unique UID and group.