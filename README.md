# Linux File Permissions and Authorization Lab

> **Disclaimer:** This lab was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario, directory structure, and tasks come from the course. The command walkthrough and takeaways are my own documentation of the work I performed.

## Overview
Hands-on lab where I examined and corrected file and directory permissions in `/home/researcher2/projects` for the `researcher2` user. Authorization controls who can access specific resources. Without it, any user could read or modify other users' files or system files, which is a serious security risk. As a security analyst, setting appropriate access permissions is critical to protecting sensitive information.

## Scenario
In this scenario, I examined and managed the permissions on the files in the `/home/researcher2/projects` directory for the `researcher2` user. The `researcher2` user is part of the `research_team` group.

I had to check the permissions for all files in the directory, including hidden files, to make sure they aligned with the authorization that should be given. Where they didn't, I changed the permissions.

**Approach:**
1. Check the user and group permissions for all files in the `projects` directory.
2. Identify files with incorrect permissions and change them as needed.
3. Check the permissions of the `drafts` directory and remove unauthorized access.

**Authorization requirements:**
- No file should allow other users to write to it.
- `project_m.txt` is restricted: only the user should be able to read or write it.
- `.project_x.txt` is a hidden, archived file: the user and group can read it, but no one can write to it.
- Only `researcher2` should have access to the `drafts` directory and its contents.

## Skills Demonstrated
- Reading and interpreting Linux permission strings
- Finding hidden files with `ls -a`
- Identifying overly permissive files and directories
- Changing permissions with `chmod` using symbolic notation
- Applying the principle of least privilege
- Verifying every change after making it

## Commands Used
| Command | Purpose |
|---|---|
| `cd` | Change directory |
| `ls -l` | List files with permissions and ownership |
| `ls -la` | Same as above, including hidden files |
| `chmod` | Change file and directory permissions |

## Understanding Permission Strings
Each entry in `ls -l` starts with a 10-character string, for example `drwx--x---`:

| Position | Meaning |
|---|---|
| 1st | File type (`d` = directory, `-` = regular file) |
| 2nd-4th | User (owner) permissions: read, write, execute |
| 5th-7th | Group permissions: read, write, execute |
| 8th-10th | Other (everyone else) permissions: read, write, execute |

In `chmod`: `u` = user, `g` = group, `o` = other. `+` adds a permission, `-` removes it.

## Walkthrough

### Task 1: Check file and directory details
Navigated to the projects directory and listed everything, including hidden files.
```bash
cd projects
ls -la
```
Initial permissions:
```
drwxr-xr-x 3 researcher2 research_team 4096 Oct  7 19:15 .
drwxr-xr-x 3 researcher2 research_team 4096 Oct  7 20:58 ..
-rw--w---- 1 researcher2 research_team   46 Oct  7 19:15 .project_x.txt
drwx--x--- 2 researcher2 research_team 4096 Oct  7 19:15 drafts
-rw-rw-rw- 1 researcher2 research_team   46 Oct  7 19:15 project_k.txt
-rw-r----- 1 researcher2 research_team   46 Oct  7 19:15 project_m.txt
-rw-rw-r-- 1 researcher2 research_team   46 Oct  7 19:15 project_r.txt
-rw-rw-r-- 1 researcher2 research_team   46 Oct  7 19:15 project_t.txt
```
Findings:
- Group owner of all files: `research_team`
- Hidden file: `.project_x.txt` (only shows with `ls -a`)
- `project_k.txt` is world-writable (`-rw-rw-rw-`)
- `project_m.txt` is group-readable but should be restricted to the user
- `.project_x.txt` has write permission for user and group, and no read for the group
- `drafts` has execute permission for the group

### Task 2: Change file permissions
**Issue 1:** `project_k.txt` allowed other users to write to it. Removed write for other:
```bash
chmod o-w project_k.txt
ls -l
```
Result: `-rw-rw-r--`

**Issue 2:** `project_m.txt` is restricted, but the group had read access. Removed read and write for the group:
```bash
chmod g-wr project_m.txt
ls -la
```
Result: `-rw-------` (only the user can read and write)

### Task 3: Change permissions on a hidden file
`.project_x.txt` is archived, so no one should write to it, but the user and group should still be able to read it.

First attempt failed because I left off the leading period:
```bash
chmod g-w,u-w project_x.txt
```
```
chmod: cannot access 'project_x.txt': No such file or directory
```
Hidden files must be referenced with the period included. Corrected commands:
```bash
chmod g-w,u-w .project_x.txt
chmod g+r,u+r .project_x.txt
ls -la
```
Result: `-r--r-----` (user and group can read, nobody can write)

### Task 4: Change directory permissions
Only `researcher2` should access `drafts`. The group had execute permission (`drwx--x---`), which lets it enter the directory. Removed it:
```bash
chmod g-x drafts
ls -la
```
Result: `drwx------`

## Final State
```
drwxr-xr-x 3 researcher2 research_team 4096 Oct  7 19:15 .
drwxr-xr-x 3 researcher2 research_team 4096 Oct  7 20:58 ..
-r--r----- 1 researcher2 research_team   46 Oct  7 19:15 .project_x.txt
drwx------ 2 researcher2 research_team 4096 Oct  7 19:15 drafts
-rw-rw-r-- 1 researcher2 research_team   46 Oct  7 19:15 project_k.txt
-rw------- 1 researcher2 research_team   46 Oct  7 19:15 project_m.txt
-rw-rw-r-- 1 researcher2 research_team   46 Oct  7 19:15 project_r.txt
-rw-rw-r-- 1 researcher2 research_team   46 Oct  7 19:15 project_t.txt
```

## Full Command Sequence
```bash
cd projects
ls -la
ls -l
chmod o-w project_k.txt
ls -l
ls -la
chmod g-wr project_m.txt
ls -la
chmod g-w,u-w .project_x.txt
chmod g+r,u+r .project_x.txt
ls -la
chmod g-x drafts
ls -la
```

## Key Takeaways
- `ls -la` is essential for audits because plain `ls -l` hides dotfiles like `.project_x.txt`.
- A world-writable file (`o+w`) lets any user on the system modify it, which is a common misconfiguration attackers exploit.
- On a directory, the execute (`x`) bit controls whether someone can enter it and reach its contents. Removing it from the group locks the group out.
- `chmod` accepts multiple changes in one command separated by commas (`g-w,u-w`).
- Hidden files require the leading period in every command. Leaving it off returns "No such file or directory".
- Least privilege: each user and group gets only the access they need and nothing more.
- Re-running `ls -la` after each `chmod` confirms the change did what was intended.

## Tools
- Linux Bash shell
- `chmod`, `ls`, `cd`
