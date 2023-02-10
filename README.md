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
