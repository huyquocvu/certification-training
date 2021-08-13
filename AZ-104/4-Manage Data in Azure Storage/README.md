# Manage Data in Azure Storage


## Migrating Data info Azure
- Over the wise
    - Use for small data sets
- Ship physical media
    - Use for large data sets

### Data Management Tools
- Azure Import/Export Service
- AzCopy
    - CLI tool
- Azure Storage Explorer

# Azure Import/Export Service
- Securely Import/export large amounts of data with physical drives
- Create jobs in Azure Portal or Azure Resource Manager REST API
- Import to Azure BlobStorage and Azure Files
- Export to Azure BlobStorage
- Supports General Purpose v1, v2, and BlobStorage storage accounts
#### Use cases
- Backups
- Content Distribution
- Data Recovery

### Azure Import Export Service Components
- Azure Import/Export Service
    - Azure portal or REST API
        - Create job for working with data you need to move
- WAImportExportTool
    - CLI tool to prepare and manage drives used for import/export jobs
- Drives
    - Two options
        - Self-provided
        - Azure Data Box - MS provides the drives

### Importing Blobs and Files to Azure
- Drives prepped with WAImportExportTool and Bitlocker
    1. Determine dataset
    2. Copy data to drive
    3. Encrypt with Bitlocker
    4. MUST be NTFS format
- Create Import job in Azure portal
- Ship drives to Azure Datacenter
- Processed drive data copied to storage account and ready for verification
- Drives returned to customer

### Exporting Blobs from Azure
- Create export job in Azure Portal
- Ship Drive to Datacenter
- Exported Data copied to encrypted drive
- Drive shipped to customer
- Unlock and verify drive with WAImportExport Unlock command

### WAImportExportTool
```PowerShell
# Parameters
.\WAImportExport.exe PrepImport
    /j:<JournalFile>
    /id:<SessionId>
    [/logdir:<LogDirectory>]
    [/sk:<StorageAccountKey>] [/silentmode]
    [/InitialDriveSet:<driveset.csv>]
    /DataSet:<dataset.csv>
```

### AzCopy
