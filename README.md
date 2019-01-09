# Falcon+

![Open-Falcon](./logo.png)

[![Build Status](https://travis-ci.org/open-falcon/falcon-plus.svg?branch=plus-dev)](https://travis-ci.org/open-falcon/falcon-plus)
[![codecov](https://codecov.io/gh/open-falcon/falcon-plus/branch/plus-dev/graph/badge.svg)](https://codecov.io/gh/open-falcon/falcon-plus)
[![GoDoc](https://godoc.org/github.com/open-falcon/falcon-plus?status.svg)](https://godoc.org/github.com/open-falcon/falcon-plus)
[![Code Issues](https://www.quantifiedcode.com/api/v1/project/5035c017b02c4a4a807ebc4e9f153e6f/badge.svg)](https://www.quantifiedcode.com/app/project/5035c017b02c4a4a807ebc4e9f153e6f)
[![Go Report Card](https://goreportcard.com/badge/github.com/open-falcon/falcon-plus)](https://goreportcard.com/report/github.com/open-falcon/falcon-plus)
[![License](https://img.shields.io/badge/LICENSE-Apache2.0-ff69b4.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)
[![Backers on Open Collective](https://opencollective.com/falcon-plus/backers/badge.svg)](#backers) 
[![Sponsors on Open Collective](https://opencollective.com/falcon-plus/sponsors/badge.svg)](#sponsors) 

# Documentations

- [Usage](http://book.open-falcon.org)
- [Open-Falcon API](http://open-falcon.org/falcon-plus)

# Prerequisite

- Git >= 1.7.5
- Go >= 1.6

# Getting Started

## Docker

Please refer to ./docker/[README.md](https://github.com/open-falcon/falcon-plus/blob/master/docker/README.md).

## Build from source
**before start, please make sure you prepared this:**

```
yum install -y redis
yum install -y mysql-server

```

*NOTE: be sure to check redis and mysql-server have successfully started.*

And then

```
# Please make sure that you have set `$GOPATH` and `$GOROOT` correctly.
# If you have not golang in your host, please follow [https://golang.org/doc/install] to install golang.

mkdir -p $GOPATH/src/github.com/open-falcon
cd $GOPATH/src/github.com/open-falcon
git clone https://github.com/open-falcon/falcon-plus.git

```

**And do not forget to init the database first (if you have not loaded the database schema before)**

```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/scripts/mysql/db_schema/
mysql -h 127.0.0.1 -u root -p < 1_uic-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 2_portal-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 3_dashboard-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 4_graph-db-schema.sql
mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql
```

**NOTE: if you are upgrading from v0.1 to current version v0.2.0,then**. [More upgrading instruction](http://www.jianshu.com/p/6fb2c2b4d030)

    mysql -h 127.0.0.1 -u root -p < 5_alarms-db-schema.sql

# Compilation

```
cd $GOPATH/src/github.com/open-falcon/falcon-plus/

# make all modules
make all

# make specified module
make agent

# pack all modules
make pack
```

* *after `make pack` you will got `open-falcon-vx.x.x.tar.gz`*
* *if you want to edit configure file for each module, you can edit `config/xxx.json` before you do `make pack`*

#  Unpack and Decompose

```
export WorkDir="$HOME/open-falcon"
mkdir -p $WorkDir
tar -xzvf open-falcon-vx.x.x.tar.gz -C $WorkDir
cd $WorkDir
```

# Start all modules in single host
```
cd $WorkDir
./open-falcon start

# check modules status
./open-falcon check

```

# Run More Open-Falcon Commands

for example:

```
# ./open-falcon [start|stop|restart|check|monitor|reload] module
./open-falcon start agent

./open-falcon check
        falcon-graph         UP           53007
          falcon-hbs         UP           53014
        falcon-judge         UP           53020
     falcon-transfer         UP           53026
       falcon-nodata         UP           53032
   falcon-aggregator         UP           53038
        falcon-agent         UP           53044
      falcon-gateway         UP           53050
          falcon-api         UP           53056
        falcon-alarm         UP           53063
```

* For debugging , You can check `$WorkDir/$moduleName/logs/xxx.log`

# Install Frontend Dashboard
- Follow [this](https://github.com/open-falcon/dashboard).

**NOTE: if you want to use grafana as the dashboard, please check [this](https://github.com/open-falcon/grafana-openfalcon-datasource).**

# Package Management

We use govendor to manage the golang packages. Please install `govendor` before compilation.

    go get -u github.com/kardianos/govendor

Most depended packages are saved under `./vendor` dir. If you want to add or update a package, just run `govendor fetch xxxx@commitID` or `govendor fetch xxxx@v1.x.x`, then you will find the package have been placed in `./vendor` correctly.

# Package Release

```
make clean all pack
```

# API Standard
- [API Standard](https://github.com/open-falcon/falcon-plus/blob/master/api-standard.md)


# Q&A

- Any issue or question is welcome, Please feel free to open [github issues](https://github.com/open-falcon/falcon-plus/issues) :)
- [FAQ](http://book.open-falcon.org/zh_0_2/faq/)


## Contributors

This project exists thanks to all the people who contribute. [[Contribute](CONTRIBUTING.md)].
<a href="https://github.com/open-falcon/falcon-plus/contributors"><img src="https://opencollective.com/falcon-plus/contributors.svg?width=890&button=false" /></a>


########### start mysql.
docker run -itd \
    --name falcon-mysql \
    -e MYSQL_ROOT_PASSWORD=test123456 \
    -p 3306:3306 \
    mysql:5.7

## init db.
for x in `ls ./scripts/mysql/db_schema/*.sql`; do
    echo init mysql table $x ...;
    docker exec -i falcon-mysql mysql -uroot -ptest123456 < $x;
done

## start redis.
docker run --name falcon-redis -p6379:6379 -d redis:4-alpine3.8

## start falcon .
docker run -itd --name falcon-plus \
     --link=falcon-mysql:db.falcon \
     --link=falcon-redis:redis.falcon \
     -p 8433:8433 \
     -p 8080:8080 \
     -p 6030:6030 \
     -e MYSQL_PORT=root:test123456@tcp\(db.falcon:3306\) \
     -e REDIS_PORT=redis.falcon:6379  \
     tern:v0.0.2.1

## start falcon backend modules, such as graph,api,etc.
    docker exec falcon-plus sh ctrl.sh start \
            graph hbs judge transfer nodata aggregator agent gateway api alarm

    ## or you can just start/stop/restart specific module as: 
    docker exec falcon-plus sh ctrl.sh start/stop/restart xxx
    docker exec falcon-plus sh ctrl.sh start transfer


    ## check status of backend modules
    docker exec falcon-plus ./open-falcon check

    ## or you can check logs at /home/work/open-falcon/logs/ in your host
    ls -l /home/work/open-falcon/logs/

docker run -itd --name falcon-dashboard \
        -p 8081:8081 \
        --link=falcon-mysql:db.falcon \
        --link=falcon-plus:api.falcon \
        -e API_ADDR=http://api.falcon:8080/api/v1 \
        -e PORTAL_DB_HOST=db.falcon \
        -e PORTAL_DB_PORT=3306 \
        -e PORTAL_DB_USER=root \
        -e PORTAL_DB_PASS=root\
        -e PORTAL_DB_NAME=falcon_portal \
        -e ALARM_DB_HOST=db.falcon \
        -e ALARM_DB_PORT=3306 \
        -e ALARM_DB_USER=root \
        -e ALARM_DB_PASS=root\
        -e ALARM_DB_NAME=alarms \
        -w /open-falcon/dashboard openfalcon/falcon-dashboard:v0.2.1  \
       './control startfg'