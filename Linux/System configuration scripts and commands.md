

### Step 1: Create, Extract, Compress, and Manage tar Backup Archives

1. Command to **extract** the `TarDocs.tar` archive to the current directory:

        tar xvvf TarDocs.tar

2. Command to **create** the `Javaless_Doc.tar` archive from the `TarDocs/` directory, while excluding the `TarDocs/Documents/Java` directory:

      tar cvf Javaless_Docs.tar --exclude=Java ~/Projects/TarDocs/Documents/

3. Command to ensure `Java/` is not in the new `Javaless_Docs.tar` archive:

             tar tvf Javaless_Docs.tar | grep Java


**Bonus** 
- Command to create an incremental archive called `logs_backup_tar.gz` with only changed files to `snapshot.file` for the `/var/log` directory:

  sudo tar --listed-incremental=snapshot.file -cvzf logs_backup_tar.gz /var/log

#### Critical Analysis Question

- Why wouldn't you use the options `-x` and `-c` at the same with `tar`?

  We use -x for extracting the files and -c to create file , if we use both -x and -c in a command, then -x will export the file created by -c.

---

### Step 2: Create, Manage, and Automate Cron Jobs
This cron job should create an archive of the following file: `/var/log/auth.log`.
   - The filename and location of the archive should be: `/auth_backup.tgz`.
   - The archiving process should be scheduled to run every Wednesday at 6 a.m.
   - Use the correct archiving zip option to compress the archive using `gzip`.


1. Cron job for backing up the `/var/log/auth.log` file:

                * 6 * * 3 gzip  -t auth_backup.tgz /var/log/auth.log >/dev/null 2>&1

---

### Step 3: Write Basic Bash Scripts
Using brace expansion, create the following four directories:
      - `~/backups/freemem`
      - `~/backups/diskuse`
      - `~/backups/openlist`
      - `~/backups/freedisk

1. Brace expansion command to create the four subdirectories:

              mkdir -p ~/backups/{freemem,diskuse,openlist,freedisk}

a script that will execute various Linux tools to parse information about the system. Each of these tools should output results to a text file inside its respective system information directory.

2. Paste your `system.sh` script edits below:

    ```bash
    #!/bin/bash
    

free -h > ~/backups/freemem/free_mem.txt

du -h > ~/backups/diskuse/disk_use.txt

lsof > ~/backups/openlist/open_list.txt

df -h > ~/backups/freedisk/free_disk.txt                            
    ```

3. Command to make the `system.sh` script executable:

          chmod +x system.sh

**Optional**
- Commands to test the script and confirm its execution:

               sudo ./system.sh

**Bonus**
- Command to copy `system` to system-wide cron directory:

                sudo cp system.sh /etc/cron.weekly

---

### Step 4. Manage Log File Sizes
 
1. Run `sudo nano /etc/logrotate.conf` to edit the `logrotate` configuration file. 

    Configure a log rotation scheme that backs up authentication messages to the `/var/log/auth.log`.

    - Add your config file edits below:

    ```bash
    [Your logrotate scheme edits here]

            /var/log/auth.log {
             weekly
             rotate 7
             notifempty
             delaycompress
             missingok
            } 
    ```
---

### Bonus: Check for Policy and File Violations

1. Command to verify `auditd` is active:

               systemctl status auditd

2. Command to set number of retained logs and maximum log file size:

               sudo nano /etc/audit/auditd.conf

    - Add the edits made to the configuration file below:

    ```bash
    [        max_log_file = 35
             num_logs = 7                  ]

    ```

3. Command using `auditd` to set rules for `/etc/shadow`, `/etc/passwd` and `/var/log/auth.log`:

                   sudo nano /etc/audit/rules.d/audit.rules


    - Add the edits made to the `rules` file below:

    ```bash
    [       -w /etc/shadow -p wra -k hashpass_audit
            -w /etc/passwd -p wra -k userpass_audit
            -w /var/log/auth.log -p wra -k authlog_audit
    ```

4. Command to restart `auditd`:
        sudo systemctl restart auditd

5. Command to list all `auditd` rules:
        sudo auditctl -l


6. Command to produce an audit report:

      sudo aureport -au

7. Create a user with `sudo useradd attacker` and produce an audit report that lists account modifications:
                 sudo aureport -m

8. Command to use `auditd` to watch `/var/log/cron
          sudo auditctl -w /var/log/cron

9. Command to verify `auditd` rules:

           sudo auditctl -l

---

### Bonus (Research Activity): Perform Various Log Filtering Techniques

1. Command to return `journalctl` messages with priorities from emergency to error:

         sudo journalctl -p 0..3

1. Command to check the disk usage of the system journal unit since the most recent boot:

        sudo journalctl -b | less


1. Command to remove all archived journal files except the most recent two:

          journalctl --vacuum-files=2


1. Command to filter all log messages with priority levels between zero and two, and save output to `/home/sysadmin/Priority_High.txt`:

     sudo journalctl -p 0..2 > /home/sysadmin/Priority_High.txt

1. Command to automate the last command in a daily cronjob. Add the edits made to the crontab file below:
               crontab -e

    ```bash
    [  * 23 * * * journalctl -p 0..2 > /home/sysadmin/Priority_High.txt ]
    ```

---
© 2020 Trilogy Education Services, a 2U, Inc. brand. All Rights Reserved.
