# Headline

## Setup

1. clone repo

    ```bash
    git clone $REPO $FOLDER
    ```

2. download and build images

    ```bash
    cd $FOLDER
    docker-compose pull && docker-compose build
    ```

3. start containers

    create data dir and change owner

    ```bash
    mkdir -p data/grafana; sudo chown 472:472 data/grafana
    ```

    start containers

    ```bash
    docker-compose up -d
    ```

4. setup retention policies

    1. connect to InfluxDB shell

        ```bash
        docker-compose exec InfluxDB bash
        ```

    2. connect to InfluxDB shell

        ```bash
        influx
        ```

    3. issue InfluxDB commands

        ```sql
        USE syslog
        SHOW RETENTION POLICIES

        ALTER RETENTION POLICY "autogen" ON "syslog" DURATION 24h REPLICATION 1 DEFAULT
        ALTER RETENTION POLICY "autogen" ON "syslog" DURATION 168h REPLICATION 1 SHARD DURATION 24h DEFAULT

        CREATE RETENTION POLICY "two_weeks" ON "syslog" DURATION 2w REPLICATION 1
        CREATE RETENTION POLICY "a_year" ON "syslog" DURATION 52w REPLICATION 1


        SHOW CONTINUOUS QUERIES

        CREATE CONTINUOUS QUERY "cq_p95_5m"  ON "syslog" BEGIN SELECT percentile("duration", 95) AS "duration" INTO "two_weeks"."logstash" FROM "logstash" GROUP BY time(5m),  "engine" END
        DROP CONTINUOUS QUERY "cq_p95_5m" ON "syslog"


        CREATE CONTINUOUS QUERY "cq_p95_30m" ON "syslog" BEGIN SELECT percentile("duration", 95) AS "duration" INTO "a_year"."logstash" FROM "syslog"."two_weeks"."logstash" GROUP BY time(30m), "engine" END
        DROP CONTINUOUS QUERY "cq_p95_30m" ON "syslog"

        ```

    4. (optional)
    remove datapoints inserted before setting up the retention policies

        ```sql
        use syslog

        show measurements

        delete FROM "logstash" WHERE time <= '2019-07-03T12:00:00.000Z'

        ```

## Usage

Access the Dasboard: http://localhost:3000

* Login: admin
* Password: admin
