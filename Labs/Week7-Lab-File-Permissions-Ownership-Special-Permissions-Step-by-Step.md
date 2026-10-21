# Week 7 Lab - Linux File Permissions, Ownership, and Special Permissions

## Estimated Time

Approximately **60–90 minutes**.

## Learning Objectives

By the end of this lab, you will be able to:

- Create Linux users and groups for permission testing.
- Create files and directories.
- View and modify permissions using symbolic and octal `chmod` notation.
- Change user and group ownership using `chown` and `chgrp`.
- Explain and safely inspect SUID.
- Configure and test SGID on a directory.
- Configure and test the sticky bit on a shared directory.
- Verify permission changes using `ls`, `stat`, and practical access tests.

---

## Scenario

You are working as a junior Linux system administrator. Your task is to configure files and directories with different ownership and permission requirements and to test Linux special permissions in a controlled lab environment.

You will work primarily in:

```text
/tmp
```

The lab uses:

```text
/tmp/secure-dir
/tmp/public-dir
```

> [!alert]
> Do not modify SUID on critical system binaries such as `/usr/bin/passwd`. The original lab demonstrates this on the live system binary; this converted lab uses safe inspection instead.

---

# Part A - Prepare the Lab

## Task 1 - Confirm Your Current Account

Run:

```bash
whoami
id
```

Confirm that your account can use `sudo`.

## Task 2 - Create `testuser`

```bash
sudo useradd -m -s /bin/bash testuser
sudo passwd testuser
```

Verify:

```bash
id testuser
```

> [!note]
> Use a temporary lab password and do not include it in screenshots.

## Task 3 - Create `testgroup`

```bash
sudo groupadd testgroup
```

Verify:

```bash
getent group testgroup
```

---

# Part B - Create Practice Files and Directories

## Task 4 - Move to `/tmp`

```bash
cd /tmp
pwd
```

Expected:

```text
/tmp
```

## Task 5 - Create the Directories

```bash
mkdir secure-dir public-dir
ls -ld secure-dir public-dir
```

## Task 6 - Create the Files

```bash
touch secure-dir/secure-file.sh
echo 'date; who' > secure-dir/secure-file.sh
touch public-dir/public-file.sh
```

Verify:

```bash
ls -l secure-dir public-dir
```

---

# Part C - Modify Permissions with `chmod`

## Task 7 - Remove Access from `secure-dir`

```bash
chmod a-rx secure-dir
ls -ld secure-dir
ls secure-dir
```

Expected when listing the directory:

```text
Permission denied
```

> [!note]
> Directory execute permission controls traversal.

## Task 8 - Restore Access

```bash
chmod ugo+rx secure-dir
ls -ld secure-dir
ls secure-dir
```

## Task 9 - Remove Group Write Permission

```bash
chmod g-w /tmp/secure-dir/secure-file.sh
ls -l /tmp/secure-dir/secure-file.sh
```

## Task 10 - Add Group and Other Read/Execute Permissions

```bash
chmod go+rx /tmp/secure-dir/secure-file.sh
ls -l /tmp/secure-dir/secure-file.sh
```

---

# Part D - Make a Shared Directory

## Task 11 - Test `public-dir` as `testuser`

```bash
su - testuser
```

Try:

```bash
touch /tmp/public-dir/testuserfile
```

Record whether it succeeds. Return with:

```bash
exit
```

## Task 12 - Allow Everyone to Use `public-dir`

```bash
chmod a=rwx /tmp/public-dir
ls -ld /tmp/public-dir
```

Expected permissions:

```text
drwxrwxrwx
```

> [!note]
> Mode `777` is used here only to demonstrate a fully writable practice directory. It is usually inappropriate for sensitive production directories.

## Task 13 - Test Again as `testuser`

```bash
su - testuser
```

Then:

```bash
touch /tmp/public-dir/testuserfile
ls -l /tmp/public-dir/testuserfile
```

Return:

