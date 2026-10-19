# Lab 08 - Linux User Management, Permissions, and Access Control Lists (ACLs)

## Estimated Time

Approximately **60–90 minutes**.

## Learning Objectives

By the end of this lab, you will be able to:

- Create and verify Linux groups.
- Create local user accounts and assign passwords.
- Add users to supplementary groups.
- Verify account and group membership using `id`, `groups`, `getent`, and system account files.
- Review standard Linux file permissions.
- Interpret symbolic and octal permission notation.
- Explain the purpose of `umask`.
- Create and inspect Access Control Lists (ACLs).
- Grant a named user permissions on a file using `setfacl`.
- Explain how the ACL mask limits effective permissions.
- Configure default ACLs on a directory.
- Verify inherited ACL permissions on newly created files.

---

## Scenario

You are working as a junior Linux system administrator. Your organisation needs several local user accounts with different access requirements.

In this lab, you will:

1. create users and groups;
2. review Linux file permissions;
3. test access using different user accounts;
4. use ACLs to grant permissions to a specific user;
5. modify the ACL mask;
6. configure default ACLs on a directory.

> [!note]
> This lab is for **self-assessment and practical learning**. No submission is required unless your instructor advises otherwise.

---

## Before You Begin

You need a Linux VM, a user account with `sudo` privileges, and the `acl` package.

Check the tools:

```bash
which getfacl
which setfacl
```

If required:

```bash
sudo apt update
sudo apt install acl -y
```

---

# Part A - Create Users and Groups

## Task 1 - Confirm Your Current Account

Run:

```bash
whoami
id
```

Review your username, UID, primary group, and supplementary groups.

---

## Task 2 - Create the `staff` Group

```bash
sudo groupadd staff
```

Verify:

```bash
getent group staff
```

Record the GID.

---

## Task 3 - Create User `anna`

```bash
sudo adduser anna
```

When prompted for the full name, enter:

```text
Anna Frey
```

Use the temporary password specified by your instructor.

Verify:

```bash
id anna
```

> [!alert]
> Do not include passwords in screenshots or reports.

---

## Task 4 - Create User `mark`

```bash
sudo adduser mark
```

Verify:

```bash
id mark
```

---

## Task 5 - Add `anna` and `mark` to `staff`

```bash
sudo usermod -aG staff anna
sudo usermod -aG staff mark
```

Verify:

```bash
groups anna
groups mark
```

> [!note]
> `-aG` appends a supplementary group. The primary group is changed with `-g`.
> Using `-G` without `-a` may replace the user's existing supplementary-group list.

---

## Task 6 - Verify the Account Records

```bash
getent passwd anna
getent passwd mark
getent group staff
```

You can also use:

```bash
grep '^anna:' /etc/passwd
grep '^mark:' /etc/passwd
grep '^staff:' /etc/group
```

> [!alert]
> Do not display the entire `/etc/shadow` file. It contains protected password-hash information.

---

# Part B - Review Standard Linux Permissions

## Task 7 - Understand the Permission String

Run:

```bash
ls -ld ~
```

A permission string may look like:

```text
drwxr-xr-x
```

Interpretation:

```text
d rwx r-x r-x
| |   |   |
| |   |   +-- Others
| |   +------ Group
| +---------- Owner
+------------ File type
```

Common file-type characters:

| Character | Meaning |
|---|---|
| `-` | Regular file |
| `d` | Directory |
| `l` | Symbolic link |

---

## Task 8 - Review Symbolic `chmod`

Create a test file:

```bash
cd ~
touch permission-demo.txt
```

Try:

```bash
chmod u+rwx permission-demo.txt
chmod u+rw permission-demo.txt
chmod o+r permission-demo.txt
chmod g-rwx permission-demo.txt
```

After each command:

```bash
ls -l permission-demo.txt
```

---

## Task 9 - Review Octal Permissions

| Number | Permission |
|---:|---|
| `7` | read + write + execute |
| `6` | read + write |
| `5` | read + execute |
| `4` | read |
| `3` | write + execute |
| `2` | write |
| `1` | execute |
| `0` | none |

Example:

```bash
chmod 640 permission-demo.txt
```

This means:

```text
Owner:  read + write
Group:  read
Others: none
```

---

# Part C - Create the ACL Practice Accounts

## Task 10 - Create the Required Accounts

Create:

```bash
sudo adduser --shell /bin/bash ben
sudo adduser --shell /bin/bash david
sudo adduser --shell /bin/bash mary
sudo adduser --shell /bin/bash masood
```

The required configuration is:

| Username | Group requirement | Home directory | Shell |
|---|---|---|---|
| `ben` | `sudo` | `/home/ben` | `/bin/bash` |
| `david` | own primary group | `/home/david` | `/bin/bash` |
| `mary` | own primary group | `/home/mary` | `/bin/bash` |
| `masood` | own primary group | `/home/masood` | `/bin/bash` |

