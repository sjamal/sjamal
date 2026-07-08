# Repository Security & Sanitization Guide

This guide details the standard security scanning and history-sanitization workflows used to audit personal code repositories before changing their visibility from **Private** to **Public**.

---

## 🛠️ Security Scanning Commands

Run these terminal commands within the root directory of a local repository to ensure no hardcoded secrets, personal data, or internal networking details are exposed.

### 1. Secret & Credential Auditing
Scan files for common variable patterns used to hold API keys, tokens, passwords, and active sessions.

*   **Universal Profile / General Scan:**
    ```bash
    grep -rnEi "api_key|token|secret|password|passwd|auth|bearer|private_key" .
    ```

*   **Python Applications (`.py`, `.json`, `.env`):**
    ```bash
    grep -rnEi "api_key|token|secret|password|passwd|auth|bearer|private_key|client_id|client_secret" --include="*.py" --include="*.json" --include="*.env" .
    ```

*   **Selenium Browser Automation:**
    ```bash
    grep -rnEi "username|user_name|login|email|password|passwd|secret|key|token|cookie|session" --include="*.py" .
    ```

*   **R Analysis Scripts & Notebooks (`.R`, `.Rmd`):**
    ```bash
    grep -rnEi "api_key|token|secret|password|passwd|auth|username|user_name" --include="*.R" --include="*.Rmd" .
    ```

*   **Ansible Infrastructure Playbooks (`.yml`, `.yaml`, `.cfg`):**
    ```bash
    grep -rnEi "ansible_password|ansible_ssh_private_key_file|secret|vault|password|passwd|token|api_key" --include="*.yml" --include="*.yaml" --include="*.cfg" .
    ```

### 2. Environment & Path Scans
Ensure that personal local system directories (which expose local machine usernames) are not explicitly hardcoded.
```bash
grep -rn "/Users/" . || grep -rn "/home/" .
```

### 3. Infrastructure & Network Leaks
Verify that no live production or internal home lab IPv4 addresses are leaked in setup scripts or inventories.
```bash
grep -rnE "([0-9]{1,3}\.){3}[0-9]{1,3}" .
```

---

### ⏳ Git History Sanitization
When sensitive data is uncovered deep inside historical commits, modifying the live file and making a normal commit will not erase the secret from Git's memory.

Use this workflow to compress the file history into a single clean snapshot while compiling all historical development logs and timestamps into the new commit description:
```bash
# 1. Extract the historical timeline into a temporary file
git log --date=short --format="- %ad: %s" > history_log.txt

# 2. Add an explicit header to the file
# Open history_log.txt and add "Initial release (Sanitized and consolidated)" 
# followed by a blank line at the very top of the file.

# 3. Roll back the Git index to the initial baseline commit without changing local files
git reset $(git rev-list --max-parents=0 HEAD)

# 4. Stage all current, sanitized scripts
git add .

# 5. Commit using the compiled text log as the source for the description
git commit -F history_log.txt

# 6. Delete the temporary log and force-push the pristine history up to GitHub
rm history_log.txt
git push origin main --force   # (Swap 'main' to 'master' depending on default branch)
```

---

Maintained by [sjamal](https://github.com/sjamal/sjamal)
