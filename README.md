#### UC Cloner

Simple proof of concept of how to perform the cloning of an existing UrbanCode Deploy and Release environment for setting up staging/testing areas.

### Caveats

This example is very limited in scope and functionality but is a practical implementation for reference on Linux platforms. The intent is to provide a minimum viable product to show the end to end process.

### Pre-Requisite Setup

This example was done on a single machine where I built out a UCD/UCR setup, integrated the two, and configured them to be functional. This was all done in my case on Ubuntu Trusty with MySQL 5.7, using the default-jre-headless package. Used `rsa-keygen` to create a pub/private key and configured so it could login to itself via ssh without prompt.

I created the initial machine(uc1) in Vagrant and then manually cloned to create another machine(uc2) when I was ready to perform the clone. So after creating the target machine I did the required changes to re-initialize the MAC address and configure a new IP address for them to co-exist on the same network and communicate with each other.

### Backup Scenarios

The backup process is pretty simple and consists of two types of backups. The complete backup and an incremental backup.

  1. Take a complete backup of an UrbanCode System(s)
  Common scenario here, that outlines the supported backup procedure as recommended in the product documentation, which requires the server to be down to perform an a complete backup.

  Should be a procedure that is completed on a regular basis, based on your infrastructure standards this could be a weekly/bi-weekly/monthly type cadence. For integrated tools, planning backups and downtimes together is logical as they can be treated as a set.

   * **UrbanCode Deploy**
      1. Stop all Deploy servers.
      1. Backups to take: (can be done in parallel)
       * Foreach server, backup `server installation` directory to capture configurations.
       * Backup `appdata` location to capture process execution logs and CodeStation artifact versions.
       * Full backup of Deploy database
      1. Start all Deploy servers.

   * **UrbanCode Release**
      1. Stop all Release servers.
      1. Backups to take: (can be done in parallel)
        * Foreach server, backup `server installation` directory to capture configuration and installed plugins.
        * Full backup of the Release database.
      1. Start all Release servers.

  1. Take an incremental backup of the UrbanCode System(s)
  Requires that we have enabled binary/transaction logging on our database system, or have a vendor specific solution for logging changes that can be applied to _catch up_ a earlier version of the same database.

  This should be a very regular automated job, and should be happening every 15-30m to ensure there is limit the amount of data loss if a failure event does occur. Since these are happening very often it is less important to try and coordinate them at the same time as they should be no more than 15-30m apart in the worst case here.

   * **UrbanCode Deploy**
     1. Incremental backup event triggered.
     1. Backups to take: (Should be done in parallel)
       * Synchronize current transaction log files with offsite.
       * Synchronize current `appdata` files with backup location.
       * Foreach server, synchronize current `server installation directory` files with offsite.

   * **UrbanCode Release**
     1. Incremental backup event triggered.
     1. Backups to take: (Should be done in parallel)
      * Synchronize current transaction log files with backup location.
      * Foreach server, synchronize current `server installation directory` files with offsite.

### Restore Scenarios

While thinking about restoring we assume that the backup was simply performing a clone. So when we decide to restore this data to another environment we need to take into account at least two scenarios described further below where the environments will run in parallel (aka clone to test) and where this is a cold copy of production ( standard backup or source for Disaster Recovery).

 1. A complete restore to Production, when production fails and we just want to recover.
 1. A complete restore targeted DR, when production fails and we need to start a DR event.
 1. A complete restore to another target environment, when we want to do a full data refresh of a target environment for any other reason.

Additionally we have a variant of these scenarios for restoring incremental changes.

 1. An incremental restore to Production, in the event that we have a failure or data loss that requires we restore to our last complete backup and replay incremental changes to the point in time we desire..
 1. An incremental restore to Disaster Recovery, when we want to _catch up_ DR, either during a DR event or just as a regular part of the DR readiness plan.
 1. An incremental restore to a target environment, when we want to refresh the data in our target environment with live data from production.

### Usage

Source server:

    ./full_backup.sh

If no shared directory is used, you must then sync your data dir between machines:

    ./sync_data_dir.sh

Target server:

    ./full_restore.sh ${data_dir}/F20140404-2113/

During the restore you are prompted to run some SQL against the UCR database, and a properties file replacement script against the new files that have been layed down.

### Next Steps:

Still focusing on only Linux targets, provide a more robust skeleton that can run in a slightly more complex environment.

  * Add steps to change HA Network on restore to differnet environment
  * Incremental backups

#### Reference:

 * [ibm.com - What is the best way to clone the IBM Urban Code Deploy server and database for testing upgrades?](http://www-01.ibm.com/support/docview.wss?uid=swg21694427)
