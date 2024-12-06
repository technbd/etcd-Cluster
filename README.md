

## etcd Cluster: 

`etcd` is a distributed **key-value store** designed to provide reliable data storage for distributed systems. It is widely used in cloud-native environments, including Kubernetes, as a primary data store for configuration, state management, and service discovery.

- Keys and Values: Stores data as key-value pairs.
- Cluster: A group of etcd nodes working together to ensure data consistency and availability.
- Leader Election: One node in the cluster is elected as the leader, responsible for processing all writes.



### Prerequisites:

- At leasttThree (03) Servers and hostnames like: node1, node2, node3
- Nodes must be able to communicate on ports:
    - 2379 (etcd client communication)
    - 2380 (etcd peer communication)


#### Mapped IP on your Local DNS: 

```
cat /etc/hosts

192.168.10.191 node1
192.168.10.192 node2
192.168.10.193 node3
```



### Install etcd Using Binary: (All nodes)


```
cd /opt

wget https://github.com/etcd-io/etcd/releases/download/v3.5.17/etcd-v3.5.17-linux-amd64.tar.gz
```


```
tar -xzvf etcd-v3.5.17-linux-amd64.tar.gz
cd etcd-v3.5.17-linux-amd64
```


```
ll


drwxr-xr-x  3 rcms rcms   40 Nov 12 22:32 Documentation
-rwxr-xr-x  1 rcms rcms  23M Nov 12 22:32 etcd
-rwxr-xr-x  1 rcms rcms  18M Nov 12 22:32 etcdctl
-rwxr-xr-x  1 rcms rcms  15M Nov 12 22:32 etcdutl
-rw-r--r--  1 rcms rcms  42K Nov 12 22:32 README-etcdctl.md
-rw-r--r--  1 rcms rcms 7.2K Nov 12 22:32 README-etcdutl.md
-rw-r--r--  1 rcms rcms 9.0K Nov 12 22:32 README.md
-rw-r--r--  1 rcms rcms 7.8K Nov 12 22:32 READMEv2-etcdctl.md
```


```
cp etcd* /usr/local/bin
```


```
ll /usr/local/bin
```


```
etcd --version


etcd Version: 3.5.17
Git SHA: 507c0de
Go Version: go1.22.9
Go OS/Arch: linux/amd64
```



```
etcdctl version

etcdctl version: 3.5.17
API version: 3.5
```



```
etcdutl version


etcdutl version: 3.5.17
API version: 3.5
```



### Create directories for `etcd`:

```
mkdir -p /var/lib/etcd
mkdir /etc/etcd
```


```
useradd --system --no-create-home etcd

or,

groupadd --system etcd
useradd -s /sbin/nologin --system -g etcd etcd
```


```
chown -R etcd:etcd /var/lib/etcd
chmod 0775 /var/lib/etcd
```


### Configure etcd on Each Node:

- Create and edit or modify the `etcd` configuration file (`/etc/etcd/etcd.conf`). Set the cluster parameters (advertise address, initial cluster, etc.).


_Syntax:_

```
name: "node"
data-dir: "/var/lib/etcd"

initial-advertise-peer-urls: "http://<IP1>:2380"
listen-peer-urls: "http://<IP1>:2380"

listen-client-urls: "http://<IP1>:2379,http://127.0.0.1:2379"
advertise-client-urls: "http://<IP1>:2379"

initial-cluster: "node1=http://<IP1>:2380,node2=http://<IP2>:2380,node3=http://<IP3>:2380"

initial-cluster-state: "new"
initial-cluster-token: "etcd-cluster"
```


#### For node1:

```
vim /etc/etcd/etcd.conf

name: "node1"
data-dir: "/var/lib/etcd"

initial-advertise-peer-urls: "http://192.168.10.191:2380"
listen-peer-urls: "http://192.168.10.191:2380"

advertise-client-urls: "http://192.168.10.191:2379"
listen-client-urls: "http://192.168.10.191:2379,http://127.0.0.1:2379"

#initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380"
initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380,node3=http://192.168.10.193:2380"

initial-cluster-state: "new"
initial-cluster-token: "etcd-cluster"
```


#### For node2:

```
vim /etc/etcd/etcd.conf

name: "node2"
data-dir: "/var/lib/etcd"

initial-advertise-peer-urls: "http://192.168.10.192:2380"
listen-peer-urls: "http://192.168.10.192:2380"

advertise-client-urls: "http://192.168.10.192:2379"
listen-client-urls: "http://192.168.10.192:2379,http://127.0.0.1:2379"

#initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380"
initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380,node3=http://192.168.10.193:2380"

initial-cluster-state: "new"
initial-cluster-token: "etcd-cluster"
```


#### For node3:

```
vim /etc/etcd/etcd.conf

name: "node3"
data-dir: "/var/lib/etcd"

initial-advertise-peer-urls: "http://192.168.10.193:2380"
listen-peer-urls: "http://192.168.10.193:2380"

advertise-client-urls: "http://192.168.10.193:2379"
listen-client-urls: "http://192.168.10.193:2379,http://127.0.0.1:2379"

#initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380"
initial-cluster: "node1=http://192.168.10.191:2380,node2=http://192.168.10.192:2380,node3=http://192.168.10.193:2380"

initial-cluster-state: "new"
initial-cluster-token: "etcd-cluster"
```





