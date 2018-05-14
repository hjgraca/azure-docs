---
title: Azure SQL Database automatic, geo-redundant backups | Microsoft Docs
description: SQL Database automatically creates a local database backup every few minutes and uses Azure read-access geo-redundant storage for geo-redundancy.
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: business continuity
ms.topic: article
ms.workload: "Active"
ms.date: 04/04/2018
ms.author: sashan
ms.reviewer: carlrab

---
# Learn about automatic SQL Database backups

SQL Database automatically creates database backups and uses Azure read-access geo-redundant storage (RA-GRS) to provide geo-redundancy. These backups are created automatically and at no additional charge. You don't need to do anything to make them happen. Database backups are an essential part of any business continuity and disaster recovery strategy because they protect your data from accidental corruption or deletion. If you want to keep backups in your own storage container you can configure a long-term backup retention policy. For more information, see [Long-term retention](sql-database-long-term-retention.md).

## What is a SQL Database backup?

SQL Database uses SQL Server technology to create [full](https://msdn.microsoft.com/library/ms186289.aspx), [differential](https://msdn.microsoft.com/library/ms175526.aspx), and [transaction log](https://msdn.microsoft.com/library/ms191429.aspx) backups for the purposes of Point-in-time restore (PITR). The transaction log backups generally happen every 5 - 10 minutes, with the frequency based on the performance level and amount of database activity. Transaction log backups, with full and differential backups, allow you to restore a database to a specific point-in-time to the same server that hosts the database. When you restore a database, the service figures out which full, differential, and transaction log backups need to be restored.


You can use these backups to:

* Restore a database to a point-in-time within the retention period. This operation will create a new database in the same server as the original database.
* Restore a deleted database to the time it was deleted or any time within the retention period. The deleted database can only be restored in the same server where the original database was created.
* Restore a database to another geographical region. This allows you to recover from a geographic disaster when you cannot access your server and database. It creates a new database in any existing server anywhere in the world. 
* Restore a database from a specific long-term backup if the database has been configured with a long-term retention policy. This allows you to restore an old version of the database to satisfy a compliance request or to run an old version of the application. See [Long-term retention](sql-database-long-term-retention.md).
* To perform a restore, see [restore database from backups](sql-database-recovery-using-backups.md).

> [!NOTE]
> In Azure storage, the term *replication* refers to copying files from one location to another. SQL's *database replication* refers to keeping to multiple secondary databases synchronized with a primary database. 
> 

## How often do backups happen?
Full database backups happen weekly, differential database backups generally happen every few hours, and transaction log backups generally happen every 5 - 10 minutes. The first full backup is scheduled immediately after a database is created. It usually completes within 30 minutes, but it can take longer when the database is of a significant size. For example, the initial backup can take longer on a restored database or a database copy. After the first full backup, all further backups are scheduled automatically and managed silently in the background. The exact timing of all database backups is determined by the SQL Database service as it balances the overall system workload. 

The backup storage geo-replication occurs based on the Azure Storage replication schedule.

## How long do you keep my backups?
Each SQL Database backup has a retention period that is based on the service tier of the database, and differs between the [DTU-based purchasing model](sql-database-service-tiers-dtu.md) and the [vCore-based purchasing model (preview)](sql-database-service-tiers-vcore.md). 


### Database Retention for DTU-based purchasing model
The retention period for a database in the DTU-based purchasing model depends on the service tier. The retention period for a database for the:

* Basic service tier is 7 days.
* Standard service tier is 35 days.
* Premium service tier is 35 days.
* General-purpose tier is configurable with 35 days maximum (7 days by default)*
* Business Critical tier (preview) is configurable with 35 days maximum (7 days by default)*

\* During preview, the backups retention period is not configurable and is fixed to 7 days.

If you convert a database with longer backups retention to a database with shorter retention, all existing backups older than target tier retention period are no longer available.

If you upgrade a database with a shorter retention period to a database with a longer period, SQL Database keeps existing backups until the longer retention period is reached. 

If you delete a database, SQL Database keeps the backups in the same way it would for an online database. For example, suppose you delete a Basic database that has a retention period of seven days. A backup that is four days old is saved for three more days.

> [!IMPORTANT]
> If you delete the Azure SQL server that hosts SQL Databases, all databases that belong to the server are also deleted and cannot be recovered. You cannot restore a deleted server.

### Database Retention for the vCore-based purchasing model (preview)

Storage for database backups is allocated to support the Point in Time Restore (PITR) and Long Term Retention (LTR) capabilities of SQL Database. This storage is allocated separately for each database and billed as two separate per-database charges. 

- **PITR**: Individual database backups are copied to RA-GRS storage are automatically. The storage size increases dynamically as the new backups are created.  The storage is used by weekly full backups, daily differential backups, and transaction log backups copied every 5 minutes. The storage consumption depends on the rate of change of the database and the retention period. You can configure a separate retention period for each database between 7 and 35 days. A minimum storage amount equal to 1x of data size is provided at no extra charge. For most databases, this amount is enough to store 7 days of backups. For more information, see [Point-in-time restore](sql-database-recovery-using-backups.md#point-in-time-restore)
- **LTR**: SQL Database offers the option configuring long-term retention of full backups for up to 10 years. If LTR policy is enabled, theses backups are stored in RA-GRS storage automatically, but you can control how often the backups are copied. To meet different compliance requirement, you can select different retention periods for weekly, monthly and/or yearly backups. This configuration will define how much storage will be used for the LTR backups. You can use the LTR pricing calculator to estimate the cost of LTR storage. For more information, see [Long-term retention](sql-database-long-term-retention.md).

## How to extend the backup retention period?

If your application requires that the backups are available for longer period of time than the maximum PITR backup retention period, you can configure a Long-term backup retention policy for individual databases (LTR policy). This allows you to extend the built-it retention period from maximum 35 days to up to 10 years. For more information, see [Long-term retention](sql-database-long-term-retention.md).

Once you add the LTR policy to a database using Azure portal or API, the weekly full database backups will be automatically copied to a separate RA-GRS storage container for long-term retention (LTR storage). If your database is encrypted with TDE the backups are automatically encrypted at rest. SQL Database will automatically delete your expired backups based on their timestamp and the LTR policy. After the policy is set up, you don't need to manage the backup schedule or worry about the cleanup of the old files. You can use the Azure portal or PowerShell to view, restore, or delete these backups.

## Are backups encrypted?

When TDE is enabled for an Azure SQL database, backups are also encrypted. All new Azure SQL databases are configured with TDE enabled by default. For more information on TDE, see [Transparent Data Encryption with Azure SQL Database](/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql).

## Are the automatic backups compliant with GDPR?
If the backup contains personal data, which is subject to General Data Protection Regulation (GDPR), you are required to apply enhanced security measures to protect the data from unauthorized access. In order to comply with the GDPR, you need a way to manage the data requests of data owners without having to access backups.  For short-term backups, one solution can be to shorten the backup window to under 30 days, which is the time allowed to complete the data access requests.  If longer term backups are required, it is recommended to store only "pseudonymized" data in backups. For example, if data about a person needs to be deleted or updated, it will not require deleting or updating the existing backups. You can find more information about the GDPR best practices in [Data Governance for GDPR Compliance](https://info.microsoft.com/DataGovernanceforGDPRCompliancePrinciplesProcessesandPractices-Registration.html).

## Next steps

- Database backups are an essential part of any business continuity and disaster recovery strategy because they protect your data from accidental corruption or deletion. To learn about the other Azure SQL Database business continuity solutions, see [Business continuity overview](sql-database-business-continuity.md).
- To restore to a point in time using the Azure portal, see [restore database to a point in time using the Azure portal](sql-database-recovery-using-backups.md).
- To restore to a point in time using PowerShell, see [restore database to a point in time using PowerShell](scripts/sql-database-restore-database-powershell.md).
- To configure, manage, and restore from long-term retention of automated backups in an Azure Recovery Services vault using the Azure portal, see [Manage long-term backup retention using the Azure portal](sql-database-long-term-backup-retention-configure.md).
- To configure, manage, and restore from long-term retention of automated backups in an Azure Recovery Services vault using PowerShell, see [Manage long-term backup retention using PowerShell](sql-database-long-term-backup-retention-configure.md).