```bash
exit
```

---

# Part E - Remove and Grant File Permissions

## Task 14 - Remove Group and Other Permissions from `secure-file.sh`

```bash
ls -l /tmp/secure-dir/secure-file.sh
chmod g-rw,o-rx /tmp/secure-dir/secure-file.sh
ls -l /tmp/secure-dir/secure-file.sh
```

## Task 15 - Give Everyone Read/Write on `public-file.sh`

```bash
chmod a=rw /tmp/public-dir/public-file.sh
ls -l /tmp/public-dir/public-file.sh
```

Expected:

```text
-rw-rw-rw-
```

---

# Part F - Ownership

## Task 16 - Change Owner and Group of `public-dir`

```bash
sudo chown testuser:testgroup /tmp/public-dir
ls -ld /tmp/public-dir
```

Record the owner and group.

## Task 17 - Change Only the Owner of `public-file.sh`

```bash
sudo chown testuser /tmp/public-dir/public-file.sh
ls -l /tmp/public-dir/public-file.sh
```

## Task 18 - Change Group Ownership Recursively

```bash
sudo chgrp -R testgroup /tmp/secure-dir
ls -ld /tmp/secure-dir
ls -l /tmp/secure-dir/secure-file.sh
```

> [!note]
> `-R` applies the change recursively.

---

# Part G - Octal Permissions

## Task 19 - Apply `764`

```bash
chmod 764 /tmp/secure-dir/secure-file.sh
stat /tmp/secure-dir/secure-file.sh
```

Interpretation:

```text
7 = rwx = owner
6 = rw- = group
4 = r-- = others
```

## Task 20 - Apply `775`

```bash
chmod 775 /tmp/secure-dir/secure-file.sh
ls -l /tmp/secure-dir/secure-file.sh
```

Expected:

```text
-rwxrwxr-x
```

---

# Part H - Special Permissions

## Task 21 - Inspect SUID Safely

SUID means **Set User ID**. A SUID executable may run with the effective UID of its owner.

Inspect the system `passwd` command without changing it:

```bash
ls -l /usr/bin/passwd
```

You may see:

```text
-rwsr-xr-x
```

The `s` in the owner's execute position indicates SUID.

> [!alert]
> Do not run `chmod u-s /usr/bin/passwd`. Removing SUID from the real `passwd` binary can break password changes and is unsafe on a shared VM.

## Task 22 - Find Other SUID Files

```bash
find /usr/bin -perm -4000 -type f 2>/dev/null | head
```

Inspect one result:

```bash
ls -l <PATH>
```

Record the path, owner, and permission string.

---

# Part I - SGID on a Directory

## Task 23 - Set Group Ownership of `public-dir`

```bash
sudo chgrp testgroup /tmp/public-dir
ls -ld /tmp/public-dir
```

## Task 24 - Enable SGID

```bash
sudo chmod g+s /tmp/public-dir
ls -ld /tmp/public-dir
```

Look for `s` in the group execute position, for example:

```text
drwxrwsrwx
```

## Task 25 - Test Group Inheritance

Create a file:

```bash
touch /tmp/public-dir/new-file.txt
```

Check:

```bash
ls -l /tmp/public-dir/new-file.txt
```

The new file should inherit `testgroup` as its group.

> [!note]
> SGID on a directory causes new files and directories inside it to inherit the directory's group ownership.

---

# Part J - Sticky Bit

## Task 26 - Enable the Sticky Bit

```bash
sudo chmod +t /tmp/public-dir
ls -ld /tmp/public-dir
```

Expected ending:

```text
...rwt
```

## Task 27 - Create a File as `testuser`

```bash
su - testuser
```

Create:

```bash
touch /tmp/public-dir/user1-file.txt
ls -l /tmp/public-dir/user1-file.txt
```

Return:

```bash
exit
```

## Task 28 - Test Sticky-Bit Protection

As another non-root user, try:

```bash
rm /tmp/public-dir/user1-file.txt
```

