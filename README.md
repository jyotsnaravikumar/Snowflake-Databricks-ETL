# Secured data ingestion(ELT) from an External Snowflake Account into Azure Blob store using Databricks secrets and Azure Keyvault

## Use secrets to keep data secure
Sometimes accessing data requires that you authenticate to external data sources through JDBC. 
Instead of directly entering your credentials into a notebook, use Azure Databricks secrets to store your credentials and reference them in notebooks and jobs. 

## Types of secrets
There are 2 types of secrets 

* Azure KeyVault backed scopes
* Databricks backed scopes 

We would prefer using Azure Keyvault backed scopes whenever possible because it has much better security controls than
Databricks backed scope which would only be for testing.

## Create a Azure Keyvault backed scope

1. [Create a Azure KeyVault instance](https://docs.microsoft.com/en-us/azure/key-vault/secrets/quick-create-portal) 
2. [Add Contributor permission on the Azure Key Vault instance that you want to use to back the secret scope](
   https://docs.microsoft.com/en-us/azure/key-vault/general/rbac-guide)
3. [Add secrets](https://docs.microsoft.com/en-us/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope)

4. Use the Databricks CLI databricks secrets list-scopes command to verify that the scope was created successfully.

```shell
  databricks configure --token
  Databricks Host: https://<databricksurl>.azuredatabricks.net/
  Token: <token>		
  databricks secrets list-scopes
```

## ETL script
[Create a snowflake-databricks-etl notebook in Azure Databricks](notebooks/snowflake-databricks-with-keyvault.ipynb)

## References
https://docs.databricks.com/data/data-sources/snowflake.html
