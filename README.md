# Maintainance MinIO Cluster


*To expand storage capacity for 1 Minio cluster. I have some solutions as follows.*

| Solution | Description | Use cases |
| :--- | :--- | :--- |
| Expand storage capacity with LVM | Extending a Logical Volume | Expand storage capacity |
| Add new pool & decommission a Server Pool | New pool must have enough capacity to store all data of the pool that needs to be cleaned up or replaced to expand storage capacity (recommend minimum x2 capacity current pool) | Expand storage capacity, replace/upgrade devices, disks, data storage location |
| Replace the new mount volume with one that has more free disk space | Take advantage of the HEALING feature | Expand storage capacity, replace/upgrade disk storage |

*In this article, I will describe how to do it: "Add new pool & decommission a server pool" and "replace the new mount volume with one that has more free disk space"*


## SOLUTION: Add new pool & decommission a server pool
> [!TIP]
> Reference source: https://min.io/docs/minio/linux/operations/install-deploy-manage/decommission-server-pool.html#

> [!IMPORTANT]  
> Problem: I currently have a pool with a used storage capacity of 85%. Therefore, I need to have an upgrade solution to increase the storage capacity of Minio before running out of storage capacity.
```
┌─────┬─────────────────────────────────┬───────────────────────┬────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status │
│ 1st │ http://minio0{1...4}/mnt/data-0 │ 85.2% (total: 20 GiB) │ Active │
└─────┴─────────────────────────────────┴───────────────────────┴────────┘
```
###  Information about the servers deploying the lab
| Hostname | IP Address | Pools | Description | 
| :--- | :--- | :--- | :--- |
| minio01 | 172.31.40.231 | http://minio0{1...4}/mnt/data-0 | This pool will be decommissioned after performing the steps below  | 
| minio02 | 172.31.44.99 | http://minio0{1...4}/mnt/data-0 | This pool will be decommissioned after performing the steps below | 
| minio03 | 172.31.36.91 | http://minio0{1...4}/mnt/data-0 | This pool will be decommissioned after performing the steps below | 
| minio04 | 172.31.40.139 | http://minio0{1...4}/mnt/data-0 | This pool will be decommissioned after performing the steps below | 
| minio05 | 172.31.36.170 | http://minio0{5...8}/mnt/data-0 | This pool will replace the just decommissioned pool | 
| minio06 | 172.31.46.10 | http://minio0{5...8}/mnt/data-0 | This pool will replace the just decommissioned pool | 
| minio07 | 172.31.47.245 | http://minio0{5...8}/mnt/data-0 | This pool will replace the just decommissioned pool | 
| minio08 | 172.31.38.9 | http://minio0{5...8}/mnt/data-0 | This pool will replace the just decommissioned pool |

###  Template configure docker-compose before adding new pool on all nodes
> [!TIP]
> Reference: https://github.com/vutrandn94/minio-multi-node-single-drive

| NODE_NAME | MINIO_ROOT_USER | MINIO_ROOT_PASSWORD | LOCAL_TIMEZONE |
| :--- | :--- | :--- | :--- |
| Minio server hostname | Root user | Root user password | Local time zone (Ex: Asia/Ho_Chi_Minh) |
```
services:
  <NODE_NAME>:
    image: 'quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z'
    restart: always
    environment:
      MINIO_ROOT_USER: "<MINIO_ROOT_USER>"
      MINIO_ROOT_PASSWORD: "<MINIO_ROOT_PASSWORD>"
      TZ: "<LOCAL_TIMEZONE>"
    command: server --console-address ":9001" http://minio0{1...4}/mnt/data-0
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - /mnt/data-0:/mnt/data-0
    networks:
      - minio-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 1m
networks:
  minio-net:
    driver: bridge
```
### Pool information before adding new pool
```
bash-5.1# mc admin info myminio
●  minio01:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio02:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio03:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio04:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1


┌──────┬───────────────────────┬─────────────────────┬──────────────┐
│ Pool │ Drives Usage          │ Erasure stripe size │ Erasure sets │
│ 1st  │ 85.2% (total: 20 GiB) │ 4                   │ 1            │
└──────┴───────────────────────┴─────────────────────┴──────────────┘

17 GiB Used, 4 Buckets, 4 Objects
4 drives online, 0 drives offline, EC:2
```

