# Azure Database for MySQL Terraform Module

Azure Database for MySQL is easy to set up, manage and scale. It automates the management and maintenance of your infrastructure and database server, including routine updates, backups and security. Enjoy maximum control of database management with custom maintenance windows and multiple configuration parameters for fine grained tuning with Flexible Server (Preview).

## Module Usage

```hcl
module "mysql-db" {
  source  = "kumarvna/mysql-db/azurerm"
  version = "1.1.0"

  # By default, this module will create a resource group
  # proivde a name to use an existing resource group and set the argument 
  # to `create_resource_group = false` if you want to existing resoruce group. 
  # If you use existing resrouce group location will be the same as existing RG.
  create_resource_group = false
  resource_group_name   = "rg-shared-westeurope-01"
  location              = "westeurope"

  # MySQL Server and Database settings
  mysqlserver_name = "mysqldbsrv01"

  mysqlserver_settings = {
    sku_name   = "GP_Gen5_16"
    storage_mb = 5120
    version    = "5.7"
    # default admin user `sqladmin` and can be specified as per the choice here
    # by default random password created by this module. required password can be specified here
    admin_username = "sqladmin"
    admin_password = "H@Sh1CoR3!"
    # Database name, charset and collection arguments  
    database_name = "demomysqldb"
    charset       = "utf8"
    collation     = "utf8_unicode_ci"
    # Storage Profile and other optional arguments
    auto_grow_enabled                 = true
    backup_retention_days             = 7
    geo_redundant_backup_enabled      = false
    infrastructure_encryption_enabled = false
    public_network_access_enabled     = true
    ssl_enforcement_enabled           = true
    ssl_minimal_tls_version_enforced  = "TLS1_2"
  }

  # MySQL Server Parameters
  # For more information: https://docs.microsoft.com/en-us/azure/mysql/concepts-server-parameters
  mysql_configuration = {
    interactive_timeout = "600"
  }

  # Use Virtual Network service endpoints and rules for Azure Database for MySQL
  subnet_id = var.subnet_id

  # The URL to a Key Vault custom managed key
  key_vault_key_id = var.key_vault_key_id

  # Creating Private Endpoint requires, VNet name and address prefix to create a subnet
  # By default this will create a `privatelink.mysql.database.azure.com` DNS zone. 
  # To use existing private DNS zone specify `existing_private_dns_zone` with valid zone name
  enable_private_endpoint       = true
  virtual_network_name          = "vnet-shared-hub-westeurope-001"
  private_subnet_address_prefix = ["10.1.5.0/29"]
  #  existing_private_dns_zone     = "demo.example.com"

  # To enable Azure Defender for database set `enable_threat_detection_policy` to true 
  enable_threat_detection_policy = true
  log_retention_days             = 30
  email_addresses_for_alerts     = ["user@example.com", "firstname.lastname@example.com"]

  # AD administrator for an Azure MySQL server
  # Allows you to set a user or group as the AD administrator for an Azure SQL server
  ad_admin_login_name = "firstname.lastname@example.com"

  # (Optional) To enable Azure Monitoring for Azure MySQL database
  # (Optional) Specify `storage_account_name` to save monitoring logs to storage. 
  log_analytics_workspace_name = "loganalytics-we-sharedtest2"

  # Firewall Rules to allow azure and external clients and specific Ip address/ranges. 
  firewall_rules = {
    access-to-azure = {
      start_ip_address = "0.0.0.0"
      end_ip_address   = "0.0.0.0"
    },
    desktop-ip = {
      start_ip_address = "49.204.228.223"
      end_ip_address   = "49.204.228.223"
    }
  }

  # Tags for Azure Resources
  tags = {
    Terraform   = "true"
    Environment = "dev"
    Owner       = "test-user"
  }
}
```

## Default Local Administrator and the Password

This module utilizes __`sqladmin`__ as a local administrator on MySQL server. If you want to you use custom username, then specify the same by setting up the argument `admin_username` with a valid user string.

By default, this module generates a strong password for MySQL server also allows you to change the length of the random password (currently 24) using the `random_password_length` variable. If you want to set the custom password, specify the argument `admin_password` with a valid string.

## `mysql_setttings` - Setting up your MySQL Server

This object helps you setup desired MySQL server and support following arguments.

| Argument | Description |
|--|--|
`sku_name`|Specifies the SKU Name for this MySQL Server. The name of the SKU, follows the tier + family + cores pattern (e.g. `B_Gen4_1`, `GP_Gen5_8`). Valid values are `B_Gen4_1`, `B_Gen4_2`, `B_Gen5_1`, `B_Gen5_2`, `GP_Gen4_2`, `GP_Gen4_4`, `GP_Gen4_8`, `GP_Gen4_16`, `GP_Gen4_32`, `GP_Gen5_2`, `GP_Gen5_4`, `GP_Gen5_8`, `GP_Gen5_16`, `GP_Gen5_32`, `GP_Gen5_64`, `MO_Gen5_2`, `MO_Gen5_4`, `MO_Gen5_8`, `MO_Gen5_16`, `MO_Gen5_32`.
`storage_mb`|Max storage allowed for a server. Possible values are between `5120` MB(5GB) and `1048576` MB(1TB) for the Basic SKU and between `5120` MB(5GB) and `4194304` MB(4TB) for General Purpose/Memory Optimized SKUs.
`version`|Specifies the version of MySQL to use. Valid values are `5.6`, `5.7`, and `8.0`.
`database_name`|Specifies the name of the MySQL Database, which needs [to be a valid MySQL identifier](https://dev.mysql.com/doc/refman/5.7/en/identifiers.html).
`charset`|Specifies the Charset for the MySQL Database, which needs [to be a valid MySQL Charset](https://dev.mysql.com/doc/refman/5.7/en/charset-charsets.html).
`collation`|Specifies the Collation for the MySQL Database, which needs [to be a valid MySQL Collation](https://dev.mysql.com/doc/refman/5.7/en/charset-mysql.html).
`administrator_login`|The Administrator Login for the MySQL Server. Required when `create_mode` is `Default`.
`auto_grow_enabled`|Enable/Disable auto-growing of the storage. Storage auto-grow prevents your server from running out of storage and becoming read-only. If storage auto grow is enabled, the storage automatically grows without impacting the workload. The default value if not explicitly specified is `true`
`backup_retention_days`|Backup retention days for the server, supported values are between `7` and `35` days.
`geo_redundant_backup_enabled`|urn Geo-redundant server backups on/off. This allows you to choose between locally redundant or geo-redundant backup storage in the General Purpose and Memory Optimized tiers. When the backups are stored in geo-redundant backup storage, they are not only stored within the region in which your server is hosted, but are also replicated to a paired data center. This provides better protection and ability to restore your server in a different region in the event of a disaster. This is not supported for the `Basic` tier.
`infrastructure_encryption_enabled`|Whether or not infrastructure is encrypted for this server. Defaults to `false`
`public_network_access_enabled`|Whether or not public network access is allowed for this server. Defaults to `true`.
`ssl_enforcement_enabled`|Specifies if SSL should be enforced on connections. Possible values are `true` and `false`
`ssl_minimal_tls_version_enforced`|The minimum TLS version to support on the sever. Possible values are `TLSEnforcementDisabled`, `TLS1_0`, `TLS1_1`, and `TLS1_2`. Defaults to `TLSEnforcementDisabled`.