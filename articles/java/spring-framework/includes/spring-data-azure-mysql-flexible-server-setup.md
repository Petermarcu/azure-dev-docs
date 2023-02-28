---
author: KarlErickson
ms.date: 02/22/2023
ms.author: v-yonghuiye
---

## Configure a firewall rule for your MySQL server

Azure Database for MySQL instances are secured by default. They have a firewall that doesn't allow any incoming connection.

To be able to use your database, open the server's firewall to allow the local IP address to access the database server. For more information, see [Manage firewall rules for Azure Database for MySQL - Flexible Server using the Azure portal](/azure/mysql/flexible-server/how-to-manage-firewall-portal).

If you're connecting to your MySQL server from Windows Subsystem for Linux (WSL) on a Windows computer, you need to add the WSL host IP address to your firewall.

## Create a MySQL non-admin user and grant permission

This step will create a non-admin user and grant all permissions on the `demo` database to it.

### [Passwordless (Recommended)](#tab/passwordless)

> [!IMPORTANT]
> To use passwordless connections, create an Azure AD admin user for your Azure Database for MySQL instance. For more information, see the [Configure the Azure AD Admin](/azure/mysql/flexible-server/how-to-azure-ad#configure-the-azure-ad-admin) section of [Set up Azure Active Directory authentication for Azure Database for MySQL - Flexible Server](/azure/mysql/flexible-server/how-to-azure-ad).

Create a SQL script called *create_ad_user.sql* for creating a non-admin user. Add the following contents and save it locally:

```bash
AZ_MYSQL_AD_NON_ADMIN_USERID=$(az ad signed-in-user show --query id -o tsv)

cat << EOF > create_ad_user.sql
SET aad_auth_validate_oids_in_tenant = OFF;
CREATE AADUSER '<your_mysql_ad_non_admin_username>' IDENTIFIED BY '$AZ_MYSQL_AD_NON_ADMIN_USERID';
GRANT ALL PRIVILEGES ON demo.* TO '<your_mysql_ad_non_admin_username>'@'%';
FLUSH privileges;
EOF
```

Then, use the following command to run the SQL script to create the Azure AD non-admin user:

```bash
mysql -h mysqlflexibletest.mysql.database.azure.com --user <your_mysql_ad_admin_username> --enable-cleartext-plugin --password=$(az account get-access-token --resource-type oss-rdbms --output tsv --query accessToken) < create_ad_user.sql
```

> [!TIP]
> To use Azure AD authentication to connect to Azure Database for MySQL, you need to sign in with the Azure AD admin user you set up, and then get the access token as the password. For more information, see [Set up Azure Active Directory authentication for Azure Database for MySQL - Flexible Server](/azure/mysql/flexible-server/how-to-azure-ad).

### [Password](#tab/password)

Create a SQL script called *create_user.sql* for creating a non-admin user. Add the following contents and save it locally:

```bash
cat << EOF > create_user.sql
CREATE USER '<your_mysql_non_admin_username>'@'%' IDENTIFIED BY '<your_mysql_non_admin_password>';
GRANT ALL PRIVILEGES ON demo.* TO '<your_mysql_non_admin_username>'@'%';
FLUSH PRIVILEGES;
EOF
```

Then, use the following command to run the SQL script to create the non-admin user:

```bash
mysql -h mysqlflexibletest.mysql.database.azure.com --user <your_mysql_admin_username> --enable-cleartext-plugin --password=<your_mysql_admin_password> < create_user.sql
```

> [!TIP]
> You can read more detailed information about creating MySQL users in [Create users in Azure Database for MySQL](/azure/mysql/single-server/how-to-create-users).

---

## Store data from Azure Database for MySQL

Now that you have an Azure Database for MySQL Flexible server instance, you can store data by using Spring Cloud Azure.

To install the Spring Cloud Azure Starter JDBC MySQL module, add the following dependencies to your *pom.xml* file:

- The Spring Cloud Azure Bill of Materials (BOM):

  ```xml
   <dependencyManagement>
     <dependencies>
       <dependency>
         <groupId>com.azure.spring</groupId>
         <artifactId>spring-cloud-azure-dependencies</artifactId>
         <version>4.5.0</version>
         <type>pom</type>
         <scope>import</scope>
         </dependency>
     </dependencies>
   </dependencyManagement>
  ```

- The Spring Cloud Azure Starter JDBC MySQL artifact:

  ```xml
  <dependency>
    <groupId>com.azure.spring</groupId>
    <artifactId>spring-cloud-azure-starter-jdbc-mysql</artifactId>
  </dependency>
  ```

> [!NOTE]
> Passwordless connections have been supported since version `4.5.0`.