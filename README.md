# docker-compose-oracle-xe21c-ords
Docker Compose file for Oracle xe21c Database and Ords / Apex

## Introduction

This docker compose yaml file is used to build a docker container stack for Oracle Database Express Edition 21c and Oracle REST Data Services (ORDS) with Oracle Application Express (APEX)

The required containers are available from the [Oracle Docker Registry](https://container-registry.oracle.com)

The build of the database container is not mapped to a host system volume, so there is no database persistence.  If you need database persistence then this docker compose file will need to be updated accordingly.

## Prerequisites

The docker compose file assumes that there is a mapped volume from the local host machine which will hold a text file called 'conn_string.txt' which will be used by the ORDS container to obtain a connection to the database in which to install APEX and ORDS.

As an example, here is a directory structure:

```
├── User
│   ├── JohnDoe
│       ├── Docker
│           ├── Compose
│               ├── Oracle
│               │   ├── docker-compose.yml
│               ├── ords_secret
│                   ├── conn_string.txt
```

## Updates required to docker-compose.yml

The database container requires you to update the environment variable that will be used to define the password for SYS, SYSTEM and PDB_ADMIN, this is on line 6 of the yaml file, change **\<password\>** to something more secure.

```
environment:
  - ORACLE_PWD=<password>
```

The ORDS container requires you to update the volume mapping to the local directory 'ords_secret' so that the file 'conn_string.txt' can be read to make connection to the database to complete the ORDS and APEX installation, this is on line 31.

```
volumes:
  - <Update to your directory holding ORDS secret>:/opt/oracle/variables
```

**IMPORTANT**

As the ORDS container needs to wait for the Database container to complete and finish the build before it can commence, the ORDS container needs to be delayed starting.

I've had mix success with this and the yaml file needs to be updated, I have tried to set a healthcheck on DB container and a depends_on on the ORDS container, this can be seen in lines 14 through to 18 on the DB service and lines 22 through to 24 on the ORDS service.

DB Service healthcheck

```
healthcheck:
  test: [ "CMD", "bash", "-c", "echo 'select 1 from dual;' | ORACLE_HOME=/opt/oracle/product/21c/dbhomeXE /opt/oracle/product/21c/dbhomeXE/bin/sqlplus -s USERNAME/PASSWORD@localhost"]
  interval: 30s
  retries: 60
  timeout: 60s
```

ORDS Service depends_ons

```
depends_on:
  db:
    condition: service_healthy
```

Occassionally the ORDS service will start before the DB service has finished, which makes the ORDS service fail as the database is not yet up so a connection can not be made.  To get around this I mess around with the interval and timeout commands.

Based on the above, I have a suspicion that the problem might be in the actual test and that the interval is what is making this work.
