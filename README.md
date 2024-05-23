# DockerPostgresBackup

This repository contains a Bash script for backing up PostgreSQL databases running in a Docker container. The script performs a backup, saves it to a specified directory on the host machine, and manages old backups.

## Prerequisites

- Docker
- PostgreSQL running inside a Docker container
- Bash shell

## Setup

1. Clone this repository to your local machine.
2. Modify the script variables to match your setup:
    - `CONTAINER_NAME`: Name of your Docker container running PostgreSQL.
    - `BACKUP_DIR`: Directory on the host machine where backups will be stored.
    - `PGPASSWORD`: Replace `your_password` with your actual PostgreSQL user password.
    - `MAX_BACKUPS`: Maximum number of backup files to keep.


## Script Description

The script performs the following steps:

1. Sets up necessary variables including the backup directory, timestamp, and maximum backups.
2. Executes the `pg_dumpall` command inside the Docker container to create a backup of the PostgreSQL database.
3. Saves the backup file to the specified directory on the host machine.
4. Optionally removes backups older than 7 days.
5. Checks the number of backups and deletes the oldest one if there are more than the specified maximum number of backups.

## Usage

1. Make the script executable:
    ```sh
    chmod +x scripts/backup_script.sh
    ```
2. Run the script:
    ```sh
    ./scripts/backup_script.sh
    ```

## Script

```bash
#!/bin/bash

# Variables
CONTAINER_NAME=
BACKUP_DIR=
TIMESTAMP=$(date +%F_%H-%M-%S)
BACKUP_FILE=$BACKUP_DIR/backup_$TIMESTAMP.sql
MAX_BACKUPS=30

PGPASSWORD=your_password  # Replace with your actual PostgreSQL user password

# Perform the backup and save it directly to the host machine
docker exec -e PGPASSWORD=$PGPASSWORD -t $CONTAINER_NAME pg_dumpall -c -U your_username > $BACKUP_FILE

# Optional: Remove backups older than 7 days
find $BACKUP_DIR -type f -name "*.sql" -mtime +7 -exec rm -f {} \;

# Check the number of backups and delete the oldest if there are more than MAX_BACKUPS
BACKUP_COUNT=$(ls -1 $BACKUP_DIR/*.sql | wc -l)
if [ $BACKUP_COUNT -gt $MAX_BACKUPS]; then
  OLDEST_BACKUP=$(ls -1 $BACKUP_DIR/*.sql | head -n 1)
  rm -f $OLDEST_BACKUP
fi
