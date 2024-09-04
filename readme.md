## 1. Initialize the config server replica set
```
docker exec -it hash-index-sharding-cfgsvr1-1 mongosh --eval '
rs.initiate({
  _id: "cfgrs",
  configsvr: true,
  members: [{ _id: 0, host: "cfgsvr1:27017" }]
})
'
```

## 2. Initialize each shard replica set:
### 2.1 Shard 1
```
docker exec -it hash-index-sharding-shard1-1 mongosh --eval '
rs.initiate({
  _id: "shard1rs",
  members: [{ _id: 0, host: "shard1:27017" }]
})
'
```

### 2.2 Shard 2
```
docker exec -it hash-index-sharding-shard2-1 mongosh --eval '
rs.initiate({
  _id: "shard2rs",
  members: [{ _id: 0, host: "shard2:27017" }]
})
'
```

### 2.3 Shard 3
```
docker exec -it hash-index-sharding-shard3-1 mongosh --eval '
rs.initiate({
  _id: "shard3rs",
  members: [{ _id: 0, host: "shard3:27017" }]
})
'
```

## 3. Add the shards to the cluster through the mongos router:
```
docker exec -it hash-index-sharding-mongos-1 mongosh --eval '
sh.addShard("shard1rs/shard1:27017");
sh.addShard("shard2rs/shard2:27017");
sh.addShard("shard3rs/shard3:27017");
'
```

## 4. Enable sharding for your database and collection:
```
docker exec -it hash-index-sharding-mongos-1 mongosh --eval '
sh.enableSharding("mbDBName");
sh.shardCollection("mbDBName.myCollection", { "_id": "hashed" });
'
```

## 5. Exec in container to count
```
docker exec -it hash-index-sharding-mongos-1 mongosh
use mbDBName;
db.myCollection.countDocuments();
```

## 6. Inserting some random data
```
docker exec -it hash-index-sharding-mongos-1 mongosh
use mbDBName;
for (let i = 0; i < 1000; i++) {
  db.myCollection.insertOne({
    randomData: Math.random().toString(36).substring(7),
    timestamp: new Date()
  });
}
print("Inserted 1000 documents");
```

## 7. Get Shard Distribution
```
docker exec -it hash-index-sharding-mongos-1 mongosh
use mbDBName;
let distrb = db.myCollection.getShardDistribution();
printjson(distrb);

print("Sharding status");
let status = sh.status();
printjson(status);
```

```
docker exec -it hash-index-sharding-mongos-1 mongosh --eval '
use config;
db.chunks.find().pretty();
'

docker exec -it hash-index-sharding-mongos-1 mongosh --eval '
sh.status();
'
```