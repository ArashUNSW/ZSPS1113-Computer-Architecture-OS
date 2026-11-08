# Week 9 Lab - CanClinic Access Control with Linux ACLs

## Estimated Time

Approximately **90–120 minutes**.

## Learning Objectives

By the end of this lab, you will be able to:

- Create Linux users and groups representing different organisational roles.
- Create a protected directory structure.
- Apply standard Linux permissions.
- Apply ACLs to individual users and groups.
- Inspect ACLs using `getfacl`.
- Explain the ACL mask and effective permissions.
- Configure default ACLs.
- Test access as different users.
- Build an access-control matrix.
- Apply the principle of least privilege.
- Identify security limitations in a file-based access-control system.

---

## Scenario

You are the system administrator for **CanClinic**.

CanClinic stores patient information in files rather than in a database. Your task is to use Linux users, groups, standard permissions, and Access Control Lists (ACLs) to control which staff members can access each patient's information.

The main security objective is:

```text
Principle of Least Privilege
```

Patients are not Linux users. Their information is stored under:

```text
/opt/CanClinic/patients/
```

The lab uses these role groups:

```text
doctors
nurses
```

## Staff Accounts

| Role | Name | Username |
|---|---|---|
| Doctor | Mary Teresa | `drmary` |
| Doctor | Beny Shannel | `drben` |
| Doctor | Eli Mo | `dreli` |
| Nurse | Lucia Blakeley | `lucia` |
| Nurse | Phil McGraw | `phil` |

Patients used in this lab:

```text
lancebourne1970
masoodmansoori2001
```

---

# Part A - Create the Roles

## Task 1 - Create the Doctor and Nurse Groups

Run:

```bash
sudo groupadd doctors
sudo groupadd nurses
```

Verify:

```bash
getent group doctors
getent group nurses
```

### Question

Why is using groups useful when multiple users have the same role?

---

# Part B - Create the Staff Accounts

## Task 2 - Create the Doctors

```bash
sudo useradd -M -d /opt -s /bin/bash drmary
sudo useradd -M -d /opt -s /bin/bash drben
sudo useradd -M -d /opt -s /bin/bash dreli
```

Add them to the `doctors` group:

```bash
sudo usermod -aG doctors drmary
sudo usermod -aG doctors drben
sudo usermod -aG doctors dreli
```

Verify:

```bash
id drmary
id drben
id dreli
```

---

## Task 3 - Create the Nurses

```bash
sudo useradd -M -d /opt -s /bin/bash lucia
sudo useradd -M -d /opt -s /bin/bash phil
```

Add them to the `nurses` group:

```bash
sudo usermod -aG nurses lucia
sudo usermod -aG nurses phil
```

Verify:

```bash
id lucia
id phil
```

Record for each user:

- UID
- primary group
- supplementary group(s)

---

# Part C - Build the CanClinic Directory Structure

## Task 4 - Create the Clinic Directory

```bash
sudo mkdir -p /opt/CanClinic/patients
```

Inspect:

```bash
ls -ld /opt/CanClinic
ls -ld /opt/CanClinic/patients
```

Apply restrictive permissions:

```bash
sudo chmod 750 /opt/CanClinic
sudo chmod 750 /opt/CanClinic/patients
```

Inspect again:

```bash
ls -ld /opt/CanClinic
ls -ld /opt/CanClinic/patients
```

### Question

What does `750` mean for the owner, group, and others?

---

# Part D - Create the Patient Files

## Task 5 - Create Lance Bourne's Record

```bash
sudo touch /opt/CanClinic/patients/lancebourne1970
```

Add the sample data:

```bash
echo "Lance,Bourne,1970,0543836456,lancb@outlook.com" | sudo tee /opt/CanClinic/patients/lancebourne1970
echo "drmary,10/05/2022,flu,ibuprofen,4mg per day" | sudo tee -a /opt/CanClinic/patients/lancebourne1970
echo "drmary,07/09/2024,ebola,vitaminC,1000mg per day" | sudo tee -a /opt/CanClinic/patients/lancebourne1970
```

Set Mary as owner:

```bash
sudo chown drmary /opt/CanClinic/patients/lancebourne1970
```

Set restrictive permissions:

```bash
sudo chmod 600 /opt/CanClinic/patients/lancebourne1970
```

Check:

```bash
ls -l /opt/CanClinic/patients/lancebourne1970
```

---

## Task 6 - Create Masood Mansoori's Record

```bash
sudo touch /opt/CanClinic/patients/masoodmansoori2001
```

Add:

```bash
echo "Masood,Mansoori,2001,081039475,masood.mans@mail.com" | sudo tee /opt/CanClinic/patients/masoodmansoori2001
echo "drmary,11/02/2023,chickenpox,scratchicilin,2mg per day" | sudo tee -a /opt/CanClinic/patients/masoodmansoori2001
echo "drben,04/03/2024,covid,Ivermectin,2mg per day" | sudo tee -a /opt/CanClinic/patients/masoodmansoori2001
echo "drmary,24/05/2024,headache,paracetamol,1000mg per day" | sudo tee -a /opt/CanClinic/patients/masoodmansoori2001
```

Set Mary as owner:

```bash
sudo chown drmary /opt/CanClinic/patients/masoodmansoori2001
```

Set:

```bash
sudo chmod 600 /opt/CanClinic/patients/masoodmansoori2001
```

Check:

```bash
ls -l /opt/CanClinic/patients/masoodmansoori2001
```

---

# Part E - Test Access Before ACLs

## Task 7 - Test Doctor Access

Try as Mary:

```bash
sudo -u drmary cat /opt/CanClinic/patients/masoodmansoori2001
```

Try as Ben:

```bash
sudo -u drben cat /opt/CanClinic/patients/masoodmansoori2001
```

Try as Eli:

```bash
sudo -u dreli cat /opt/CanClinic/patients/masoodmansoori2001
```

Record:

| User | Read succeeds? |
|---|---|
| `drmary` | |
| `drben` | |
| `dreli` | |

### Question

Why can Mary access the file while Ben and Eli cannot?

---

# Part F - Grant Access to an Assigned Doctor

Masood has:

```text
Primary doctor: drmary
Assigned doctor: drben
Not assigned: dreli
```

## Task 8 - Give Ben Access Using an ACL

```bash
sudo setfacl -m u:drben:rw /opt/CanClinic/patients/masoodmansoori2001
```

Inspect:

```bash
getfacl /opt/CanClinic/patients/masoodmansoori2001
```

Look for:

```text
user:drben:rw-
mask::rw-
```

Also run:

```bash
ls -l /opt/CanClinic/patients/masoodmansoori2001
```

### Question

What does the `+` after the normal permission bits indicate?

---

# Part G - Test the ACL

## Task 9 - Test Ben Again

Read:

```bash
sudo -u drben cat /opt/CanClinic/patients/masoodmansoori2001
```

Try writing:

```bash
sudo -u drben sh -c 'echo "drben,test visit,test issue,test medication,1mg" >> /opt/CanClinic/patients/masoodmansoori2001'
```

Display:

```bash
cat /opt/CanClinic/patients/masoodmansoori2001
```

Ben should now be able to read and write.

---

## Task 10 - Test Eli

```bash
sudo -u dreli cat /opt/CanClinic/patients/masoodmansoori2001
```

Expected:

```text
Permission denied
```

### Question

Why does Ben have access but Eli does not, even though both belong to `doctors`?

---

# Part H - Investigate the ACL Mask

## Task 11 - Inspect the Current Mask

```bash
getfacl /opt/CanClinic/patients/masoodmansoori2001
```

Find:

```text
user:drben:rw-
mask::rw-
```

---

## Task 12 - Restrict the Mask

```bash
sudo setfacl -m m:r /opt/CanClinic/patients/masoodmansoori2001
```

Inspect:

```bash
getfacl /opt/CanClinic/patients/masoodmansoori2001
```

You may see:

```text
user:drben:rw-      #effective:r--
mask::r--
```

### Questions

1. Has Ben's ACL entry changed?
2. Why can his effective permissions still change?

---

## Task 13 - Test Ben's Write Access

```bash
sudo -u drben sh -c 'echo "mask test" >> /opt/CanClinic/patients/masoodmansoori2001'
```

Expected:

```text
Permission denied
```

Restore:

```bash
sudo setfacl -m m:rw /opt/CanClinic/patients/masoodmansoori2001
```

---

# Part I - Nurses and Least Privilege

CanClinic requires that nurses:

- must not modify patient information;
- must not see the full medical record;
- should not see the health issue.

An ACL applies to the whole file. It cannot express a rule such as:

```text
Lucia may read columns 1, 2 and 4 but not column 3.
```

## Task 14 - Test Nurse Access to the Full Record

```bash
sudo -u lucia cat /opt/CanClinic/patients/masoodmansoori2001
```

Expected:

```text
Permission denied
```

### Discussion

Why would this command violate the case-study requirement?