### Step-by-step add new pool and decommission old pool
*Stop Minio cluster to adjust configuration to add new pool:*
```
# docker-compose down
```

*Edit configure docker-compose according to the template below on all nodes:*
> [!TIP]
> Reference: https://github.com/vutrandn94/minio-multi-node-single-drive

| NODE_NAME | MINIO_ROOT_USER | MINIO_ROOT_PASSWORD | LOCAL_TIMEZONE |
| :--- | :--- | :--- | :--- |
| Minio server hostname | Root user | Root user password | Local time zone (Ex: Asia/Ho_Chi_Minh) |
```
services:
  <NODE_NAME>:
    image: 'quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z'
    restart: always
    environment:
      MINIO_ROOT_USER: "<MINIO_ROOT_USER>"
      MINIO_ROOT_PASSWORD: "<MINIO_ROOT_PASSWORD>"
      TZ: "<LOCAL_TIMEZONE>"
    command: server --console-address ":9001" http://minio0{1...4}/mnt/data-0 http://minio0{5...8}/mnt/data-0
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - /mnt/data-0:/mnt/data-0
    networks:
      - minio-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 1m
networks:
  minio-net:
    driver: bridge
```

*Start Minio cluster and verify new pool added:*
```
# docker-compose up -d
```

```
bash-5.1# mc admin info myminio
●  minio01:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio02:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio03:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio04:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio05:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 2

●  minio06:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 2

●  minio07:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 2

●  minio08:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 8/8 OK 
   Drives: 1/1 OK 
   Pool: 2

┌──────┬───────────────────────┬─────────────────────┬──────────────┐
│ Pool │ Drives Usage          │ Erasure stripe size │ Erasure sets │
│ 1st  │ 85.2% (total: 20 GiB) │ 4                   │ 1            │
│ 2nd  │ 0.3% (total: 60 GiB)  │ 4                   │ 1            │
└──────┴───────────────────────┴─────────────────────┴──────────────┘

17 GiB Used, 4 Buckets, 4 Objects
8 drives online, 0 drives offline, EC:2
```

*Verify pool status:*
```
bash-5.1# mc admin decommission status myminio
┌─────┬─────────────────────────────────┬───────────────────────┬────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status │
│ 1st │ http://minio0{1...4}/mnt/data-0 │ 85.2% (total: 20 GiB) │ Active │
│ 2nd │ http://minio0{5...8}/mnt/data-0 │ 0.3% (total: 60 GiB)  │ Active │
└─────┴─────────────────────────────────┴───────────────────────┴────────┘
```

*Start decommission pool "http://minio0{1...4}/mnt/data-0":*
```
bash-5.1# mc admin decommission start myminio/ http://minio0{1...4}/mnt/data-0  

Decommission started successfully for `http://minio0{1...4}/mnt/data-0`.
```

*Check processs decommission status ("Draining" means the decommission process is in progress):*
```
bash-5.1# mc admin decommission status myminio http://minio0{1...4}/mnt/data-0

Decommissioning rate at 15 MiB/sec [11 GiB/20 GiB]
Started: 6 minutes ago
```

```
bash-5.1# mc admin decommission status myminio

┌─────┬─────────────────────────────────┬───────────────────────┬──────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status   │
│ 1st │ http://minio0{1...4}/mnt/data-0 │ 85.2% (total: 20 GiB) │ Draining │
│ 2nd │ http://minio0{5...8}/mnt/data-0 │ 17.6% (total: 60 GiB) │ Active   │
└─────┴─────────────────────────────────┴───────────────────────┴──────────┘
```

*Wait for the pool status to change from "Draining" to "Complete" then the pool decommission process is complete:*
```
bash-5.1# mc admin decommission status myminio

┌─────┬─────────────────────────────────┬───────────────────────┬──────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status   │
│ 1st │ http://minio0{1...4}/mnt/data-0 │ 1.0% (total: 20 GiB)  │ Complete │
│ 2nd │ http://minio0{5...8}/mnt/data-0 │ 42.5% (total: 60 GiB) │ Active   │
└─────┴─────────────────────────────────┴───────────────────────┴──────────┘
```

```
bash-5.1# mc admin decommission status myminio http://minio0{1...4}/mnt/data-0

