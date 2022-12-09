# Disaster Recovery & Point-in-time recovery (PITR)

### Disaster Recovery

All Hydra plans are monitored continuously by our infrastructure and will automatically recover in the case of hardware failure. All production plans include high availability allowing seamless instant recovery should there be a failure.

In addition, WAL (write-ahead logs) are backed up to an encrypted S3 bucket for 30 days. Should there be a need to restore your data from backup, we can restore your data to any point in the last 30 days. Should you need this service, please open a support ticket.

### Point-in-time Recovery, Hydra Cloud

Point-in-time recovery (PITR) is a method of restoring a database to a specific point in time. This can be useful in situations where data has been accidentally deleted or corrupted, or if you need to undo a series of changes that were made to the database.

With Hydra, PITR is achieved through the use of continuous archiving and WAL (write-ahead logging). Continuous archiving involves constantly copying transaction log files (WAL files) to a remote location as they are generated. This allows you to restore the database to any point in time by replaying the WAL files up to the desired point.

To restore your database to a point-in-time create a support ticket from the Hydra dashboard that includes the following:

* Database name
* Database plan size
* Timestamp (day, hour) you would like to restore to

### Self-managed PITR (Hydra open source users)

To set up PITR in , you first need to enable WAL archiving by modifying the `postgresql.conf` configuration file and setting the `wal_level` parameter to `archive`. You also need to specify the location where the WAL files should be stored using the `archive_mode` and `archive_command` parameters.

Once continuous archiving is enabled, you can use the `pg_basebackup` command to create a backup of the current database, including the WAL files. This backup can then be restored using the `pg_restore` command, along with the `--recovery-target` option to specify the point in time to which you want to restore the database.





PITR can be a valuable tool for protecting your database against data loss and ensuring that you can always recover from unexpected events. However, it's important to regularly test your PITR setup to ensure that it is working properly and that you can restore the database to the desired point in time.