#### Configure Systemd and start etcd service:


```
vim /etc/systemd/system/etcd.service

[Unit]
Description=etcd key-value store
Documentation=https://github.com/etcd-io/etcd
After=network.target

[Service]
User=etcd
Type=notify

#Environment=ETCD_DATA_DIR=/var/lib/etcd
#Environment=ETCD_NAME=%m
#ExecStart=/usr/local/bin/etcd

EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.conf

Restart=always
RestartSec=10s
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```



```
systemctl daemon-reload

systemctl start etcd

systemctl restart etcd
systemctl status etcd
```



```
netstat -tlpn | grep etcd

tcp        0      0 192.168.10.191:2379     0.0.0.0:*               LISTEN      14523/etcd
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      14523/etcd
tcp        0      0 192.168.10.191:2380     0.0.0.0:*               LISTEN      14523/etcd
```


### Verify:


_Verify etcd is running:_

```
etcdctl endpoint health


### Output:

127.0.0.1:2379 is healthy: successfully committed proposal: took = 4.738ms
```


```
etcdctl --endpoints="http://192.168.10.191:2379" endpoint health
etcdctl --endpoints="http://192.168.10.192:2379" endpoint health
etcdctl --endpoints="http://192.168.10.193:2379" endpoint health


### Output:

http://192.168.10.191:2379 is healthy: successfully committed proposal: took = 2.579962ms
```


```
curl http://192.168.10.191:2379/health
```




```
etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379" endpoint health

etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379,http://192.168.10.193:2379" endpoint health


### Output:

http://192.168.10.191:2379 is healthy: successfully committed proposal: took = 4.328881ms
http://192.168.10.193:2379 is healthy: successfully committed proposal: took = 3.774002ms
http://192.168.10.192:2379 is healthy: successfully committed proposal: took = 6.498669m
```





```
etcdctl member list


### Output:

629800e15063a06, started, node1, http://192.168.10.191:2380, http://192.168.10.191:2379, false
6d33e35fe9d0aadb, started, node3, http://192.168.10.190:2380, http://192.168.10.193:2379, false
9d0245a62d2d402c, started, node2, http://192.168.10.192:2380, http://192.168.10.192:2379, false
```




```
etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379" member list
etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379,http://192.168.10.193:2379" member list


### Output:

629800e15063a06, started, node1, http://192.168.10.191:2380, http://192.168.10.191:2379, false
6d33e35fe9d0aadb, started, node3, http://192.168.10.190:2380, http://192.168.10.193:2379, false
9d0245a62d2d402c, started, node2, http://192.168.10.192:2380, http://192.168.10.192:2379, false
```


```
etcdctl member list -w table


### Output:

+------------------+---------+-------+----------------------------+----------------------------+------------+
|        ID        | STATUS  | NAME  |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
+------------------+---------+-------+----------------------------+----------------------------+------------+
|  629800e15063a06 | started | node1 | http://192.168.10.191:2380 | http://192.168.10.191:2379 |      false |
| 6d33e35fe9d0aadb | started | node3 | http://192.168.10.190:2380 | http://192.168.10.193:2379 |      false |
| 9d0245a62d2d402c | started | node2 | http://192.168.10.192:2380 | http://192.168.10.192:2379 |      false |
+------------------+---------+-------+----------------------------+----------------------------+------------+
```



_To determine the **leader** in your etcd cluster:_


```
etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379" endpoint status

etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379,http://192.168.10.193:2379" endpoint status


### Output:

http://192.168.10.191:2379, 629800e15063a06, 3.5.17, 20 kB, true, false, 7, 58, 58,
http://192.168.10.192:2379, 9d0245a62d2d402c, 3.5.17, 25 kB, false, false, 7, 58, 58,
http://192.168.10.193:2379, 6d33e35fe9d0aadb, 3.5.17, 20 kB, false, false, 7, 58, 58,
```


```
etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379" endpoint status --write-out=table

etcdctl --endpoints="http://192.168.10.191:2379,http://192.168.10.192:2379,http://192.168.10.193:2379" endpoint status --write-out=table


### Output:

+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|          ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://192.168.10.191:2379 |  629800e15063a06 |  3.5.17 |   20 kB |      true |      false |         7 |         58 |                 58 |        |
| http://192.168.10.192:2379 | 9d0245a62d2d402c |  3.5.17 |   25 kB |     false |      false |         7 |         58 |                 58 |        |
| http://192.168.10.193:2379 | 6d33e35fe9d0aadb |  3.5.17 |   20 kB |     false |      false |         7 |         58 |                 58 |        |
+----------------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```




### Save a test key-value pair: 

Let’s also try writing to etcd.

```
etcdctl put /message "Hello World"
```


```
etcdctl put foo "Hello etcd"
```





_Read the value of message back – It should work on all nodes._

```
etcdctl get /message
```


```
etcdctl get foo
```



### Links: 

- [etcd release | github.com](https://github.com/etcd-io/etcd/releases/)
- [Setup Etcd Cluster](https://computingforgeeks.com/setup-etcd-cluster-on-centos-debian-ubuntu/)

