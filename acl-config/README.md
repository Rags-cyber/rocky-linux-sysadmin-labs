# Access Control Lists (ACL) on Rocky Linux

> Fine-grained file and directory permission management using ACL on Rocky Linux, including ACL masks and how they override user permissions.

**Environment:** Rocky Linux inside Oracle VirtualBox

📝 **Full write-up with screenshots:** [Medium Article](https://medium.com/@gurungatwork98/mastering-linux-access-control-lists-acl-a-hands-on-guide-13f5bc70c21e)

---

## Table of Contents

- [Part 1: ACL for Files](#part-1-acl-for-files)
- [Part 2: ACL for Directories](#part-2-acl-for-directories)
- [Part 3: ACL Mask](#part-3-acl-mask)
- [What I Learned](#what-i-learned)

---

## Part 1: ACL for Files

### Step 1: Create a user and verify

```bash
sudo useradd user1
sudo passwd user1
```

Verify the user was created:

```bash
id user1
```

### Step 2: Check user credentials

```bash
cat /etc/passwd | grep user1
```

### Step 3: Create a test file as the new user

Switch to the user and go to `/tmp`:

```bash
su - user1
cd /tmp
echo "This is a test file" > testfile.txt
cat testfile.txt
```

### Step 4: Remove permissions for others

```bash
chmod o-rwx testfile.txt
ls -l testfile.txt
```

Now only the file owner has access. Others have no permissions at all.

### Step 5: Test permissions as another user

Switch to a different user and try to read the file — it should be denied:

```bash
su - anotheruser
cat /tmp/testfile.txt
# Expected: Permission denied
```

### Step 6: Grant read-only ACL permission to user1

```bash
setfacl -m u:user1:r testfile.txt
ls -l testfile.txt
```

> The `+` symbol at the end of the permissions (e.g., `-rw-------+`) confirms that ACL is applied on this file.

Now only `user1` has read-only access via ACL.

### Step 7: Verify user1 can read the file

```bash
su - user1
cat /tmp/testfile.txt
# Expected: file contents displayed
```

### Step 8: Verify others still cannot access the file

Switch to a user that is not `user1` and confirm access is denied:

```bash
su - otherrandomuser
cat /tmp/testfile.txt
# Expected: Permission denied
```

---

## Part 2: ACL for Directories

### Step 1: Create a directory in /tmp

```bash
sudo mkdir /tmp/testdir
```

### Step 2: Set base permissions on the directory

Grant read, write, execute to owner; read and execute to group; no permissions to others:

```bash
chmod 750 /tmp/testdir
ls -ld /tmp/testdir
```

### Step 3: Verify another user cannot access it

```bash
su - anotheruser
ls /tmp/testdir
# Expected: Permission denied
```

### Step 4: Grant user1 read and execute via ACL

```bash
setfacl -m u:user1:rx /tmp/testdir
getfacl /tmp/testdir
```

### Step 5: Access the directory as user1

```bash
su - user1
ls /tmp/testdir
# Expected: directory contents listed successfully
```

### Step 6: Verify others still cannot access it

```bash
su - otherrandomuser
ls /tmp/testdir
# Expected: Permission denied
```

---

## Part 3: ACL Mask

The ACL mask limits the **maximum effective permissions** for any named user or group, regardless of what ACL entries say.

### Step 1: Grant user1 full permissions

```bash
setfacl -m u:user1:rwx /tmp/testfile.txt
getfacl /tmp/testfile.txt
```

### Step 2: Set the mask to read-only

```bash
setfacl -m m::r /tmp/testfile.txt
getfacl /tmp/testfile.txt
```

> Even though `user1` was granted `rwx`, the mask of `r` limits the **effective** permission to read-only. user1 cannot write or execute.

### Step 3: Verify the mask is working

```bash
su - user1
echo "trying to write" >> /tmp/testfile.txt
# Expected: Permission denied
cat /tmp/testfile.txt
# Expected: file contents displayed (read still works)
```

---

## Key Commands Reference

| Command | Purpose |
|---|---|
| `setfacl -m u:user:perms file` | Set ACL for a specific user |
| `setfacl -m m::perms file` | Set the ACL mask |
| `getfacl file` | View all ACL entries on a file |
| `setfacl -x u:user file` | Remove a specific ACL entry |
| `setfacl -b file` | Remove all ACL entries |

---

## What I Learned

- ACL provides permission control beyond the standard owner/group/others model
- The `+` symbol in `ls -l` output indicates ACL is active on a file or directory
- The ACL mask acts as an upper limit — it overrides individual ACL permissions if the mask is more restrictive
- ACL is especially useful in multi-user environments where different users need different access levels to the same file
