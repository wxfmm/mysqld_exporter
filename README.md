# MySQL Server Exporter [![Build Status](https://travis-ci.org/prometheus/mysqld_exporter.svg)][travis]

[![CircleCI](https://circleci.com/gh/prometheus/mysqld_exporter/tree/master.svg?style=shield)][circleci]
[![Docker Repository on Quay](https://quay.io/repository/prometheus/mysqld-exporter/status)][quay]
[![Docker Pulls](https://img.shields.io/docker/pulls/prom/mysqld-exporter.svg?maxAge=604800)][hub]
[![Go Report Card](https://goreportcard.com/badge/github.com/prometheus/mysqld_exporter)](https://goreportcard.com/report/github.com/prometheus/mysqld_exporter)

Prometheus exporter for MySQL server metrics.

Supported MySQL & MariaDB versions: 5.5 and up.

NOTE: Not all collection methods are supported on MySQL/MariaDB < 5.6

## Building and running

### Required Grants

```sql
CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'XXXXXXXX' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';
```

##多实例汇总表为undb_master与workgroup表的关联查询的固定字段，可根据具体情况调整，主机和端口必须连通，

NOTE: It is recommended to set a max connection limit for the user to avoid overloading the server with monitoring scrapes under heavy load. This is not supported on all MySQL/MariaDB versions; for example, MariaDB 10.1 (provided with Ubuntu 18.04) [does _not_ support this feature](https://mariadb.com/kb/en/library/create-user/#resource-limit-options).

### Build

    make

### Running

1. 以读取表中多实例方式采集数据

    export DATA_SOURCE_NAME='user:password@(hostname:3306)/'
    ./mysqld_exporter

2. 以读取配置文件中多实例方式采集数据

	vim /conf/exporter.cnf

    ./mysqld_exporter
	
3. 以配置环境变量方式采集单个或多个实例数据，格式： host1,port1,username1,password1,instance_name1:host2,port2,username2,password2,instance_name2

   export SOURCE_DATA=172.20.4.26,3306,wx_alarm,wx@alarm,微信告警数据库:139.186.30.34,3306,wx_alarm,wx@alarm,微信告警数据库
   
   ./mysqld_exporter

### Collector Flags

Name                                                         | MySQL Version | Description
-------------------------------------------------------------|---------------|------------------------------------------------------------------------------------
collect.auto_increment.columns                               | 5.1           | Collect auto_increment columns and max values from information_schema.
collect.binlog_size                                          | 5.1           | Collect the current size of all registered binlog files
collect.engine_innodb_status                                 | 5.1           | Collect from SHOW ENGINE INNODB STATUS.
collect.engine_tokudb_status                                 | 5.6           | Collect from SHOW ENGINE TOKUDB STATUS.
collect.global_status                                        | 5.1           | Collect from SHOW GLOBAL STATUS (Enabled by default)
collect.global_variables                                     | 5.1           | Collect from SHOW GLOBAL VARIABLES (Enabled by default)
collect.info_schema.clientstats                              | 5.5           | If running with userstat=1, set to true to collect client statistics.
collect.info_schema.innodb_metrics                           | 5.6           | Collect metrics from information_schema.innodb_metrics.
collect.info_schema.innodb_tablespaces                       | 5.7           | Collect metrics from information_schema.innodb_sys_tablespaces.
collect.info_schema.innodb_cmp                               | 5.5           | Collect InnoDB compressed tables metrics from information_schema.innodb_cmp.
collect.info_schema.innodb_cmpmem                            | 5.5           | Collect InnoDB buffer pool compression metrics from information_schema.innodb_cmpmem.
collect.info_schema.processlist                              | 5.1           | Collect thread state counts from information_schema.processlist.
collect.info_schema.processlist.min_time                     | 5.1           | Minimum time a thread must be in each state to be counted. (default: 0)
collect.info_schema.query_response_time                      | 5.5           | Collect query response time distribution if query_response_time_stats is ON.
collect.info_schema.replica_host                             | 5.6           | Collect metrics from information_schema.replica_host_status.
collect.info_schema.tables                                   | 5.1           | Collect metrics from information_schema.tables.
collect.info_schema.tables.databases                         | 5.1           | The list of databases to collect table stats for, or '`*`' for all.
collect.info_schema.tablestats                               | 5.1           | If running with userstat=1, set to true to collect table statistics.
collect.info_schema.schemastats                              | 5.1           | If running with userstat=1, set to true to collect schema statistics
collect.info_schema.userstats                                | 5.1           | If running with userstat=1, set to true to collect user statistics.
collect.perf_schema.eventsstatements                         | 5.6           | Collect metrics from performance_schema.events_statements_summary_by_digest.
collect.perf_schema.eventsstatements.digest_text_limit       | 5.6           | Maximum length of the normalized statement text. (default: 120)
collect.perf_schema.eventsstatements.limit                   | 5.6           | Limit the number of events statements digests by response time. (default: 250)
collect.perf_schema.eventsstatements.timelimit               | 5.6           | Limit how old the 'last_seen' events statements can be, in seconds. (default: 86400)
collect.perf_schema.eventsstatementssum                      | 5.7           | Collect metrics from performance_schema.events_statements_summary_by_digest summed.
collect.perf_schema.eventswaits                              | 5.5           | Collect metrics from performance_schema.events_waits_summary_global_by_event_name.
collect.perf_schema.file_events                              | 5.6           | Collect metrics from performance_schema.file_summary_by_event_name.
collect.perf_schema.file_instances                           | 5.5           | Collect metrics from performance_schema.file_summary_by_instance.
collect.perf_schema.indexiowaits                             | 5.6           | Collect metrics from performance_schema.table_io_waits_summary_by_index_usage.
collect.perf_schema.tableiowaits                             | 5.6           | Collect metrics from performance_schema.table_io_waits_summary_by_table.
collect.perf_schema.tablelocks                               | 5.6           | Collect metrics from performance_schema.table_lock_waits_summary_by_table.
collect.perf_schema.replication_group_member_stats           | 5.7           | Collect metrics from performance_schema.replication_group_member_stats.
collect.perf_schema.replication_applier_status_by_worker     | 5.7           | Collect metrics from performance_schema.replication_applier_status_by_worker.
collect.slave_status                                         | 5.1           | Collect from SHOW SLAVE STATUS (Enabled by default)
collect.slave_hosts                                          | 5.1           | Collect from SHOW SLAVE HOSTS
collect.heartbeat                                            | 5.1           | Collect from [heartbeat](#heartbeat).
collect.heartbeat.database                                   | 5.1           | Database from where to collect heartbeat data. (default: heartbeat)
collect.heartbeat.table                                      | 5.1           | Table from where to collect heartbeat data. (default: heartbeat)


### General Flags
Name                                       | Description
-------------------------------------------|--------------------------------------------------------------------------------------------------
config.my-cnf                              | Path to .my.cnf file to read MySQL credentials from. (default: `~/.my.cnf`)
log.level                                  | Logging verbosity (default: info)
exporter.lock_wait_timeout                 | Set a lock_wait_timeout on the connection to avoid long metadata locking. (default: 2 seconds)
exporter.log_slow_filter                   | Add a log_slow_filter to avoid slow query logging of scrapes.  NOTE: Not supported by Oracle MySQL.
web.listen-address                         | Address to listen on for web interface and telemetry.
web.telemetry-path                         | Path under which to expose metrics.
version                                    | Print the version information.

## Using Docker
For example:

```bash
docker pull registry.un-net.com/triangle/triangle-undb-exporter:tags

docker run --name exporter 
-p 8080:8080 
-e DATA_SOURCE_NAME='root:123qwe@(172.23.27.108:10000)/triangle_undb' 
-e EXPOSE_PORT=':8080' 
-d registry.un-net.com/triangle/triangle-undb-exporter:v1.0.2
```
说明：如果指定了环境变量DATA_SOURCE_NAME,配置文件不生效；未指定改变量，读取/usr/local/service/mysql_exporter/exporter.cnf。
如果未指定环境变量EXPOSE_PORT,则默认以9104为启动端口



## heartbeat

With `collect.heartbeat` enabled, mysqld_exporter will scrape replication delay
measured by heartbeat mechanisms. [Pt-heartbeat][pth] is the
reference heartbeat implementation supported.

[pth]:https://www.percona.com/doc/percona-toolkit/2.2/pt-heartbeat.html


## Filtering enabled collectors

The `mysqld_exporter` will expose all metrics from enabled collectors by default. This is the recommended way to collect metrics to avoid errors when comparing metrics of different families.

For advanced use the `mysqld_exporter` can be passed an optional list of collectors to filter metrics. The `collect[]` parameter may be used multiple times.  In Prometheus configuration you can use this syntax under the [scrape config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#<scrape_config>).

```yaml
params:
  collect[]:
  - foo
  - bar
```

This can be useful for having different Prometheus servers collect specific metrics from targets.

## Example Rules

There are some sample rules available in [example.rules](example.rules)

[circleci]: https://circleci.com/gh/prometheus/mysqld_exporter
[hub]: https://hub.docker.com/r/prom/mysqld-exporter/
[travis]: https://travis-ci.org/prometheus/mysqld_exporter
[quay]: https://quay.io/repository/prometheus/mysqld-exporter
[parsetime]: https://github.com/go-sql-driver/mysql#parsetime
