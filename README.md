# Run etcd cluster by docker-compose

### Usage

#### Local Access

```shell
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

gongpengjun@mbp ~$ curl -s http://10.0.0.23:12379/version | jq
{
  "etcdserver": "3.5.7",
  "etcdcluster": "3.5.0"
}
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

### Persist Data

```shell
# 初始data目录为空
gongpengjun@mbp docker-compose-etcd$ tree
.
├── README.md
├── data
└── docker-compose.yml

1 directory, 2 files
gongpengjun@mbp a$ ls -l
total 24
-rw-r--r--  1 gongpengjun  staff  7789  2 28 17:52 README.md
drwxr-xr-x  3 gongpengjun  staff    96  2 28 17:50 data
-rw-r--r--  1 gongpengjun  staff  2819  2 28 17:49 docker-compose.yml
gongpengjun@mbp a$ ls -l data
total 0

# 启动etcd cluster
gongpengjun@mbp docker-compose-etcd$ docker-compose up -d
Creating network "etcd-cluster_bridge-network" with driver "bridge"
Creating etcd-cluster_etcd-2_1 ... done
Creating etcd-cluster_etcd-3_1 ... done
Creating etcd-cluster_etcd-1_1 ... done

# 查看data目录树，etcd已经初始化好了数据目录
gongpengjun@mbp docker-compose-etcd$ tree
.
├── README.md
├── data
│   ├── etcd1
│   │   └── member
│   │       ├── snap
│   │       │   └── db
│   │       └── wal
│   │           ├── 0.tmp
│   │           └── 0000000000000000-0000000000000000.wal
│   ├── etcd2
│   │   └── member
│   │       ├── snap
│   │       │   └── db
│   │       └── wal
│   │           ├── 0.tmp
│   │           └── 0000000000000000-0000000000000000.wal
│   └── etcd3
│       └── member
│           ├── snap
│           │   └── db
│           └── wal
│               ├── 0.tmp
│               └── 0000000000000000-0000000000000000.wal
└── docker-compose.yml

13 directories, 11 files

# 初始状态没有数据 secret不存在
gongpengjun@mbp docker-compose-etcd$ etcdctl --endpoints=http://127.0.0.1:12379 get secret

# 设置并获取secret
gongpengjun@mbp docker-compose-etcd$ etcdctl --endpoints=http://127.0.0.1:12379 put secret gpj
OK
gongpengjun@mbp docker-compose-etcd$ etcdctl --endpoints=http://127.0.0.1:12379 get secret
secret
gpj

# 关闭etcd cluster
gongpengjun@mbp docker-compose-etcd$ docker-compose down
Stopping etcd-cluster_etcd-1_1 ... done
Stopping etcd-cluster_etcd-3_1 ... done
Stopping etcd-cluster_etcd-2_1 ... done
Removing etcd-cluster_etcd-1_1 ... done
Removing etcd-cluster_etcd-3_1 ... done
Removing etcd-cluster_etcd-2_1 ... done
Removing network etcd-cluster_bridge-network

# 启动etcd cluster
gongpengjun@mbp docker-compose-etcd$ docker-compose up -d
Creating network "etcd-cluster_bridge-network" with driver "bridge"
Creating etcd-cluster_etcd-1_1 ... done
Creating etcd-cluster_etcd-3_1 ... done
Creating etcd-cluster_etcd-2_1 ... done

# 启动后可以直接查询到secret
gongpengjun@mbp docker-compose-etcd$ etcdctl --endpoints=http://127.0.0.1:12379 get secret
secret
gpj
```

### Docker Images

- [docker-image-etcd](https://quay.io/repository/coreos/etcd?tab=tags&tag=v3.5.7)
- [etcd official release](https://github.com/etcd-io/etcd/releases/tag/v3.5.7)
