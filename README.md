# docker-compose-mysql-demo

## Overview

This docker formation brings up the following docker containers:

1. *mysql*
1. *phpmyadmin/phpmyadmin*
1. *[senzing/mysql-init](https://github.com/Senzing/docker-mysql-init)*
1. *[senzing/python-demo](https://github.com/Senzing/docker-python-demo)*

Also shown in the demonstration are commands to run the following Docker images:

1. *[senzing/g2loader](https://github.com/Senzing/docker-g2loader)* in [Add content](#add-content)
1. *[senzing/g2command](https://github.com/Senzing/docker-g2command)* in [Run G2Command.py](#run-g2commandpy)

### Contents

1. [Preparation](#preparation)
    1. [Set environment variables](#set-environment-variables)
    1. [Clone repository](#clone-repository)
    1. [Software](#software)
    1. [Docker images](#docker-images)
1. [Run Docker formation with external volume](#run-docker-formation-with-external-volume)
    1. [Create SENZING_DIR](#create-senzing_dir)
    1. [Set environment variables](#set-environment-variables)
    1. [Launch docker formation](#launch-docker-formation)
    1. [Add content](#add-content)
    1. [Run G2Command.py](#run-g2commandpy)
1. [Run Docker formation without external volume](#run-docker-formation-without-external-volume)
    1. [Set environment variables for docker without external volume](#set-environment-variables-for-docker-without-external-volume)
    1. [Launch docker formation without external volume](#launch-docker-formation-without-external-volume)
    1. [Add content without external volume](#add-content-without-external-volume)
    1. [Run G2Command.py without external volume](#run-g2commandpy-without-external-volume)
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
sudo docker --version
sudo docker run hello-world
```

#### docker-compose

```console
sudo docker-compose --version
```

### Docker images

The following docker images need to be installed.

1. Creating `senzing/python-base` docker image.

   There are two options for building `senzing/python-base`.

    1. **Option #1** mounts `/opt/senzing` as an external volume.

       ```console
       sudo docker build --tag senzing/python-base https://github.com/senzing/docker-python-base.git
       ```

    1. **Option #2** includes `/opt/senzing` within the docker container's Union File System.
       Follow build instructions at
       [Senzing/docker-python-base-complete](https://github.com/Senzing/docker-python-base-complete#build)

1. Create remaining docker images.

```console
sudo docker build --tag senzing/mysql       https://github.com/senzing/docker-mysql.git
sudo docker build --tag senzing/mysql-init  https://github.com/senzing/docker-mysql-init.git
sudo docker build --tag senzing/python-demo https://github.com/senzing/docker-python-demo.git
sudo docker build --tag senzing/g2loader    https://github.com/senzing/docker-g2loader.git
sudo docker build --tag senzing/g2command   https://github.com/senzing/docker-g2command.git
```

1. Based on your choice of "with" or "without" external volume, proceed to next section to "Run Docker formation".

    1. **Option #1:** [Run Docker formation with external volume](#run-docker-formation-with-external-volume)
    1. **Option #2:** [Run Docker formation without external volume](#run-docker-formation-without-external-volume)

## Run Docker formation with external volume

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
sudo docker-compose up
```

Once docker formation is up, phpMyAdmin will be available at
[localhost:8080](http://localhost:8080).

The database storage will be on the local system at ${MYSQL_STORAGE}.
The default database storage path is `/storage/docker/senzing/docker-compose-mysql-demo`.

After the schema is loaded, the demonstration python/Flask app will be available at
[localhost:5000](http://localhost:5000).

### Add content

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    sudo docker run -it  \
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
    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/${MYSQL_DATABASE}" \
      senzing/g2command
    ```

## Run Docker formation without external volume

### Set environment variables for docker without external volume

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
    export MYSQL_HOST=senzing-mysql
    export MYSQL_DATABASE=G2
    export MYSQL_ROOT_PASSWORD=root
    export MYSQL_USERNAME=g2
    export MYSQL_PASSWORD=g2
    export MYSQL_NETWORK=dockercomposemysqldemo_backend
    export MYSQL_STORAGE=/storage/docker/senzing/docker-compose-mysql-demo
    ```

### Launch docker formation without external volume

```console
cd ${GIT_REPOSITORY_DIR}
sudo docker-compose \
    --file docker-compose-complete.yaml \
    up
```

Once docker formation is up, phpMyAdmin will be available at
[localhost:8080](http://localhost:8080).

The database storage will be on the local system at ${MYSQL_STORAGE}.
The default database storage path is `/storage/docker/senzing/docker-compose-mysql-demo`.

After the schema is loaded, the demonstration python/Flask app will be available at
[localhost:5000](http://localhost:5000).

### Add Senzing schemas

In a separate terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Set `MYSQL_DIR` and `MYSQL_FILE`.
   See [github.com/Senzing/docker-mysql](https://github.com/Senzing/docker-mysql#run-docker-container)

    ```console
    export MYSQL_DIR=/opt/senzing/g2/data
    export MYSQL_FILE=g2core-schema-mysql-create.sql
    ```

1. Run `docker` command.

    ```console
    sudo docker run -it  \
      --volume ${MYSQL_DIR}:/sql \
      --net ${MYSQL_NETWORK} \
      senzing/mysql \
        --user=${MYSQL_USERNAME} \
        --password=${MYSQL_PASSWORD} \
        --host=${MYSQL_HOST} \
        --database=${MYSQL_DATABASE} \
        --execute="source /sql/${MYSQL_FILE}"
    ```

### Add content without external volume

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    sudo docker run -it  \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/${MYSQL_DATABASE}" \
      senzing/g2loader \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

### Run G2Command.py without external volume

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker` command

    ```console
    sudo docker run -it  \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/${MYSQL_DATABASE}" \
      senzing/g2command
    ```

## Cleanup

In a separate (or reusable) terminal window:

1. [Set environment variables for docker](#set-environment-variables-for-docker)
1. Run `docker-compose` command.

    ```console
    cd ${GIT_REPOSITORY_DIR}
    sudo docker-compose down
    ```

1. Delete database storage.

    ```console
    sudo rm -rf ${MYSQL_STORAGE}
    ```

1. Delete SENZING_DIR.

    ```console
    sudo rm -rf ${SENZING_DIR}
    ```

1. Delete git repository.

    ```console
    sudo rm -rf ${GIT_REPOSITORY_DIR}
    ```
