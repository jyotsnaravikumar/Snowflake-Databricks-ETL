

## How to Configure databricks CLI from command line

```
pip install databricks-cli 

databricks configure --token 
 • Databricks Host (should begin with https://): (Enter you databricks host url here : 
        example https://adb-3589745579496882.2.azuredatabricks.net/ \
 • Token: (Enter you token here : example : dapif2e583021978735717ccfd4717f0f151) 

databricks secrets list-scopes 

databricks secrets create-scope --scope

create-scope --scope snowflake-trial

databricks secrets write --scope snowflake-trial --key username
 • opens vi editor - enter snowflake username as jyotsna19 and save

databricks secrets write --scope snowflake-trial --key password
 • open vi editor - enter snowflake password as d5HMkGcPSp7Ttjn and save

```

## Start with a basic ETL script to read a table from snowflake

(Snowflake ETL Docs) [https://docs.databricks.com/data/data-sources/snowflake.html]

```
user = dbutils.secrets.get("snowflake-trial", "username")
password = dbutils.secrets.get("snowflake-trial", "password")

options = {
  "sfUrl": "https://mi31170.west-us-2.azure.snowflakecomputing.com",
  "sfUser": user,
  "sfPassword": password,
  "sfDatabase": "CLINICAL_DB",
  "sfSchema": "PUBLIC",
  "sfWarehouse": "COMPUTE_WH"
}

df = spark.read \
  .format("snowflake") \
  .options(**options) \
  .option("query",  "select * from encounter_level_datamart") \
  .load()

df.show()

```

## Create a ADLS Gen2 Account to ingest data from snowflake to ADLSGen2
[https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal]

## Create a Service Principal to write to ADLSGen2 from databricks dataframe

1. Create a new App Registration give it a name as jyotsnar-app-databricks
2. Certificates and Secrets add a new secret with name jyotsnar-databricks-secret and note the clientid and clientsecret
3. Goto the ADLSGen2 -> Access Control (IAM) -> Add Role Assignment
    * Choose Role as Storage Blob Data Contributor
    * Assign access to as Azure AD, User, Service Principal
    * Select resource as the app that was created earlier as jyotsnar-app-databricks

4. Store the  clientid and clientsecret in the databricks scope to be used in the code

```
databricks secrets write --scope snowflake-trial --key spn-clientid

databricks secrets write --scope snowflake-trial --key spn-clientsecret
```

## Continue in the ETL script to read a table from snowflake and write it to ADLSGen2

```
clientid = dbutils.secrets.get("snowflake-trial", "spn-clientid")
servicecredential = dbutils.secrets.get("snowflake-trial", "spn-clientsecret")

configs = {"fs.azure.account.auth.type": "OAuth",
"fs.azure.account.oauth.provider.type": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
"fs.azure.account.oauth2.client.id": clientid,
"fs.azure.account.oauth2.client.secret": servicecredential,
"fs.azure.account.oauth2.client.endpoint": "https://login.microsoftonline.com/72f988bf-86f1-41af-91ab-2d7cd011db47/oauth2/token"}

dbutils.fs.mount(
  source = "abfss://databricks-output@jyotsnarstorage.dfs.core.windows.net/",
  mount_point = "/mnt/clinicalmount",
  extra_configs = configs)

dbutils.fs.ls("/mnt/clinicalmount") 

df.write.mode("overwrite").format("com.databricks.spark.csv").option("header", "true").csv("/mnt/clinicalmount/databricks_output/encounter_level_datamart.csv")

```

## Connect Snowflake to Databricks with Azure Key Vault-backed secret scope

* Steps done [https://medium.com/@cprosenjit/azure-databricks-with-azure-key-vaults-c00df6548222]
* Create Azure Key Vault-backed secret scope in Databricks [https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes]
