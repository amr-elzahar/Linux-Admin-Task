# Linux Administration Assessment | LinuxPlus

### Description

This assessment is an integral part of the LinuxPlus Company DevOps internship program.

### Infrastructure

**1. Five RHEL Servers:** These servers are running the web application and are used by developers.

**2. DNS/DHCP/LDAP/Logging Server:** DNS (Domain Name System) for domain resolution, DHCP (Dynamic Host Configuration Protocol) for IP allocation, LDAP (Lightweight Directory Access Protocol) for user authentication and logging.

**3. NFS Server:** This server hosts a 5TB file system shared with other servers. It allows multiple servers to access the same files.

**4. NFS Client Configuration:** Each server mounts the NFS share on the /opt/ecommerce directory using the /etc/fstab file.

### Directories:

**1. /app/backend:** This directory holds the web application's backend files. It's accessible by developers, the `admins` group, and the `dev` group. Its permissions are set to (rwx rwx rx). The owner is `dev` and the group is `dev`.

**2. /app/backend-dev:** This directory with permissions set to (rwx rx rx) and owned by `dev` and `dev` group. It's a temporary directory for training purposes.

### Users & Groups:

**1.admins:** Administrators group.

**2. dev:** Developers group.

**3. db:** Database admins group.

**4. temp_user:** This user will be creaded for temporary user access during the training.

### Security:

**1. SELinux:** It is enabled on the RHEL servers.

---

### Tickets

**1. Developer can't edit files in shared directory (/app/backend):**

1. Check the permissions and ownership of the `/app/backend` by running:

```
ls -l
```

It should be having `(rwx rwx rw)` and owned by `(dev:dev)`. If necessary, check the permissions and ownership of the files under `/app/backend`.

2. Check the group memberships of the `dev` user using to ensure that the ` dev`` user is a member of the  `dev` group by running:

```
groups dev
```

3. If necessary, add the `dev` user to the `dev` group by running:

```
usermod -aG dev dev
```

4. Check SELinux context and comparing to the expected context by running:

```
ls -Z /app/backend
```

**2. Files deleted in (/app/backend)**

1. Check logs for `/var/log/secure` or `/var/log/messages` by running:

```
sudo cat /var/log/secure

sudo cat /var/log/messages
```

2. Check access control:

   - Review user and group persmissions to understand who has the access to to `/app/backend`.

   - Identify users of the `dev` group who have been ganted access to `/app/backend` to ensure that no unauthorized users have access to it by running:

   ```
   getent group dev
   ```

   - Implement least privilege principles and remove users who no longer need access to the directory.

3. Consider taking a snapshot to recover deleted files later usong LVM snapshots.

**3. Providing Temporary User Access**

Use temporary ACL:

1. Create new user called `temp_user`

2. Add temporary ACL entry for the user on `/app/backend-dev`:

```
setfacl -m u:temp_user:rw /app/backend-dev
```

3. Once the trainig is complete, remove the ACL entry and remove the user.

**4. Slow Web Application Response**

1. Check network latency between frontend, backend and database servers
2. Check NFS server performance using `nfsstat`
3. Check database performane and use indexes to optimize query execution time
4. Check server's resources (CPU, memory, disk) using `top`

**5. Developers Can't Access Web Application URL**

1. Check permissions and ownership of `/var/www`
2. Check web server configurations
3. Check web server logs for any error messages
4. Check firewalls rules and SELinux contexts

**6. Mysqldump Backup Last Day of Each Month**

1. Edit cron configurations:

```
cronttab -e
```

2. Add the command that will be used to backup database:

```
0 0 L * * mysqldump -u <username> -p<password> <database> > /backup.sql
```

- The first field (0) represents the minute and is set to zero.
- The second field (0) represents the hour and is also set to zero.
- The third field (L) represents the day of the month. L refers to the "last" day of the month.
- The fourth field (\*) represents the month and is set to allow any month.
- The fifth field (\*) represents the day of the week and is also set to allow any day.
- By combining these fields, it means 12:00 AM on the last day of evry month
- `<username>`: MySQL user
- `<password>`: MySQL user's password
- `<database>`: Name of the DB that will be backed up.
- The backup will be saved to `/backup.sql`

**7. Mysqldump Backup on Specific Days**

1. Edit cron configurations:

```
crontab -e
```

2. Add this command:

```
0 0 5,25 * 5 mysqldump -u <username> -p<password> <database> > /backup.sql
```

- Combination of these fields translates to 12:00 AM on the 5th and 25th day of every month that falls on a Friday.

**8. Server Fails to Boot After SELinux and NFS Changes**

1. Boot into rescue mode by appending `systemd.unit=rescue.target` to the kernel command during GRUB
2. Check boot logs `/var/log/boot.log` and other logs like `messages` and `secure`
3. Check if SELinux and NFS server changes are the root cause for this issue
4. If they are, revert the changes made during configuration

**9. Podman Containers Disappear**

1. Ensure you logged in with the same user as before.
2. Check if there are any stopped containers `podman ps -a` and check their logs to be aware of the root cuase for this problem
3. Check if the containers and imafes were created using `podman` and not a different container runtime.

**10. File System Reaches 90% Utilization**

1. Identify large files conduming the most space using `du` command
2. Check if the logs are not rotated properly.
3. Remove unnecessary files.
4. Resize the filesystem and add additional storage.

**11. Container with Persistent Storage Fails to Start**

1. Check container logs for error messages.
2. Confirm that the required ports for port mapping aren't conflicting with other services.
3. Check the persistent storage is properly mounted with the container.
4. The container may depend on others service which is not running.

**12. Backup and Restore of Giant Directory**

1. Create a compressed archive of the directory on the source server:

```
tar -czvf backup.tar.gz /path/to/source_directory
```

- `-c`: To create a new archive
- `-z`: To use gzip compression algorithm
- `-v`: verbose mode, shows the progress of the transfer.
- `-f`: To specify the name of the archive file will be created
- `backup.tar.gz`: The new archive file
- `/path/to/source_directory`: Path to the directory that I want to archive

2. Transfer the archive file to the target server using `rsync`:

```
rsync -avz backup.tar.gz user@target_server:/path/on/target/server/
```

- `-a`: To enable archive mode that preserves file permissions and timestamp

- `-v`: Verbose mode, shows the progress of the transfer.
- `-z`: Compresses data during transfer.
- `user@target_server`: The IP or the hostname of the target server
- `/path/on/target/server/`: The directory path on the target server where you want to transfer the archive.

3. Extract the arhive on the target server:

```
tar -xzvf backup.tar.gz -C /path/to/destination_directory
```

- `-x`: To Extract files from the archive
- `-z`: To use decompression algorithm
- `-v`: Verbose mode, shows the progress of the transfer.
- `-f`: To specify the name of the archove file from which files will be extracted
- `backup.tar.gz`: The name of the archive file that you want to extract from
- `-C`: To specify the destination directory
- `/path/to/destination_directory`: The destination directory where the contents of the archive will be extracted

### General Question

**Differences Between Emergency Target, Rescue Target, and rd.break**

1. `emergency.target`: Systemd target that is used for troubleshooting and recovery. It provides a minimal environment to fix extremly critical issues. No services are started in this target. It is used in suitations that do not require running services like filesystem corruption.
2. `rescue.target`: Systemd target designed to repair the system. Unlike emergency.target, it starts essential services. It is used when you need more functional environment with essential services to troublshoot and fix issues.
3. `rd.break`: It's a kernel parameter that can be added during boot to interrupt the boot process. It is used to make manual changes before the system starts normally.