Decommission of pool http://minio0{1...4}/mnt/data-0 is complete, you may now remove it from server command line
```
> [!NOTE]  
> At this point, all data in pool 1st has been migrated to pool 2nd and we can remove pool 1st from Minio Cluster to use the storage in pool 2nd (right after removing pool 1st, pool 2nd will automatically be changed to 1st when restarting the cluster).

### Step-by-step remove old pool
*Stop Minio cluster to adjust configuration to remove old pool:*
```
# docker-compose down
```

*Edit configure docker-compose according to the template below on all nodes:*
> [!TIP]
> Reference: https://github.com/vutrandn94/minio-multi-node-single-drive

| NODE_NAME | MINIO_ROOT_USER | MINIO_ROOT_PASSWORD | LOCAL_TIMEZONE |
| :--- | :--- | :--- | :--- |
| Minio server hostname | Root user | Root user password | Local time zone (Ex: Asia/Ho_Chi_Minh) |
```
services:
  <NODE_NAME>:
    image: 'quay.io/minio/minio:RELEASE.2025-01-20T14-49-07Z'
    restart: always
    environment:
      MINIO_ROOT_USER: "<MINIO_ROOT_USER>"
      MINIO_ROOT_PASSWORD: "<MINIO_ROOT_PASSWORD>"
      TZ: "<LOCAL_TIMEZONE>"
    command: server --console-address ":9001" http://minio0{5...8}/mnt/data-0
    ports:
      - 9000:9000
      - 9001:9001
    volumes:
      - /mnt/data-0:/mnt/data-0
    networks:
      - minio-net
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 1m
networks:
  minio-net:
    driver: bridge
```

*Start Minio cluster and re-verify:*
```
# docker-compose up -d
```

```
bash-5.1# mc admin info myminio
●  minio05:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio06:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio07:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

●  minio08:9000
   Uptime: 14 minutes 
   Version: 2025-01-20T14:49:07Z
   Network: 4/4 OK 
   Drives: 1/1 OK 
   Pool: 1

┌──────┬───────────────────────┬─────────────────────┬──────────────┐
│ Pool │ Drives Usage          │ Erasure stripe size │ Erasure sets │
│ 1st  │ 42.5% (total: 60 GiB) │ 4                   │ 1            │
└──────┴───────────────────────┴─────────────────────┴──────────────┘

26 GiB Used, 4 Buckets, 4 Objects
4 drives online, 0 drives offline, EC:2
```

```
bash-5.1# mc admin decommission status myminio

┌─────┬─────────────────────────────────┬───────────────────────┬──────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status   │
│ 1st │ http://minio0{5...8}/mnt/data-0 │ 42.5% (total: 60 GiB) │ Active   │
└─────┴─────────────────────────────────┴───────────────────────┴──────────┘
```


## SOLUTION: Replace mount volume with one that has more free disk space (Take advantage of the HEALING feature)

> [!IMPORTANT]  
> Problem: I currently have a pool with a used storage capacity of 85%. Therefore, I need to have an upgrade solution to increase the storage capacity of Minio before running out of storage capacity.
```
┌─────┬─────────────────────────────────┬───────────────────────┬────────┐
│ ID  │ Pools                           │ Drives Usage          │ Status │
│ 1st │ http://minio0{1...4}/mnt/data-0 │ 85.2% (total: 20 GiB) │ Active │
└─────┴─────────────────────────────────┴───────────────────────┴────────┘
```
###  Information about the servers deploying the lab
| Hostname | IP Address | Pools |
| :--- | :--- | :--- |
| minio01 | 172.31.40.231 | http://minio0{1...4}/mnt/data-0 |
| minio02 | 172.31.44.99 | http://minio0{1...4}/mnt/data-0 |
| minio03 | 172.31.36.91 | http://minio0{1...4}/mnt/data-0 |
| minio04 | 172.31.40.139 | http://minio0{1...4}/mnt/data-0 |