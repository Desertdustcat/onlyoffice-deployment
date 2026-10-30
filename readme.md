# My ONLYOFFICE-workspace docker Installation
https://github.com/ONLYOFFICE/Docker-CommunityServer

This is just my personal documentaion of how i installed onlyoffice on a proxmox LXC on my homeserver. All the commands while be as root, so do what that information what you want(maybe add `sudo`, idk). I'm running an alpine LXC with nano, docker compose and docker installed. Changes i've made, away from the offical install:
- No Mail server
- moved Passwords to .env
- newer Versions
- other smaller things
- ...

## Installation
### 1. Clone this repo
To start, clone this github repo into a folder youd like to have you docker compose files.
```sh
git clone https://github.com/Desertdustcat/onlyoffice-deployment.git 
```
and rename env.example to .env
```sh
cd onlyoffice-deployment
mv env.example .env
```
### 2. Change the Passwords
Change the Passwords in .env, use `openssl rand -hex 32` to get secure passwords.
- `APPDATA_PATH` – Change data path, default = `app/openoffice`
- `MYSQL_ROOT_PASSWORD` – Set a password for the SQL root account
- `ONLYOFFICE_DB_USER` – Optional: change user, best leave default = `onlyoffice_user`
- `ONLYOFFICE_DB_PASSWORD` – Set a password for the onlyoffice_user DB
- `ONLYOFFICE_MACHINEKEY` – Set a password, shared between Community server & Control panel
- `ONLYOFFICE_JWT_SECRET` – Set a password, shared between Community server & Document server

## Creating needed folders and Network
Onlyoffice needs a few folders, namely:
```
    # MySQL
/app/onlyoffice/mysql/conf.d
/app/onlyoffice/mysql/data
/app/onlyoffice/mysql/initdb
    # community server – data and logs
/app/onlyoffice/CommunityServer/data
/app/onlyoffice/CommunityServer/logs
    # docmunet server — data and logs
/app/onlyoffice/DocumentServer/data
/app/onlyoffice/DocumentServer/logs
    # controlpanel
/app/onlyoffice/ControlPanel/data
/app/onlyoffice/ControlPanel/logs
```
all those folders can be quickly added with this command, replace the installation directiory if you'd like (do not forget to change it in .env too):
```sh
mkdir -p /app/onlyoffice/{mysql/{conf.d,data,initdb},{CommunityServer/,DocumentServer/,ControlPanel/}{data,logs}}
```
ONLYOFFICE also needs a docker network
```sh
docker network create --driver bridge onlyoffice
```
## Installing MySQL
For the proper Installation, two files need to have data.

Add this to `/app/onlyoffice/mysql/conf.d/onlyoffice.cnf`:
```ini
[mysqld]
sql_mode = 'NO_ENGINE_SUBSTITUTION'
max_connections = 1000
max_allowed_packet = 1048576000
group_concat_max_len = 2048
```
and this to `/app/onlyoffice/mysql/initdb/setup.sql` and change <MYSQL_ROOT_PASSWORD> and <ONLYOFFICE_DB_PASSWORD> to the password in .env so they match:
>if you changed your user in .env change that here to.

```sql
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '<MYSQL_ROOT_PASSWORD>';
CREATE USER IF NOT EXISTS 'onlyoffice_user'@'%' IDENTIFIED WITH mysql_native_password BY '<ONLYOFFICE_DB_PASSWORD>';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
GRANT ALL PRIVILEGES ON *.* TO 'onlyoffice_user'@'%';
FLUSH PRIVILEGES;
```
now, as the last thing, set the folders user permissions:
```sh
chmod 644 /app/onlyoffice/mysql/{conf.d/onlyoffice.cnf,/initdb/setup.sql}
```
