# Docker MSSQL


> The password should follow the SQL Server default password policy, otherwise the container can not setup SQL server and will stop working. By default, the password must be at least 8 characters long and contain characters from three of the following four sets: Uppercase letters, Lowercase letters, Base 10 digits, and Symbols. You can examine the error log by executing the docker logs command.

```PowerShell
# Windows
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong!Passw0rd>" `
   -p 1433:1433 --name mssql `
   -d mcr.microsoft.com/mssql/server:2017-latest
```

## 在 Mac 上安装 MSSQL
```sh
# Mac
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=Password-123" -p 1433:1433 -v ~/Backup:/Backup --name mssql -d mcr.microsoft.com/mssql/server:2017-latest
```

[Parameter Description](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-2017)

```
db:
        image: "mcr.microsoft.com/mssql/server"
        environment:
            SA_PASSWORD: "Your_password123"
            ACCEPT_EULA: "Y"
```

### 登录容器查询数据库
```
docker exec -it mssql bash
```

```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'Password-123'
```

## 安装 sqlcmd 工具
```sh
# Mac
brew tap microsoft/mssql-release https://github.com/Microsoft/homebrew-mssql-release
brew update
brew install --no-sandbox msodbcsql17 mssql-tools
```

```
# Ubuntu
sudo su 
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -

#Ubuntu 18.04
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > /etc/apt/sources.list.d/mssql-release.list

exit
apt-get update
ACCEPT_EULA=Y apt-get install msodbcsql17
# optional: for bcp and sqlcmd
ACCEPT_EULA=Y apt-get install mssql-tools
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
# optional: for unixODBC development headers
apt-get install unixodbc-dev
```
[See Installing the Microsoft ODBC Driver for SQL Server on Linux and macOS](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-2017)