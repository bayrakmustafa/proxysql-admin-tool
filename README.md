ProxySQL Admin
==============

The ProxySQL Admin (proxysql-admin) solution configures Percona XtraDB cluster nodes into ProxySQL. 

https://github.com/percona/proxysql-admin-tool

fault tolerans for write hostgroup,
first node is write node.and all nodes are read node.
when first node is down , second node is write node 
when second node is down , third node is write/read node ,
if hostgroup is down , all related rules are made as passive
and 
if hostgroup is open , all related rules are made as active
 

mysql> 
```

delete from  mysql_servers;
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (121,'10.145.172.21',3306,10000,'WRITE');
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (122,'10.145.172.22',3306,10000,'WRITE');
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (123,'10.145.172.23',3306,10000,'WRITE');
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (101,'10.145.172.21',3306,10000,'READ');
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (101,'10.145.172.22',3306,10000,'READ');
 INSERT INTO mysql_servers(hostgroup_id, hostname, port,max_connections,comment) VALUES (101,'10.145.172.23',3306,10000,'READ');

delete from mysql_users;
INSERT INTO mysql_users(username,password,default_hostgroup) VALUES ('test','testPass9$',101);

delete from mysql_query_rules;
INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply) VALUES
(1,'^select',101,0),

INSERT INTO mysql_query_rules(active,match_pattern,destination_hostgroup,apply)
VALUES
(1,'^INSERT',121,1),
(1,'^INSERT',122,1),
(1,'^INSERT',123,1);

delete from scheduler;
INSERT INTO scheduler(id,active,interval_ms,filename,arg1,arg2,arg3,arg4,arg5)
VALUES (1,'1','10000','/usr/bin/proxysql_galera_checker','121','101','0','1','/var/lib/proxysql/proxysql_galera_checker.log');

LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK; 
LOAD SCHEDULER TO RUNTIME;
SAVE SCHEDULER TO DISK;

```


Please log ProxySQL Admin bug reports here https://jira.percona.com/projects/PSQLADM.

proxysql-admin usage info

```bash
Usage: [ options ]
Options:
  --config-file                      Read login credentials from a configuration file (overrides any login credentials specified on the command line)
  --quick-demo                       Setup a quick demo with no authentication
  --proxysql-datadir=<datadir>       Specify proxysql data directory location
  --proxysql-username=user_name      Username for connecting to the ProxySQL service
  --proxysql-password[=password]     Password for connecting to the ProxySQL service
  --proxysql-port=port_num           Port Nr. for connecting to the ProxySQL service
  --proxysql-hostname=host_name      Hostname for connecting to the ProxySQL service
  --cluster-username=user_name       Username for connecting to the Percona XtraDB Cluster node
  --cluster-password[=password]      Password for connecting to the Percona XtraDB Cluster node
  --cluster-port=port_num            Port Nr. for connecting to the Percona XtraDB Cluster node
  --cluster-hostname=host_name       Hostname for connecting to the Percona XtraDB Cluster node
  --cluster-app-username=user_name   Application username for connecting to the Percona XtraDB Cluster node
  --cluster-app-password[=password]  Application password for connecting to the Percona XtraDB Cluster node
  --without-cluster-app-user         Configure Percona XtraDB Cluster without application user
  --monitor-username=user_name       Username for monitoring Percona XtraDB Cluster nodes through ProxySQL
  --monitor-password[=password]      Password for monitoring Percona XtraDB Cluster nodes through ProxySQL
  --enable, -e                       Auto-configure Percona XtraDB Cluster nodes into ProxySQL
  --disable, -d                      Remove any Percona XtraDB Cluster configurations from ProxySQL
  --node-check-interval=3000         Interval for monitoring node checker script (in milliseconds)
  --mode=[loadbal|singlewrite]       ProxySQL read/write configuration mode, currently supporting: 'loadbal' and 'singlewrite' (the default) modes
  --write-node=host_name:port        Writer node to accept write statments. This option is supported only when using --mode=singlewrite
                                     Can accept comma delimited list with the first listed being the highest priority.
  --include-slaves=host_name:port    Add specified slave node(s) to ProxySQL, these nodes will go into the reader hostgroup and will only be put into
                                     the writer hostgroup if all cluster nodes are down.  Slaves must be read only.  Can accept comma delimited list.
                                     If this is used make sure 'read_only=1' is in the slave's my.cnf
  --adduser                          Adds the Percona XtraDB Cluster application user to the ProxySQL database
  --syncusers                        Sync user accounts currently configured in MySQL to ProxySQL (deletes ProxySQL users not in MySQL)
  --version, -v                      Print version info
```
Pre-requisites 
--------------
* ProxySQL and Cluster should be up and running.
* For security purposes, please ensure to change the default user settings in the ProxySQL configuration file.
* _ProxySQL configuration file(/etc/proxysql-admin.cnf)_
```bash
  # proxysql admin interface credentials.
  export PROXYSQL_DATADIR='/var/lib/proxysql'
  export PROXYSQL_USERNAME='admin'
  export PROXYSQL_PASSWORD='admin'
  export PROXYSQL_HOSTNAME='localhost'
  export PROXYSQL_PORT='6032'

  # PXC admin credentials for connecting to pxc-cluster-node.
  export CLUSTER_USERNAME='admin'
  export CLUSTER_PASSWORD='admin'
  export CLUSTER_HOSTNAME='localhost'
  export CLUSTER_PORT='3306'

  # proxysql monitoring user. proxysql admin script will create this user in pxc to monitor pxc-nodes.
  export MONITOR_USERNAME='monitor'
  export MONITOR_PASSWORD='monit0r'

  # Application user to connect to pxc-node through proxysql
  export CLUSTER_APP_USERNAME='proxysql_user'
  export CLUSTER_APP_PASSWORD='passw0rd'

  # ProxySQL read/write hostgroup 
  export WRITE_HOSTGROUP_ID='10'
  export READ_HOSTGROUP_ID='11'

  # ProxySQL read/write configuration mode.
  export MODE="singlewrite"

  # ProxySQL Cluster Node Priority File
  export HOST_PRIORITY_FILE=$PROXYSQL_DATADIR/host_priority.conf

 #When 1, update apply status of rules by state of servers in hostgroup
 #        minumum one server of all hosts state is not  ONLINE ,change apply state of rules so
 #        rules is not active
 export PROXYSQL_UPDATE_RULE="1"

```
