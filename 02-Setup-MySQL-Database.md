# Objectives
Our sample application uses a MySQL database.
In later sections, we will demostrate the hardcoded versus secretless connection methods used by the sample application to connect to this MySQL database.

# Setup the MySQL Database

We have provisioned the utilities host and setup Docker in task 0

1.0 Download the world database

We will leverage on the sample world database from MySQL: https://dev.mysql.com/doc/world-setup/en/

Change directory to the azureuser home directory
```console
cd ~
```

Download and extract the sample world datacase
```console
curl https://downloads.mysql.com/docs/world-db.tar.gz -o world-db.tar.gz
tar xvf world-db.tar.gz
```

Verify that the world.sql file exists:
```console
file /home/azureuser/world-db/world.sql
```

2.0 Provision the MySQL container on the utilities host

Run the MySQL container with below options:
- --name mysqldb # name of the MySQL container
- -v /home/azureuser/world-db:/docker-entrypoint-initdb.d # volume map the /home/azureuser/world-db to /docker-entrypoint-initdb.d
- -e MYSQL_ROOT_PASSWORD=Cyberark1 # sets MySQL root password to Cyberark1
- -e MYSQL_DATABASE=world # specify the name of a database to be created on image startup
- -e MYSQL_USER=cityapp # creates a user named cityapp
- -e MYSQL_PASSWORD=Cyberark1 # set the password of the user to Cyberark1
- -p "3306:3306" # expose the MySQL port on the Docker host
- -d mysql:5.7.35 # specify the container image with tag "5.7.35"

```console
docker run --name mysqldb -v /home/azureuser/world-db:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=Cyberark1 -e MYSQL_DATABASE=world -e MYSQL_USER=cityapp -e MYSQL_PASSWORD=Cyberark1 -p "3306:3306" -d mysql:5.7.35
```

3.0 Check if your mysql contrainer running correctly

```console
docker ps
```

Sample output:
```console
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
32361ae8dbfe   mysql:5.7.35   "docker-entrypoint.s…"   12 seconds ago   Up 10 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   mysqldb
```

4.0 Verify that the MySQL database is running

Attach to the container bash shell
```console
docker exec -it mysqldb bash
```

Sample container bash shell:
```console
root@32361ae8dbfe:/#
```

Login to the MySQL database:
```console
mysql -u cityapp -p
```
Enter the configured user password on the password prompt

Select the world database and list the tables:
```console
use world;
show tables;
```

Sample output:
```console
mysql> use world;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_world |
+-----------------+
| city            |
| country         |
| countrylanguage |
+-----------------+
3 rows in set (0.01 sec)
```
