# Задание 2
## Запуск инфраструктуры и приложения проекта

Запускаем необходимые контейнеры: 
- configSrv - конфигурационный сервер;
- shard_1 - шард №1
- shard_2 - шард №2
- mongos_router - роутер(определяет на какой шард пойдет запрос)
- pymongo_api - приложение
```bash
docker compose up -d
```
Инициализация конфигурационного сервера
```bash
docker compose exec -T configSrv mongosh --port 27017 --quiet <<EOF

rs.initiate(
  {
    _id : "config_server",
       configsvr: true,
    members: [
      { _id : 0, host : "configSrv:27017" }
    ]
  }
);
EOF
```
Инициализация шарда №1
```bash
docker compose exec -T shard_1 mongosh --port 27018 --quiet <<EOF

rs.initiate(
    {
      _id : "shard_1",
      members: [
        { _id : 0, host : "shard_1:27018" },
       // { _id : 1, host : "shard_2:27019" }
      ]
    }
);
EOF
```
Инициализация шарда №2
```bash
docker compose exec -T shard_2 mongosh --port 27019 --quiet <<EOF

rs.initiate(
    {
      _id : "shard_2",
      members: [
       // { _id : 0, host : "shard_1:27018" },
        { _id : 1, host : "shard_2:27019" }
      ]
    }
  );
EOF
```
Инициализация роутера
```bash
docker compose exec -T mongos_router mongosh --port 27020 --quiet <<EOF

sh.addShard( "shard_1/shard_1:27018");
sh.addShard( "shard_2/shard_2:27019");
sh.enableSharding("somedb");
sh.shardCollection("somedb.helloDoc", { "name" : "hashed" } )
EOF
```

Сохраняем в коллекцию helloDoc 1000 элементов (запросы идут на роутер)
```bash
docker compose exec -T mongos_router mongosh --port 27020 --quiet <<EOF

use somedb
for(var i = 0; i < 1000; i++) db.helloDoc.insert({age:i, name:"ly"+i})
db.helloDoc.countDocuments() 
EOF
```
Проверяем количество элементов в коллекции helloDoc на шарде №1
```bash
docker compose exec -T shard_1 mongosh --port 27018 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
```
Проверяем количество элементов в коллекции helloDoc на шарде №2
```bash
docker compose exec -T shard_2 mongosh --port 27019 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
```
**Результат:** количество элементов на шард №1 +  шард №2 = 1000