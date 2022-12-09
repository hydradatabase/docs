# Backups

### Backups, Hydra Cloud



### Backups, Self-managed

One option is to use the `pg_dump` utility, which allows you to create a binary dump of the database. This dump can be used to restore the database to its current state at any time. The `pg_dump` command has several options that allow you to customize the format and contents of the backup, such as specifying which tables or schemas to include in the backup.

Another option is to use continuous archiving and point-in-time recovery (PITR), which involves constantly copying transaction log files (WAL files) to a remote location. This allows you to restore the database to any point in time by replaying the WAL files up to the desired point. This can be useful if you need to undo a series of changes that were made to the database.

