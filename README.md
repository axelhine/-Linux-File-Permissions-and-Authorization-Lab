# File Permissions in Linux

> **Disclaimer:** This project was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario and file structure come from the course. The commands, outputs, and explanations are my own documentation of the work I performed.

## Project description
My organization's research team needed its file permissions audited to make sure only authorized users had access. Using Linux commands, I checked the permissions on all files and directories in `/home/researcher2/projects`, including hidden files, and compared them to the access each one should allow. I then used `chmod` to remove unauthorized access and strengthen the security of the system.

## Check file and directory details
I navigated to the projects directory and used `ls -la` to display all files and directories with their permissions, including hidden files. The `-l` option gives the detailed listing and the `-a` option includes hidden files.

```bash
cd projects
ls -la
```

Output:
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
- All files are owned by `researcher2` and the group `research_team`.
- `.project_x.txt` is a hidden file. It only appears when the `-a` option is used.
- `project_k.txt` is writable by other users.
- `project_m.txt` is readable by the group, but it should be restricted to the user.
- `.project_x.txt` is writable by the user and group, but it is archived and should be read-only.
- `drafts` has execute permission for the group, but only `researcher2` should have access.

## Describe the permissions string
Each entry in the `ls -l` output starts with a 10-character string that shows the type and permissions of the file or directory. Example from the output above: `drwx--x---` (the `drafts` directory).

| Position | Characters | Meaning |
|---|---|---|
| 1st | `d` | File type. `d` is a directory, `-` is a regular file |
| 2nd-4th | `rwx` | User (owner) can read, write, and execute |
| 5th-7th | `--x` | Group can only execute (no read or write) |
| 8th-10th | `---` | Other users have no permissions |

`r` = read, `w` = write, `x` = execute, and `-` means that permission is not granted. In `chmod`, `u` = user, `g` = group, `o` = other, `+` adds a permission, and `-` removes it.

## Change file permissions
The organization does not allow other users to have write access to any file. `project_k.txt` (`-rw-rw-rw-`) was the only file that violated this rule. I removed write permission for other:

```bash
chmod o-w project_k.txt
ls -l
```

Output:
```
-rw-rw-r-- 1 researcher2 research_team 46 Oct  7 19:15 project_k.txt
```

`project_m.txt` is a restricted file, so only the user should be able to read or write it. The group had read access (`-rw-r-----`), so I removed read and write for the group:

```bash
chmod g-wr project_m.txt
ls -la
```

Output:
```
-rw------- 1 researcher2 research_team 46 Oct  7 19:15 project_m.txt
```

## Change file permissions on a hidden file
`.project_x.txt` has been archived, so no one should be able to write to it, but the user and group should still be able to read it.

My first attempt failed because I left off the leading period, which hidden files require:

```bash
chmod g-w,u-w project_x.txt
```
```
chmod: cannot access 'project_x.txt': No such file or directory
```

I corrected the filename and ran the commands again. The first removes write for the user and group. The second adds read for both, though the user already had read, so only the group's permissions changed:

```bash
chmod g-w,u-w .project_x.txt
chmod g+r,u+r .project_x.txt
ls -la
```

Output:
```
-r--r----- 1 researcher2 research_team 46 Oct  7 19:15 .project_x.txt
```

The user and group can now read the file, and no one can write to it.

## Change directory permissions
Only `researcher2` should be able to access the `drafts` directory and its contents. The group had execute permission (`drwx--x---`), which allows it to enter the directory. I removed it:

```bash
chmod g-x drafts
ls -la
```

Output:
```
drwx------ 2 researcher2 research_team 4096 Oct  7 19:15 drafts
```

Only the user has permissions on the directory now.

### Final state of the projects directory
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

## Summary
I audited the permissions in the projects directory with `ls -la` and found four problems: other users could write to `project_k.txt`, the group could read the restricted `project_m.txt`, the archived hidden file `.project_x.txt` was writable, and the group could access the `drafts` directory. I fixed each one with `chmod` so that every file and directory matches the access it should allow. This applies the principle of least privilege and protects the research team's data from unauthorized changes.

## Key Takeaways
- `ls -la` is essential for audits because plain `ls -l` hides dotfiles.
- A world-writable file lets any user on the system modify it, a common misconfiguration attackers exploit.
- On a directory, the execute bit controls whether someone can enter it. Removing it from the group locks the group out.
- `chmod` accepts multiple changes in one command separated by commas (`g-w,u-w`).
- Hidden files require the leading period in every command.
- Re-running `ls -la` after each `chmod` confirms the change worked.
