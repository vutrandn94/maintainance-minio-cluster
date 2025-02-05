# maintainance-minio-cluster


*To expand storage capacity for 1 Minio cluster. I have some solutions as follows.*
| Solution | Description |
| :--- | :--- |
| Expand storage capacity with LVM | Extending a Logical Volume |
| Add new pool & decommission a Server Pool | New pool must have enough capacity to store all data of the pool that needs to be cleaned up or replaced to expand storage capacity (recommend minimum x2 capacity current pool) |
| Replace the new mount volume with one that has more free disk space | Take advantage of the HEALING feature |