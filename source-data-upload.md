# How to upload sourcdata

### Overview
The sourcedata is packaged in a `tar.gz` file and send to an S3 object store.
The current store in use is `https://s3-nba-brondata.naturalis.nl/brondata-<source-system>`
An easy tool to use is `mc`, which is the `minio client`

### Installing Minio

On Linux
```
wget -q https://dl.minio.io/client/mc/release/linux-amd64/mc && chmod +x mc
```
For Mac / Windows and more info:
https://docs.minio.io/docs/minio-client-quickstart-guide

### Configuring Minio Client to S3 bucket
```
mc config host add s3-nba-brondata https://s3-nba-brondata.naturalis.nl <access id> <secret key>
```
The `access id` and `secret key` will be supplied

### Brahms
Go to the directory where the brahms CSV files are and run
```
tar -zcf brahms-<year>-<month>-<day of harvest>.tar.gz *.CSV
/path/to/mc cp brahms-<year>-<month>-<day of harvest>.tar.gz s3-nba-brondata/brondata-brahms/ 
```

### COL
Go to the directory where the brahms CSV files are and run
```
tar -zcf col-<year>.tar.gz *.txt
/path/to/mc cp col-<year>.tar.gz s3-nba-brondata/brondata-col/ 
```
