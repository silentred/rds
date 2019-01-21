# CoreDNS plugin for using RDS as record backend

NOTE: Code is basically copied from https://github.com/wenerme/wps. I removed some irrelevant code and add some to have a purer and useful version for myself.

The plugin name is still `pdsql` in the code. It returns NXDOMAIN code if wild match returns no records.

## Syntax

``` txt
pdsql <dialect> <arg> {
    # enable debug mode
    debug [db]
    # create table for test
    [auto-migrate]
}
```

## Install Driver
pdsql need db driver for dialect, to install a driver you need to add import in plugin.cfg, like

``` txt
pdsql_mysql:github.com/jinzhu/gorm/dialects/mysql
pdsql_sqlite:github.com/jinzhu/gorm/dialects/sqlite
```

pdsql_mysql and pdsql_sqlite are meaningless, choose to prevent duplicated.

## Examples

Start a server on the 53 port, use test.db as backend.

``` corefile
.:53 {
    pdsql sqlite3 /tmp/test.db {
        debug db
        auto-migrate
    }
    fallback original NXDOMAIN . 8.8.8.8:53
    fallback original SERVFAIL . 8.8.8.8:53
    log
}
```

Prepare data for test.

``` bash
# Insert records for wener.test
sqlite3 ./test.db 'insert into records(name,type,content,ttl,disabled)values("wener.test","A","192.168.1.1",3600,0)'
sqlite3 ./test.db 'insert into records(name,type,content,ttl,disabled)values("wener.test","TXT","TXT Here",3600,0)'
```

When queried for "wener.test. A", CoreDNS will respond with:

``` txt
;; QUESTION SECTION:
;wener.test.			IN	A

;; ANSWER SECTION:
wener.test.		3600	IN	A	192.168.1.1
```

When queried for "wener.test. ANY", CoreDNS will respond with:

``` txt
;; QUESTION SECTION:
;wener.test.			IN	ANY

;; ANSWER SECTION:
wener.test.		3600	IN	A	192.168.1.1
wener.test.		3600	IN	TXT	"TXT Here"
```

### Wildcard

``` bash
# domain id 1
sqlite3 ./test.db 'insert into domains(name,type)values("example.test","NATIVE")'
sqlite3 ./test.db 'insert into records(domain_id,name,type,content,ttl,disabled)values(1,"*.example.test","A","192.168.1.1",3600,0)'
```

When queried for "first.example.test. A", CoreDNS will respond with:

``` txt
;; QUESTION SECTION:
;first.example.test.		IN	A

;; ANSWER SECTION:
first.example.test.	3600	IN	A	192.168.1.1
```
