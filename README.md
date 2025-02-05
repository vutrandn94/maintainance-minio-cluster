# maintainance-minio-cluster


*To expand storage capacity for 1 Minio cluster. I have some solutions as follows.*

| Solution | Description | Use cases |
| :--- | :--- | :--- |
| Expand storage capacity with LVM | Extending a Logical Volume | Expand storage capacity |
| Add new pool & decommission a Server Pool | New pool must have enough capacity to store all data of the pool that needs to be cleaned up or replaced to expand storage capacity (recommend minimum x2 capacity current pool) | Expand storage capacity, replace/upgrade devices, disks, data storage location |
| Replace the new mount volume with one that has more free disk space | Take advantage of the HEALING feature | Expand storage capacity, replace/upgrade disk storage |

*In this article, I will describe how to do it: "Add new pool & decommission a server pool" and "replace the new mount volume with one that has more free disk space"*


## Add new pool & decommission a server pool
> [!TIP]
> Reference source: https://min.io/docs/minio/linux/operations/install-deploy-manage/decommission-server-pool.html#

> [!IMPORTANT]  
> Problem: I currently have a pool with a used storage capacity of 85%. Therefore, I need to have an upgrade solution to increase the storage capacity of Minio before running out of storage capacity.
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