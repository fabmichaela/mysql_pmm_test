# docker-compose
docker compose up -d

# replication setting
## one(master): create replication user
```
mysql -uroot -ptest -h127.0.0.1 -P3301
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
exit
```

## two(slave): change master and start slave
```
mysql -uroot -ptest -h127.0.0.1 -P3302
change master to master_host='one',master_port=3306, master_user='repl', \
  master_password='repl', master_auto_position=1;
start slave;
show slave status \G
```


# monitoring using pmm
## one(master): create pmm user
```
mysql -uroot -ptest -h127.0.0.1 -P3301
CREATE USER 'pmm'@'%' IDENTIFIED BY 'pmm' WITH MAX_USER_CONNECTIONS 10;
GRANT SELECT, PROCESS, REPLICATION CLIENT, RELOAD, BACKUP_ADMIN ON *.* TO 'pmm'@'%';
```

## pmm-admin add
```
docker exec mysql-replication_pmm-client_1 \
pmm-admin add mysql --cluster=my80replication --replication-set=my80replication --username=pmm --password=pmm --query-source=perfschema  --service-name=one --host=one --port=3306

docker exec mysql-replication_pmm-client_1 \
pmm-admin add mysql --cluster=my80replication --replication-set=my80replication --username=pmm --password=pmm --query-source=perfschema --service-name=two --host=two --port=3306
```

## comment out PMM_AGENT_SETUP=1 on docker-compose.yaml
```
      # - PMM_AGENT_SETUP=1 ## used at first setup only
```