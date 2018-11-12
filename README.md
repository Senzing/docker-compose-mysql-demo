# docker-compose-mysql-demo

## Overview

This docker formation brings up the following docker containers:

1. *mysql*
1. *phpmyadmin/phpmyadmin*
1. *senzing/python-demo*

Also shown in the demonstration are commands to run the following Docker images:

1. *senzing/mysql* in [Add Senzing schemas](#add-senzing-schemas)
1. *senzing/g2loader* in [Add content](#add-content)
1. *senzing/g2command* in [Run G2Command.py](#run-g2commandpy)

### Contents

1. [Preparation](#preparation)
    1. [Set environment variables](#set-environment-variables)
    1. [Clone repository](#clone-repository)
    1. [Software](#software)
    1. [Docker images](#docker-images)
1. [Run Docker formation](#run-docker-formation)
    1. [Create SENZING_DIR](#create-senzing_dir)
    1. [Set environment variables](#set-environment-variables)
    1. [Launch docker formation](#launch-docker-formation)
    1. [Add Senzing schemas](#add-senzing-schemas)
    1. [Add content](#add-content)
    1. [Run G2Command.py](#run-g2commandpy)
1. [Cleanup](#cleanup)

### Shortcut

The following is the full description of the demonstration.
For a "short cut" version of the command, see
[demo-shortcuts](doc/demo-shortcuts.md).

## Preparation

### Set environment variables

These variables may be modified, but do not need to be modified.
The variables are used throughout the installation procedure.

```console
export GIT_ACCOUNT=senzing
export GIT_REPOSITORY=docker-compose-mysql-demo
```

Synthesize environment variables.

```console
export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
export GIT_REPOSITORY_URL="https://github.com/${GIT_ACCOUNT}/${GIT_REPOSITORY}.git"
```

### Clone repository

Get repository.

```console
mkdir --parents ${GIT_ACCOUNT_DIR}
cd  ${GIT_ACCOUNT_DIR}
git clone ${GIT_REPOSITORY_URL}
```

### Software

The following software programs need to be installed.

#### docker

```console
docker --version
docker run hello-world
```

#### docker-compose

```console
docker-compose --version
```

### Docker images

The following docker images need to be installed.

This is the short-cut for installing all of the Senzing docker images:

```console
docker build --tag senzing/mysql       https://github.com/senzing/docker-mysql.git
docker build --tag senzing/python-base https://github.com/senzing/docker-python-base.git
docker build --tag senzing/python-demo https://github.com/senzing/docker-python-demo.git
docker build --tag senzing/g2loader    https://github.com/senzing/docker-g2loader.git
docker build --tag senzing/g2command   https://github.com/senzing/docker-g2command.git
```

#### senzing/mysql

```console
docker run -it senzing/mysql --version
```

If docker image not available, create it by following instructions at
[github.com/Senzing/docker-mysql](https://github.com/Senzing/docker-mysql#build-docker-image).

#### senzing/g2loader

```console
$ docker run -it senzing/g2loader --projectFile /dummy-file

Using internal database
python: can't open file 'G2Loader.py': [Errno 2] No such file or directory
```

*Note:* this is a hack as `--version` option does not exist.

If docker image not available, create it by following instructions at
[github.com/Senzing/docker-g2loader](https://github.com/Senzing/docker-g2loader#build-docker-image).

#### senzing/g2command

```console
$ docker run -it senzing/g2command

Using internal database
python: can't open file 'G2Command.py': [Errno 2] No such file or directory
```

*Note:* this is a hack as `--version` option does not exist.

If docker image not available, create it by following instructions at
[github.com/Senzing/docker-g2command](https://github.com/Senzing/docker-g2loader#build-docker-image).

#### senzing/python-demo

```console
docker run -it senzing/python-demo --version
```

If docker image not available, create it by following instructions at
[github.com/Senzing/docker-python-demo](https://github.com/Senzing/docker-python-demo#build-docker-image).

## Run Docker formation

### Create SENZING_DIR

If you do not already have an `/opt/senzing` directory on your local system, visit
[HOWTO - Create SENZING_DIR](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/create-senzing-dir.md).

### Set environment variables for docker

1. **SENZING_DIR** -
   Path on the local system where
   [Senzing_API.tgz](https://s3.amazonaws.com/public-read-access/SenzingComDownloads/Senzing_API.tgz)
   has been extracted.
   See [Create SENZING_DIR](#create-senzing_dir).
   No default.
   Usually set to "/opt/senzing".
1. **MYSQL_ROOT_PASSWORD** -
   The password for the the database "root" user name.
   Default: "root"
1. **MYSQL_STORAGE** -
   Path on local system where the database files are stored.
   Default: "/storage/docker/senzing/docker-compose-mysql-demo"
1. See [github.com/Senzing/docker-mysql](https://github.com/Senzing/docker-mysql)
   for more details on how to find values for other environment variables.
1. Example:

    ```console
    export SENZING_DIR=/opt/senzing

    export MYSQL_HOST=senzing-mysql
    export MYSQL_DATABASE=G2
    export MYSQL_ROOT_PASSWORD=root
    export MYSQL_USERNAME=g2
    export MYSQL_PASSWORD=g2
    export MYSQL_NETWORK=dockercomposemysqldemo_backend
    export MYSQL_STORAGE=/storage/docker/senzing/docker-compose-mysql-demo
    ```

### Launch docker formation

```console
cd ${GIT_REPOSITORY_DIR}
docker-compose up
```

Once docker formation is up, phpMyAdmin will be available at
[localhost:8080](http://localhost:8080).

The database storage will be on the local system at ${MYSQL_STORAGE}.
The default database storage path is `/storage/docker/senzing/docker-compose-mysql-demo`.

### Add Senzing schemas

In a separate terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command.

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      senzing/mysql \
        --user=${MYSQL_USERNAME} \
        --password=${MYSQL_PASSWORD} \
        --host=${MYSQL_HOST} \
        --database=${MYSQL_DATABASE} \
        --execute="source /opt/senzing/g2/data/g2core-schema-mysql-create.sql"
    ```

After the schema is loaded, the demonstration python/Flask app will be available at
[localhost:5000](http://localhost:5000).

### Add content

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/${MYSQL_DATABASE}" \
      senzing/g2loader \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

### Run G2Command.py

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/${MYSQL_DATABASE}" \
      senzing/g2command
    ```

## Cleanup

```console
cd ${GIT_REPOSITORY_DIR}
docker-compose down

sudo rm -rf /storage/docker/senzing/docker-compose-mysql-demo
```
