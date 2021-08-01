## Migrate Databases from SQL-Server On-Prem to SQL-Server on Azure VM

- **Tools for Migration from MS:**
  - Microsoft Assessment and Planung Toolkit(MAP)
  - Data Migration Assistant(DMA)
  - Database Migration Service(DMS on Azure Portal)

**Scenario:**

We would migrate databases from SQL-Server on-prem to SQL Server on Azure VM, we need to simulate the migration process(see diagramm Migrate SQL-Server):
- On-Prem: set up VM with Windows Server 2012 R2 with SQL-Server 2014 with developer licence on azure
- On Azure: set up VM with Windows Server 2016 with SQL-Server 2017
- Create Blob Storage with file Share to store DB-Backups during migration
- Set up Data Migration Assistant on the machine which database are running and will be migrated(in this case on Windows-Server 12 R2)
  - Run this comand after setting up the DMA to get get no problems with firewall during the migration process<br/>
    cmd: netsh advfirewall set currentprofile settings remotemanagement enable

![alt text](https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Diagramm/SQL-Migration%20daigramm.jpg?row=true "Diagramm SQL-Migration")

**Screenshot for creating on-prem Enviroment and Azure Enviroment**
<img src="https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Pictures/SQL-Server_14%20on%20Windows%20Server_12_r2_.JPG" width="650" height="450">
<img src="https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Pictures/SQL-Server_17%20on%20Windows%20Server_16.JPG" width="650" height="450">
<img src="https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Pictures/SQL-Server%20Settings%20by%20Creation.JPG" width="650" height="450">

**Steps for DB-Migration**
- Create an Assessment with Microsoft Assessment and Planung tools
  - Check if there are any issues or compatibility problems
-  Create Assessment with Data Migration Assistant(DMA)
-  Create FileShare on Blobstorage to save DB-backup
-  Do Migtration

**Screenshot for DB-Assessment**
<img src="https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Pictures/Report%20of%20DB-Assessment.JPG" width="850" height="450">
<img src="https://github.com/MElashkr/Azure-Labs/blob/main/Data-Migration/Pictures/Result%20migration%20for%20SQL-DB%20by%20DMA.JPG" width="750" height="180">

**How Handle Erros:**<br/>
If you get this error during migration process in DMA: Migration Could not open a connection to SQL Server) The network path was not found.
That because, you need to adapt the name of target server(in this case Windows Server 16 on Azure) on DMA. Create DNS name for vm on Azure and set it as Server name in **1 Specify Source and Target Step**
And enable public internet in choose VM -> SQL-Configuration-> click Manage SQL Virtual machine -> Security Configuration -> choose SQL connectivity as public instead of private & enable SQL-Authenication

**Define a share-file for both SQL-Servers**<br/>
Create a file-share to Save backup in Blob-Storage on cloud 
You get this info from "Connect" of file-share(right click and choose connect). Adapt xp_cmdshell based on your Configuration
Run this DB-Script separate for both DB on-prem and azure SQL to define a file-share

```
SP_CONFIGURE 'show advanced options', 1
RECONFIGURE

SP_CONFIGURE 'xp_cmdshell', 1
RECONFIGURE
GO

exec xp_cmdshell 'cmdkey /add:test.file.core.windows.net /user:localhost\test /pass:safafa....BgcTXgsploMY3cQ=='

exec xp_cmdshell 'net use Z: \\test.file.core.windows.net\test-sharefile01 /persist:Yes'
exec xp_cmdshell 'dir z:'

SP_CONFIGURE 'xp_cmdshell', 0
RECONFIGURE
```
