# Etcd - distributed key-value store

### Prerequisites

- Docker Engine 20.10+

### Usage

#### Local Access

```shell
 $ docker compose pull
    [+] Running 3/3
     ⠿ etcd-3 Pulled                                                       3.9s
     ⠿ etcd-1 Pulled                                                       3.9s
     ⠿ etcd-2 Pulled                                                       3.9s

$ docker compose up -d
    [+] Running 7/7
     ⠿ Network etcd-cluster_default     Created                            0.1s
     ⠿ Volume "etcd-cluster_etcd3"      Created                            0.0s
     ⠿ Volume "etcd-cluster_etcd1"      Created                            0.0s
     ⠿ Volume "etcd-cluster_etcd2"      Created                            0.0s
     ⠿ Container etcd-cluster-etcd-2-1  Started                            0.9s
     ⠿ Container etcd-cluster-etcd-3-1  Started                            1.1s
     ⠿ Container etcd-cluster-etcd-1-1  Started                            1.1s

$ docker-compose up -d
Creating network "etcd-cluster_default" with the default driver
Creating etcd-cluster_etcd-3_1 ... done
Creating etcd-cluster_etcd-2_1 ... done
Creating etcd-cluster_etcd-1_1 ... done
$ docker-compose ps
        Name                       Command               State                 Ports
--------------------------------------------------------------------------------------------------
etcd-cluster_etcd-1_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32773->2379/tcp, 2380/tcp
etcd-cluster_etcd-2_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32771->2379/tcp, 2380/tcp
etcd-cluster_etcd-3_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32772->2379/tcp, 2380/tcp
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl --write-out=table endpoint status
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | b8c6addf901e4e46 |   3.5.7 |   20 kB |     false |      false |         2 |          8 |                  8 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl --write-out=table endpoint health
+----------------+--------+------------+-------+
|    ENDPOINT    | HEALTH |    TOOK    | ERROR |
+----------------+--------+------------+-------+
| 127.0.0.1:2379 |   true | 2.010323ms |       |
+----------------+--------+------------+-------+
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl --write-out=table member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| 88d11e2649dad027 | started | etcd-2 | http://etcd-2:2380 | http://etcd-2:2379 |      false |
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
| c3697a4fd7a20dcd | started | etcd-3 | http://etcd-3:2380 | http://etcd-3:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl put secret password
OK
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl get secret
secret
password
$ docker exec -it etcd-cluster_etcd-2_1 etcdctl get secret
secret
password
$ docker exec -it etcd-cluster_etcd-3_1 etcdctl get secret
secret
password
```

#### Remote Access

```shell
gongpengjun@nuc:docker-compose-etcd$ docker-compose up -d
Creating network "etcd-cluster_default" with the default driver
Creating etcd-cluster_etcd-2_1 ... done
Creating etcd-cluster_etcd-3_1 ... done
Creating etcd-cluster_etcd-1_1 ... done
gongpengjun@nuc:docker-compose-etcd$ docker-compose ps
        Name                       Command               State                        Ports
-----------------------------------------------------------------------------------------------------------------
etcd-cluster_etcd-1_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:12379->2379/tcp, 0.0.0.0:12380->2380/tcp
etcd-cluster_etcd-2_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:22379->2379/tcp, 0.0.0.0:22380->2380/tcp
etcd-cluster_etcd-3_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32379->2379/tcp, 0.0.0.0:32380->2380/tcp
gongpengjun@nuc:docker-compose-etcd$ ifconfig | grep "10.0.0." | sed -En 's/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'
10.0.0.23


gongpengjun@mbp ~$ ETCDCTL_API=3 etcdctl --write-out=table --endpoints=http://10.0.0.23:12379 member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| 88d11e2649dad027 | started | etcd-2 | http://etcd-2:2380 | http://etcd-2:2379 |      false |
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
| c3697a4fd7a20dcd | started | etcd-3 | http://etcd-3:2380 | http://etcd-3:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
gongpengjun@mbp ~$ ETCDCTL_API=3 etcdctl --write-out=table --endpoints=http://10.0.0.23:22379 member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| 88d11e2649dad027 | started | etcd-2 | http://etcd-2:2380 | http://etcd-2:2379 |      false |
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
| c3697a4fd7a20dcd | started | etcd-3 | http://etcd-3:2380 | http://etcd-3:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
gongpengjun@mbp ~$ ETCDCTL_API=3 etcdctl --write-out=table --endpoints=http://10.0.0.23:32379 member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| 88d11e2649dad027 | started | etcd-2 | http://etcd-2:2380 | http://etcd-2:2379 |      false |
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
| c3697a4fd7a20dcd | started | etcd-3 | http://etcd-3:2380 | http://etcd-3:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
gongpengjun@mbp ~$ etcdctl --endpoints=http://10.0.0.23:12379 put secret password
OK
gongpengjun@mbp ~$ etcdctl --endpoints=http://10.0.0.23:12379 get secret
secret
password
gongpengjun@mbp ~$ etcdctl --endpoints=http://10.0.0.23:22379 get secret
secret
password
gongpengjun@mbp ~$ etcdctl --endpoints=http://10.0.0.23:32379 get secret
secret
password
```

### Docker Images

- [etcd][docker-image-etcd]

[docker-image-etcd]: https://quay.io/repository/coreos/etcd?tab=tags
