go-mysql-syncer is a service syncing your MySQL data into mysql client automatically.

It uses `mysqldump` to fetch the origin data at first, then syncs data incrementally with binlog.

## Install

+ Install Go (1.6+) and set your [GOPATH](https://golang.org/doc/code.html#GOPATH)
+ `go get github.com/YuanDdQiao/go-mysql-syncer`, it will print some messages in console, skip it. :-)
+ cd `$GOPATH/src/github.com/YuanDdQiao/go-mysql-syncer`
+ `make`

## How to use?

+ Create table in MySQL.
+ Config base, see the example config [river.toml](./etc/river.toml).
+ Set MySQL source in config file, see [Source](#source) below.
+ Customize MySQL and client mapping rule in config file, see [Rule](#rule) below.
+ Start `./bin/go-mysql-syncer -config=./etc/river.toml` and enjoy it.

## Notice

+ binlog format must be **row**.
+ binlog row image must be **full** for MySQL, you may lost some field data if you update PK data in MySQL with minimal or noblob binlog row image. MariaDB only supports full row image.
+ Can not alter table format at runtime.
+ `mysqldump` must exist in the same node with go-mysql-syncer, if not, go-mysql-syncer will try to sync binlog only.
+ Don't change too many rows at same time in one SQL.

## Source

In go-mysql-syncer, you must decide which tables you want to sync into client in the source config.

The format in config file is below:
# MySQL address, user and password
# user must have replication privilege in MySQL.
# xxx平台
my_addr = "xxx:3306"

my_user = "xxx"

my_pass = "xxx"

my_charset = "xxx"

# Set true when slave_mysql 
slave_addr = "xxx:3307"
# slave_mysql user and password, maybe set by shield, nginx, or x-pack
slave_user = "xxx"

slave_pass = "xxx"

# Path to store data, like master.info, if not set or empty,
# we must use this to support breakpoint resume syncing. 
# TODO: support other storage, like etcd. 
data_dir = "./var"

# Inner Http status address
stat_addr = "127.0.0.1:12800"

# pseudo server id like a slave 
server_id = 1001

# mysql or mariadb
flavor = "mysql"

# mysqldump execution path
# if not set or empty, ignore mysqldump.
mysqldump = "mysqldump"

# if we have no privilege to use mysqldump with --master-data,
# we must skip it.
#skip_master_data = false
skip_master_data = true

# minimal items to be inserted in one bulk
bulk_size = 128

# force flush the pending requests if we don't have enough items >= bulk_size
flush_bulk_time = "200ms"

# Ignore table without primary key
skip_no_pk_table = false

# MySQL data source
[[source]]

schema = "dba_demotohbase"

# Only below tables will be synced into slave_mysql.
# I don't think it is necessary to sync all tables in a database.
#tables = ["cre_request_record","cre_response_detail"]
tables = ["cre_request_record"]

# Below is for special rule mapping

# Very simple example
# 
# desc t;
# +-------+--------------+------+-----+---------+-------+
# | Field | Type         | Null | Key | Default | Extra |
# +-------+--------------+------+-----+---------+-------+
# | id    | int(11)      | NO   | PRI | NULL    |       |
# | name  | varchar(256) | YES  |     | NULL    |       |
# +-------+--------------+------+-----+---------+-------+
# 
[[rule]]

schema = "dba_demotohbase"

table = "cre_request_record"

slaveschema = "test"

slavetable = "cre_request_record"


#[[rule]]
#schema = "dba_demotohbase"
#table = "cre_response_detail"
#slaveschema = "test"
#slavetable = "cre_response_detail"
