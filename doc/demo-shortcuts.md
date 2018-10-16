# docker-compose-mysql-demo demo shortcuts

1. Install all of the Senzing docker images:

    ```console
    docker build --tag senzing/mysql       https://github.com/senzing/docker-mysql.git
    docker build --tag senzing/python-base https://github.com/senzing/docker-python-base.git
    docker build --tag senzing/python-demo https://github.com/senzing/docker-python-demo.git
    docker build --tag senzing/g2loader    https://github.com/senzing/docker-g2loader.git
    docker build --tag senzing/g2command   https://github.com/senzing/docker-g2command.git
    ```

1. Set environment variables

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

1. Launch docker formation

    ```console
    docker-compose up
    ```

1. Add Senzing schema. In a separate terminal window, set environment variables then run:

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

1. Run G2Loader.

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/?schema=${MYSQL_DATABASE}" \
      senzing/g2loader \
        --purgeFirst \
        --projectFile /opt/senzing/g2/python/demo/sample/project.csv
    ```

1. Run G2Command.

    ```console
    docker run -it  \
      --volume ${SENZING_DIR}:/opt/senzing \
      --net ${MYSQL_NETWORK} \
      --env SENZING_DATABASE_URL="mysql://${MYSQL_USERNAME}:${MYSQL_PASSWORD}@${MYSQL_HOST}:3306/?schema=${MYSQL_DATABASE}" \
      senzing/g2command
    ```