```bash
sudo setfacl -m u:lucia:r /opt/CanClinic/patients/masoodmansoori2001
```

---

# Part J - Create a Nurse-Safe View

## Task 15 - Create the Nurse View

```bash
sudo mkdir -p /opt/CanClinic/nurse-view
```

Create Masood's nurse-safe record:

```bash
sudo touch /opt/CanClinic/nurse-view/masoodmansoori2001
```

Add only permitted information:

```bash
echo "Masood,Mansoori,2001" | sudo tee /opt/CanClinic/nurse-view/masoodmansoori2001
echo "11/02/2023,drmary,scratchicilin,2mg per day" | sudo tee -a /opt/CanClinic/nurse-view/masoodmansoori2001
echo "04/03/2024,drben,Ivermectin,2mg per day" | sudo tee -a /opt/CanClinic/nurse-view/masoodmansoori2001
```

> [!note]
> The health issue is intentionally excluded.

---

# Part K - Use a Group ACL for Nurses

## Task 16 - Give the Nurses Group Read Access

```bash
sudo chmod 700 /opt/CanClinic/nurse-view/masoodmansoori2001
```

Then:

```bash
sudo setfacl -m g:nurses:r /opt/CanClinic/nurse-view/masoodmansoori2001
```

Check:

```bash
getfacl /opt/CanClinic/nurse-view/masoodmansoori2001
```

Test Lucia:

```bash
sudo -u lucia cat /opt/CanClinic/nurse-view/masoodmansoori2001
```

Test Phil:

```bash
sudo -u phil cat /opt/CanClinic/nurse-view/masoodmansoori2001
```

Test writing:

```bash
sudo -u lucia sh -c 'echo "test" >> /opt/CanClinic/nurse-view/masoodmansoori2001'
```

Expected:

```text
Permission denied
```

### Question

How does this demonstrate least privilege?

---

# Part L - Default ACLs

## Task 17 - Set a Default Nurse ACL

```bash
sudo setfacl -d -m g:nurses:r /opt/CanClinic/nurse-view
```

Inspect:

```bash
getfacl /opt/CanClinic/nurse-view
```

Look for:

```text
default:group:nurses:r--
```

---

## Task 18 - Test Inheritance

Create:

```bash
sudo touch /opt/CanClinic/nurse-view/lancebourne1970
```

Inspect:

```bash
getfacl /opt/CanClinic/nurse-view/lancebourne1970
```

Confirm that the new file inherited an ACL for `nurses`.

### Question

Why might a default ACL be useful in this directory?

---

# Part M - A Dangerous Default ACL

Consider:

```bash
sudo setfacl -d -m g:doctors:rw /opt/CanClinic/patients
```

> [!alert]
> Do not run this command.

Predict what would happen when a new patient file is created.

Would these doctors all inherit access?

```text
drmary
drben
dreli
```

### Questions

1. Would this satisfy the requirement that doctors access only assigned patients?
2. Why could this violate least privilege?

---

# Part N - Build an Access Matrix

## Task 19 - Complete the Matrix

Use:

```text
r = read
w = write
x = execute/traverse
- = no access
```

| Subject | Lance full record | Masood full record | Nurse view | Patients directory |
|---|---|---|---|---|
| `drmary` | | | | |
| `drben` | | | | |
| `dreli` | | | | |
| `lucia` | | | | |
| `phil` | | | | |

### Questions

1. Who is the primary doctor for Masood?
2. Which additional doctor has access?
3. Why does Eli not have access?
4. Why do nurses use a separate view?
5. Which permissions demonstrate least privilege?

---

# Part O - Inspect the Final Permissions

## Task 20 - Inspect Masood's Full Record

```bash
ls -l /opt/CanClinic/patients/masoodmansoori2001
getfacl /opt/CanClinic/patients/masoodmansoori2001
```

## Task 21 - Inspect Lance's Full Record

```bash
ls -l /opt/CanClinic/patients/lancebourne1970
getfacl /opt/CanClinic/patients/lancebourne1970
```

## Task 22 - Inspect the Nurse View

```bash
ls -l /opt/CanClinic/nurse-view/masoodmansoori2001
getfacl /opt/CanClinic/nurse-view/masoodmansoori2001
```

For each object identify:

- owner;
- owning group;
- standard permissions;
- named users;
- named groups;
- ACL mask;
- effective permissions.

---

# Part P - Security Analysis

## Task 23 - Scenario 1

Consider:

```bash
chmod 777 /opt/CanClinic/patients
```

What confidentiality or integrity problems could this create?

---

## Task 24 - Scenario 2

