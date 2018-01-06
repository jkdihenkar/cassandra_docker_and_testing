### Pull casssandra img from hub

docker pull casssandra

### starting up a cluster

```bash
docker run --name cass_node1 -p19040:9042 -d -e MAX_HEAP_SIZE='2g' -e HEAP_NEWSIZE="100m" cassandra
docker run --name cass_node2 -p19041:9042 -d -e MAX_HEAP_SIZE='2g' -e HEAP_NEWSIZE="100m" -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cass_node1)" cassandra
docker run --name cass_node3 -p19042:9042 -d -e MAX_HEAP_SIZE='2g' -e HEAP_NEWSIZE="100m" -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cass_node1)" cassandra
```

### check logs of container

```bash
# no follow
docker logs cass_node1
# follow
docker logs cass_node1 -f
```

## check cluster status

After the cluster is up:

```
[jd@jdpc cassandra_testing]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                         NAMES
d6e40c1e7f44        cassandra           "docker-entrypoint..."   39 seconds ago       Up 38 seconds       7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cass_node3
330d9b83b126        cassandra           "docker-entrypoint..."   About a minute ago   Up About a minute   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cass_node2
e09550cca2cb        cassandra           "docker-entrypoint..."   3 minutes ago        Up 3 minutes        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   cass_node1
```

```
[jd@jdpc cassandra_testing]$ docker exec cass_node1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  103.16 KiB  256          67.4%             0cddba3d-0039-4d95-9937-86b266996571  rack1
UN  172.17.0.2  108.61 KiB  256          65.9%             d54acb1f-3d28-44b9-a4b7-bd2abbe540f7  rack1
UN  172.17.0.4  89.35 KiB  256          66.7%             041e3d3f-4c53-47c7-833e-1ce3b5e6b5c9  rack1
```

Check ports are exposed as expected:

```
[jd@jdpc cassandra_testing]$ sudo netstat -ntlp | grep 1904
[sudo] password for jd:
tcp6       0      0 :::19040                :::*                    LISTEN      22420/docker-proxy-
tcp6       0      0 :::19041                :::*                    LISTEN      22587/docker-proxy-
tcp6       0      0 :::19042                :::*                    LISTEN      22829/docker-proxy-
```

Connecting to cqlsh:
```
[jd@jdpc cassandra_testing]$ docker exec -it cass_node1 cqlsh
Connected to Test Cluster at 127.0.0.1:9042.
[cqlsh 5.0.1 | Cassandra 3.11.1 | CQL spec 3.4.4 | Native protocol v4]
Use HELP for help.
cqlsh>
```

So we are up and running for tests!


### cleaning up containers once tests are done

```
[jd@jdpc cassandra_testing]$ docker stop cass_node1
cass_node1
[jd@jdpc cassandra_testing]$ docker stop cass_node2
cass_node2
[jd@jdpc cassandra_testing]$ docker stop cass_node3
cass_node3
[jd@jdpc cassandra_testing]$ docker rm cass_node3 cass_node1 cass_node2
cass_node3
cass_node1
cass_node2
```
