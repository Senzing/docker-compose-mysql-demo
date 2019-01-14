# docker-compose-mysql-demo demo shortcuts

## Install all of the Senzing docker images

1. Build all Senzing docker images locally.

    ```console
    sudo docker build --tag senzing/mysql       https://github.com/senzing/docker-mysql.git
    sudo docker build --tag senzing/mysql-init  https://github.com/senzing/docker-mysql-init.git
    sudo docker build --tag senzing/python-base https://github.com/senzing/docker-python-base.git
    sudo docker build --tag senzing/python-demo https://github.com/senzing/docker-python-demo.git
    sudo docker build --tag senzing/g2loader    https://github.com/senzing/docker-g2loader.git
    sudo docker build --tag senzing/g2command   https://github.com/senzing/docker-g2command.git
    ```

## Set environment variables

1. Set environment variables used in docker-compose and docker images. Example:

    ```console
    export MYSQL_HOST=senzing-mysql
    export MYSQL_DATABASE=G2
    export MYSQL_ROOT_PASSWORD=root
    export MYSQL_USERNAME=g2
    export MYSQL_PASSWORD=g2
    export MYSQL_STORAGE=/storage/docker/senzing/docker-compose-mysql-demo

    export SENZING_NETWORK=dockercomposemysqldemo_backend
    export SENZING_DIR=/opt/senzing
    export SENZING_DATABASE_URL="${DATABASE_PROTOCOL}://${DATABASE_USERNAME}:${DATABASE_PASSWORD}@${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_DATABASE}"
    ```

## Launch docker formation

1. Bring up docker formation.

    ```console
    sudo docker-compose up
    ```

## Run G2Loader

1. Run G2Loader.py in docker container.

    ```console
    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${SENZING_NETWORK} \
      --env SENZING_DATABASE_URL="${SENZING_DATABASE_URL}" \
      senzing/g2loader \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

## Run G2Command

1. Run G2Command.py in docker container.

    ```console
    sudo docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${SENZING_NETWORK} \
      --env SENZING_DATABASE_URL="${SENZING_DATABASE_URL}" \
      senzing/g2command
    ```
