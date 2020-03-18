# Step by Step Upgrade MongoDB 3.4 to MongoDB 4.2

Remember, **DWYOR**

## Prerequisite(s)
- You have a java service
- You have a cluster mongodb 3.4, 1 primary and 2 secondary
- **You won't your java service stopped**

## How To
**Remember, please do step by step upgrading from 3.4 -> 3.6 -> 4.0 -> 4.2**

This is our existing example mongodb:
```
(A) Mongo 3.4 - RocksDB - Priority 10 - Primary
(B) Mongo 3.4 - RocksDB - Priority 9 - Secondary
(C) Mongo 3.4 - RocksDB - Priority 8 - Secondary
```

1. Create an instance mongodb 3.6 (D)
    - Make sure storage engine already **wiredTiger**
    - Make sure you have same **replSetName**, eg: rs0
2. Login to mongo cli in mongodb (A), then add mongodb (D) to cluster using command **rs.add**. eg: ``` ~ rs.add( { host: "IP_MONGODB:PORT_MONGODB", priority: 7 } ) ```
Now you will have existing mongodb like this:
```
(A) Mongo 3.4 - RocksDB - Priority 10 - Primary
(B) Mongo 3.4 - RocksDB - Priority 9 - Secondary
(C) Mongo 3.4 - RocksDB - Priority 8 - Secondary
(D) Mongo 3.6 - wiredTiger - Priority 7 - Secondary
```
3. Add mongodb (D) to your java service, then redeploy your java service
4. Change mongodb (D) as primary
```$bash
~ cfg = rs.conf()
~ cfg.members[3] = 11
```
Now you will have existing mongodb like this:
```
(A) Mongo 3.4 - RocksDB - Priority 10 - Secondary
(B) Mongo 3.4 - RocksDB - Priority 9 - Secondary
(C) Mongo 3.4 - RocksDB - Priority 8 - Secondary
(D) Mongo 3.6 - wiredTiger - Priority 11 - Primary
```

5. Make sure your service is stable and there isn't error about mongodb. Then remove mongodb (A) from your service. Then redeploy your java service
6. Shutdown mongodb (A), and your existing mongodb will like this:
```
(B) Mongo 3.4 - RocksDB - Priority 9 - Secondary
(C) Mongo 3.4 - RocksDB - Priority 8 - Secondary
(D) Mongo 3.6 - wiredTiger - Priority 11 - Primary
```
7. Repeat step 1 to step 6, and don't forget change priority to 10 9 8. You will have mongodb like this:
```
(D) Mongo 3.6 - wiredTiger - Priority 10 - Primary
(E) Mongo 3.6 - wiredTiger - Priority 9 - Secondary
(F) Mongo 3.6 - wiredTiger - Priority 8 - Secondary
```
8. Create an instance mongodb 4.0 (G)
    - Don't forget to run ``` db.adminCommand( { setFeatureCompatibilityVersion: "3.6" } ) ``` on your mongodb (G)
9. Login to mongo cli in mongodb (D), then add mongodb (G) to cluster using command **rs.add**. eg: ``` ~ rs.add( { host: "IP_MONGODB:PORT_MONGODB", priority: 7 } ) ```
Now you will have existing mongodb like this:
```
(D) Mongo 3.6 - wiredTiger - Priority 10 - Primary
(E) Mongo 3.6 - wiredTiger - Priority 9 - Secondary
(F) Mongo 3.6 - wiredTiger - Priority 8 - Secondary
(G) Mongo 4.0 - wiredTiger - Priority 7 - Secondary
```
10. Add mongodb (G) to your java service, then redeploy your java service
11. Change mongodb (G) as primary
```$bash
~ cfg = rs.conf()
~ cfg.members[3] = 11
```
Now you will have existing mongodb like this:
```
(D) Mongo 3.6 - wiredTiger - Priority 10 - Secondary
(E) Mongo 3.6 - wiredTiger - Priority 9 - Secondary
(F) Mongo 3.6 - wiredTiger - Priority 8 - Secondary
(G) Mongo 4.0 - wiredTiger - Priority 11 - Primary
```
12. Make sure your service is stable and there isn't error about mongodb. Then remove mongodb (G) from your service. Then redeploy your java service
13. Shutdown mongodb (D), and your existing mongodb will like this:
```
(E) Mongo 3.6 - wiredTiger - Priority 9 - Secondary
(F) Mongo 3.6 - wiredTiger - Priority 8 - Secondary
(G) Mongo 4.0 - wiredTiger - Priority 11 - Primary
```
14. Repeat step 8 to step 13, and don't forget change priority to 10 9 8. You will have mongodb like this:
```
(G) Mongo 4.0 - wiredTiger - Priority 10 - Primary
(H) Mongo 4.0 - wiredTiger - Priority 9 - Secondary
(I) Mongo 4.0 - wiredTiger - Priority 8 - Secondary
```
15. Login to server mongodb (I), and stop mongo service ``` systemctl stop mongod ```. Than install binaries mongodb 4.2
```$bash
~ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
~ dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb
~ percona-release enable psmdb-42 release
~ apt-get update
~ apt-get install percona-server-mongodb
```
16. After installation complete, start your mongo ``` systemctl start mongod ```
17. Repeat step 15 and 16 in mongodb (H)
18. Change primary mongodb from (G) to (I)
```$bash
cfg = rs.conf()
cfg.members[2].priority = 11
```
19. Wait until mongodb (I) be primary, than repeat step 15 and 16 in mongodb (G)
20. Change primary mongodb from (I) to (G)

Done