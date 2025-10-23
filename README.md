# Remote Linux Rsync Backup
=====================================

## Introduction
---------------

A Bash script that uses rsync to create a full, mirrored backup of one or more remote Linux servers. It's designed to run from a central backup server and pull backups over SSH.

## Author and License
--------------------

* Author: Chris Tian
* License: GPLv3

## Credits
----------

Based on code from [github.com/cloudnull/InstanceSync](https://github.com/cloudnull/InstanceSync)

## Core Function
----------------

This script iterates through a list of remote servers and uses rsync to create a mirror of the remote server's entire / filesystem on the local backup host.

* Stores each backup in a structured path: `<BKP_MAIN_PATH>/<server_name>/<BKP_REL_PATH>`

## Key Features
----------------

* **Multi-Server**: Backs up all servers listed in `REMOTE_SERVERS_LIST`.
* **Exclusion List**: Uses a comprehensive, space-separated exclude list (`EXCLUDE_LIST`) to skip temporary files, cache, and system-specific directories (`/proc`, `/sys`, etc.).
* **Log Handling**: By default (`VAR_LOG_TREE_ONLY=true`), it only backs up the directory structure of `/var/log` (not the log files themselves) to save space.
* **Retry Logic**: Automatically retries the rsync command up to 5 times in case of a connection failure.
* **Hooks**: Can run optional `PRE_EXEC_SCRIPT` and `POST_EXEC_SCRIPT` on the backup server before and after the jobs.

## Requirements
----------------

* **Rsync**: The rsync utility must be installed on the backup server AND all remote servers.
* **SSH Key Authentication**: The backup server must be able to SSH to each remote server as the root user without a password.
	+ You must have your SSH public key (`~/.ssh/id_rsa.pub`) from the backup server added to the `~/.ssh/authorized_keys` file for the root user on all remote servers.
	+ Test this with `ssh root@remote1.example.com`. It must log you in immediately without prompting for a password.

## Configuration
-----------------

All configuration is done by editing the variables at the top of the `remote_backup.sh` script.

| Variable | Description |
| --- | --- |
| `REMOTE_SERVERS_LIST` | A space-separated list of hostnames or IPs. (e.g., `'host1.com host2.com'`) |
| `BKP_MAIN_PATH` | The main parent directory on the backup server to store all backups. (e.g., `'/space/backups/hosts'`) |
| `BKP_REL_PATH` | The final subdirectory name where the rsync mirror will be created. (e.g., `'rsync_host_bkp'`) |
| `EXCLUDE_LIST` | A space-separated list of files and directories to exclude. |
| `VAR_LOG_TREE_ONLY` | Set to `true` to only back up the `/var/log` directory structure (not files), or `false` to back up all logs. |
| `VAR_LOG_PATH` | The path to the log directory on remote servers (default: `"/var/log"`). |
| `PRE_EXEC_SCRIPT` | (Optional) Full path to a script to run before any backups start. |
| `POST_EXEC_SCRIPT` | (Optional) Full path to a script to run after all backups finish. |
| `RSYNC_FLAGS` | The rsync flags to use. The default is `aHEAXSzx --delete -v`. |

## Usage
---------

1. Ensure all Requirements are met.
2. Configure the variables in the script.
3. Make the script executable (if needed): `chmod +x remote_backup.sh`
4. Run the script: `bash remote_backup.sh` or `./remote_backup.sh` (if it's executable)