---

## Task 11 - Add `ben` to `sudo`

```bash
sudo usermod -aG sudo ben
```

Verify:

```bash
groups ben
```

---

## Task 12 - Verify the Accounts

```bash
id ben
id david
id mary
id masood
```

Also:

```bash
getent passwd ben david mary masood
```

---

# Part D - `umask` and File Creation

## Task 13 - Switch to `student`

```bash
su - student
```

Confirm:

```bash
whoami
```

Expected:

```text
student
```

---

## Task 14 - Set the `umask`

Check:

```bash
umask
```

Set:

```bash
umask 002
```

Verify:

```bash
umask
```

Expected display may be:

```text
0002
```

---

## Task 15 - Create `studentfile.txt`

```bash
cd ~
touch studentfile.txt
echo "This file was created by user student" >> studentfile.txt
ls -l studentfile.txt
```

---

## Task 16 - Inspect the Initial ACL

```bash
getfacl studentfile.txt
```

Record the owner, group, and permissions.

---

# Part E - Test Access Before Adding an ACL

## Task 17 - Test Access as `ben`

```bash
su - ben
```

Try:

```bash
echo "This is from user ben" >> /home/student/studentfile.txt
```

Record whether it succeeds.

Return:

```bash
exit
```

---

## Task 18 - Test Access as `david`

```bash
su - david
```

Try:

```bash
echo "This is from user david" >> /home/student/studentfile.txt
```

Record whether it succeeds.

Return:

```bash
exit
```

---

# Part F - Add a Named User ACL

## Task 19 - Grant `david` Read and Write Access

As `student`:

```bash
setfacl -m u:david:rw studentfile.txt
```

Breakdown:

| Part | Meaning |
|---|---|
| `setfacl` | Modify ACLs |
| `-m` | Modify an ACL |
| `u` | User entry |
| `david` | Named user |
| `rw` | Read and write |
| `studentfile.txt` | Target file |

---

## Task 20 - Confirm the ACL Marker

```bash
ls -l studentfile.txt
```

A `+` after the permission bits indicates an extended ACL.

---

## Task 21 - Inspect the ACL

```bash
getfacl studentfile.txt
```

Look for:

```text
user:david:rw-
```

and a mask such as:

```text
mask::rw-
```

---

## Task 22 - Test `david` Again

```bash
su - david
```

Try:

```bash
echo "This is from user david after ACL" >> /home/student/studentfile.txt
```

Display the file:

```bash
cat /home/student/studentfile.txt
```

Return:

```bash
exit
```

---

# Part G - Understand the ACL Mask

## Task 23 - Review the Mask

Run:

```bash
getfacl studentfile.txt
```

The ACL mask defines the maximum effective permissions for named users, named groups, and the file's group entry.

---

## Task 24 - Restrict the Mask to Read-Only

```bash
setfacl -m m:r studentfile.txt
```

Then:

```bash
getfacl studentfile.txt
```

You may see:

```text
user:david:rw-              #effective:r--
mask::r--
```

---

## Task 25 - Test the Mask as `david`

```bash
su - david
```

Try:

```bash
echo "David write test with restrictive mask" >> /home/student/studentfile.txt
```

Expected:

```text
Permission denied
```

Return:

```bash
exit
```

---

## Task 26 - Expand `david`'s ACL

As `student`:

```bash
setfacl -m u:david:rwx studentfile.txt
getfacl studentfile.txt
```

Observe whether the mask is recalculated.

---

# Part H - Default ACLs

## Task 27 - Create a Directory

```bash
cd ~
mkdir -p stdir
```

---

## Task 28 - Assign a Default ACL

```bash
setfacl -d -m u:david:rw stdir
```

Inspect:

```bash
getfacl stdir
```

Look for:

```text
default:user:david:rw-
```

---

## Task 29 - Create a File Inside the Directory

```bash
touch stdir/newstfile.txt
```

Inspect:

```bash
getfacl stdir/newstfile.txt
```

Confirm that the file inherited an ACL entry for `david`.

---

## Task 30 - Test the Inherited ACL

```bash
su - david
```

Try:

```bash
echo "Created through inherited ACL permissions" >> /home/student/stdir/newstfile.txt
```

Record whether it succeeds.

Return:

```bash
exit
```

---

# Part I - Compare Standard Permissions and ACLs

## Task 31 - Review All Permission Information

Run:

```bash
ls -l studentfile.txt
getfacl studentfile.txt
```

Answer:

1. Who owns the file?
2. What is the owning group?
3. What are the standard user/group/other permissions?
4. Is there a named ACL entry for `david`?
5. What is the ACL mask?
6. What are `david`'s effective permissions?