Expected result:

```text
Operation not permitted
```

> [!note]
> In a sticky-bit directory, users may create files if they have write access, but they cannot normally delete files owned by another user.

---

# Part K - Compare Special Permissions

## Task 29 - Complete the Table

| Permission | Symbol | Typical use |
|---|---|---|
| SUID | `s` in owner execute position | Run executable with owner's effective UID |
| SGID | `s` in group execute position | Group inheritance on directories |
| Sticky bit | `t` in others execute position | Prevent users deleting each other's files in shared directories |

---

# Part L - Clean Up

## Task 30 - Remove Practice Directories

```bash
sudo rm -rf /tmp/secure-dir
sudo rm -rf /tmp/public-dir
```

## Task 31 - Remove Test Account and Group

Only if instructed:

```bash
sudo userdel -r testuser
sudo groupdel testgroup
```

Verify:

```bash
id testuser
getent group testgroup
```

---

# Final Validation

Before completing the lab, confirm that you have:

- [ ] Created `testuser` and `testgroup`.
- [ ] Created the secure and public practice directories.
- [ ] Modified permissions with symbolic `chmod`.
- [ ] Tested directory access as another user.
- [ ] Applied octal permissions.
- [ ] Changed user ownership with `chown`.
- [ ] Changed group ownership with `chgrp`.
- [ ] Reviewed SUID without changing a system binary.
- [ ] Enabled SGID and verified group inheritance.
- [ ] Enabled and tested the sticky bit.
- [ ] Cleaned up temporary resources if required.

---

# Troubleshooting

## `testuser` Already Exists

```bash
id testuser
```

Reuse it or remove it only if instructed.

## `testgroup` Already Exists

```bash
getent group testgroup
```

## Cannot Enter `secure-dir`

```bash
ls -ld /tmp/secure-dir
chmod u+x /tmp/secure-dir
```

## `testuser` Cannot Create a File in `public-dir`

```bash
ls -ld /tmp/public-dir
```

Confirm that the applicable permission class has both write and execute permission.

## SGID Does Not Produce the Expected Group

```bash
ls -ld /tmp/public-dir
```

Confirm the directory group is `testgroup` and the `s` bit is present.

## Sticky-Bit Test Does Not Fail

Confirm the deleting user is not root and does not own the file:

```bash
ls -ld /tmp/public-dir
ls -l /tmp/public-dir/user1-file.txt
```

---

# Knowledge Check

1. What do `r`, `w`, and `x` mean for a regular file?
2. What does `x` mean for a directory?
3. What does `chmod a-rx` do?
4. What does permission `764` mean?
5. What is the difference between `chown` and `chgrp`?
6. What does `chgrp -R` do?
7. What is SUID?
8. Why should SUID on a critical system binary not be modified casually?
9. What is SGID?
10. How does SGID affect a directory?
11. What is the sticky bit?
12. Why is the sticky bit commonly used on shared directories such as `/tmp`?
13. What is the difference between `chmod 777` and `chmod 1777`?
14. Why is a world-writable directory without a sticky bit risky?

---

# Challenge

Create a shared project directory with these requirements:

```text
Directory: /tmp/project-share
Group: testgroup
```

Requirements:

1. members of `testgroup` can create files;
2. new files inherit `testgroup`;
3. other users can enter the directory but cannot modify files unless permitted;
4. users cannot delete files owned by other users;
5. verify permissions using `ls -ld`;
6. create files as two different users and verify ownership/group behaviour.

Then write:

```text
Owner permissions:
Group permissions:
Other permissions:
SGID effect:
Sticky-bit effect:
```

---

## Summary

In this lab, you:

- created test users and groups;
- created files and directories;
- modified Linux permissions with symbolic and octal notation;
- changed user and group ownership;
- safely reviewed SUID;
- configured SGID and verified group inheritance;
- configured and tested the sticky bit;
- practised Linux permission troubleshooting.