Imagine every patient file gives:

```text
group:doctors:rw-
```

Why is this easier to administer but weaker from a security perspective?

---

## Task 25 - Scenario 3

A patient file contains:

```text
user:drben:rw-
mask::r--
```

Why can Ben not write?

---

## Task 26 - Scenario 4

A nurse is given read permission directly to the full patient file.

Why does this violate the policy even though the nurse cannot modify it?

---

## Task 27 - Scenario 5

A new doctor joins CanClinic.

Which is safer?

```text
Giving the doctor access to the entire patients directory
```

or:

```text
Adding a named ACL only when the doctor is assigned to a patient
```

Explain your answer.

---

# Final Review

## Task 28 - Explain the Core Concepts

### Standard Permissions

```text
owner
group
other
```

### Extended ACLs

```text
named user
named group
mask
```

### Default ACLs

```text
Permissions inherited by new objects created inside a directory.
```

### Least Privilege

```text
Give a user only the permissions required to perform their job.
```

---

# Final Validation

Before completing the lab, confirm that you have:

- [ ] Created `doctors` and `nurses`.
- [ ] Created all five staff accounts.
- [ ] Added users to the correct role groups.
- [ ] Created `/opt/CanClinic/patients`.
- [ ] Applied restrictive directory permissions.
- [ ] Created both patient records.
- [ ] Set Mary as owner of both records.
- [ ] Tested doctor access before ACLs.
- [ ] Added a named ACL for Ben.
- [ ] Verified the `+` ACL marker.
- [ ] Tested Ben's read/write access.
- [ ] Confirmed Eli is denied.
- [ ] Restricted the ACL mask.
- [ ] Created the nurse-safe view.
- [ ] Granted nurses read-only group ACL access.
- [ ] Configured and tested a default ACL.
- [ ] Completed the access-control matrix.
- [ ] Analysed the five security scenarios.

---

# Troubleshooting

## Problem - `setfacl` or `getfacl` Is Missing

```bash
sudo apt update
sudo apt install acl -y
```

## Problem - A User Cannot Access a File Even Though the ACL Looks Correct

Check:

```bash
getfacl <FILE>
```

Then:

```bash
namei -l <FILE>
```

A user needs execute (`x`) permission on each parent directory in the path.

## Problem - A Group Already Exists

```bash
getent group doctors
getent group nurses
```

Reuse the lab group if appropriate.

## Problem - A User Already Exists

```bash
id <username>
```

Do not create duplicates.

## Problem - Ben Cannot Write After Receiving `rw`

Check:

```bash
getfacl /opt/CanClinic/patients/masoodmansoori2001
```

Inspect the:

```text
mask::
```

entry.

---

# Knowledge Check

1. Why are groups useful for role-based access?
2. What does `750` mean?
3. Why is `600` appropriate as an initial permission for a patient file?
4. What does `setfacl -m u:drben:rw` do?
5. What does the `+` in `ls -l` indicate?
6. What is the ACL mask?
7. Why can effective permission differ from the ACL entry?
8. Why should Eli not receive access just because he belongs to `doctors`?
9. Why can an ACL not hide individual columns in a text file?
10. Why is a separate nurse-safe view useful?
11. Why is a group ACL suitable for nurses?
12. What is a default ACL?
13. Why is a default `doctors:rw` ACL on all patient records dangerous?
14. What does least privilege mean in this case study?
15. Why is `chmod 777` inappropriate for sensitive patient information?

---

# Challenge

Create a third patient record using the same CanClinic model.

Requirements:

1. choose one primary doctor;
2. choose one additional assigned doctor;
3. ensure the third doctor has no access;
4. create a nurse-safe view;
5. grant nurses read-only access;
6. configure inheritance where appropriate;
7. verify all access using `sudo -u`;
8. build a small access matrix;
9. write:

```text
Primary doctor:
Assigned doctor:
Denied doctor:
Nurse access:
Least-privilege control:
```

---

## Summary

In this lab, you:

- created role-based Linux groups;
- created staff accounts;
- built a protected CanClinic directory structure;
- created patient records;
- applied standard permissions;
- granted named-user ACL access;
- investigated the ACL mask;
- created a nurse-safe view;
- granted nurses read-only access;
- configured default ACLs;
- built an access-control matrix;
- analysed weak and strong access-control policies;
- applied the principle of least privilege.

The key takeaway is that standard Linux permissions are often not flexible enough for role-sensitive systems such as CanClinic. ACLs provide finer-grained control, but the security policy still needs to be designed carefully.