---

## Task 32 - Compare Standard Permissions and ACLs

| Feature | Standard permissions | ACL |
|---|---|---|
| Owner permissions | Supported | Supported |
| Group permissions | Supported | Supported |
| Other permissions | Supported | Supported |
| Named user permissions | Limited | Supported |
| Named group permissions | Limited | Supported |
| Effective-permission mask | No | Yes |
| Default inheritance on directory | Limited | Supported |

---

# Part J - Clean Up

## Task 33 - Remove Practice Files

```bash
rm -f ~/studentfile.txt
rm -rf ~/stdir
rm -f ~/permission-demo.txt
```

---

## Task 34 - Remove Temporary Users

Only if instructed:

```bash
sudo deluser --remove-home anna
sudo deluser --remove-home mark
sudo deluser --remove-home ben
sudo deluser --remove-home david
sudo deluser --remove-home mary
sudo deluser --remove-home masood
```

---

## Task 35 - Remove the `staff` Group

If no longer needed:

```bash
sudo groupdel staff
```

Verify:

```bash
getent group staff
```

---

# Final Validation

Before completing the lab, confirm that you have:

- [ ] Created the `staff` group.
- [ ] Created `anna` and `mark`.
- [ ] Added both users to `staff`.
- [ ] Verified accounts with `id`, `groups`, and `getent`.
- [ ] Reviewed symbolic permissions.
- [ ] Reviewed octal permissions.
- [ ] Created `ben`, `david`, `mary`, and `masood`.
- [ ] Added `ben` to `sudo`.
- [ ] Set `umask 002`.
- [ ] Created `studentfile.txt`.
- [ ] Inspected its initial ACL.
- [ ] Tested access before adding an ACL.
- [ ] Added an ACL entry for `david`.
- [ ] Observed the `+` ACL indicator.
- [ ] Restricted effective permissions using the ACL mask.
- [ ] Created a default ACL on `stdir`.
- [ ] Verified ACL inheritance.

---

# Troubleshooting

## Problem - `setfacl` or `getfacl` Is Missing

```bash
sudo apt update
sudo apt install acl -y
```

## Problem - User Cannot Access `/home/student/studentfile.txt`

Check:

```bash
ls -l /home/student/studentfile.txt
getfacl /home/student/studentfile.txt
namei -l /home/student/studentfile.txt
```

A user must also have execute (`x`) permission on parent directories to reach the file.

## Problem - A User Already Exists

```bash
id <username>
```

Do not create a duplicate.

## Problem - `staff` Already Exists

```bash
getent group staff
```

Continue using the existing group if appropriate.

## Problem - `david` Still Cannot Write

Check:

```bash
getfacl studentfile.txt
```

Pay attention to both:

```text
user:david:
```

and:

```text
mask::
```

Then check path permissions with:

```bash
namei -l /home/student/studentfile.txt
```

---

# Knowledge Check

1. What is the difference between a Linux user and group?
2. What does `groupadd` do?
3. What is the difference between `usermod -g` and `usermod -aG`?
4. What does `id <username>` display?
5. What information is stored in `/etc/passwd`?
6. Why is `/etc/shadow` protected?
7. What do owner, group, and other permissions represent?
8. What does permission `640` mean?
9. What is the purpose of `umask`?
10. What is an Access Control List?
11. What does `setfacl -m` do?
12. What does the `+` in `ls -l` indicate?
13. What is the ACL mask?
14. Why can an ACL entry show `rw` while the effective permission is only `r`?
15. What is a default ACL?
16. How does a default ACL differ from an ordinary access ACL?
17. Why is `getfacl` useful for troubleshooting?
18. Why should `usermod -G` generally not be used alone when adding a supplementary group?

---

# Challenge

1. Create a user named `alex`.
2. Create a group named `project`.
3. Add `alex` to `project` as a supplementary group.
4. Create `project-data` as `student`.
5. Create `notes.txt` inside it.
6. Give `alex` read/write ACL access to `notes.txt`.
7. Verify the ACL.
8. Change the ACL mask so `alex` has only effective read access.
9. Prove that `alex` can read but cannot modify the file.
10. Configure a default ACL on `project-data` so new files grant `alex` read/write permission.
11. Create another file and verify ACL inheritance.

Then write:

```text
Standard permission observation:
ACL observation:
Mask observation:
Default ACL observation:
```

---

## Summary

In this lab, you:

- created Linux users and groups;
- assigned supplementary groups;
- verified account configuration;
- reviewed symbolic and octal permissions;
- used `umask`;
- created and inspected ACLs;
- granted permissions to a named user;
- examined and modified the ACL mask;
- configured default ACLs;
- verified inherited permissions;
- troubleshot access with `getfacl`, `ls`, and `namei`.
