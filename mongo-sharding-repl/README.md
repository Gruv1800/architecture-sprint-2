# Задание 3
## Запуск инфраструктуры и приложения проекта

Запускаем необходимые контейнеры: 
- configSrv - конфигурационный сервер;
- shard_1_* - набор реплик шарда №1
- shard_2_* - набор реплик шарда №2
- mongos_router - роутер(определяет на какой шард пойдет запрос)
- pymongo_api - приложение
```bash
docker compose up -d
```
Создаем набор реплик для шарда №1
```bash
docker compose exec -T shard_1_1 mongosh --port 27018 --quiet <<EOF
rs.initiate({_id: "shard_1", members: [
  {_id: 0, host: "shard_1_1:27018"},
  {_id: 1, host: "shard_1_2:27118"},
  {_id: 2, host: "shard_1_3:27218"}
]}) 
EOF
```
Создаем набор реплик для шарда №2
```bash
docker compose exec -T shard_2_1 mongosh --port 27019 --quiet <<EOF
rs.initiate({_id: "shard_2", members: [
  {_id: 0, host: "shard_2_1:27019"},
  {_id: 1, host: "shard_2_2:27119"},
  {_id: 2, host: "shard_2_3:27219"}
]}) 
EOF
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
docker compose exec -T shard_1_1 mongosh --port 27018 --quiet <<EOF

rs.initiate(
    {
      _id : "shard_1_rep",
      members: [
        { _id : 0, host : "shard_1_1:27018" },
        { _id : 1, host : "shard_1_2:27118" },
        { _id : 2, host : "shard_1_3:27218" }
       // { _id : 3, host : "shard_2_1:27019" },
       // { _id : 4, host : "shard_2_2:27119" },
       // { _id : 5, host : "shard_2_3:27219" }
      ]
    }
);
EOF
```
Инициализация шарда №2
```bash
docker compose exec -T shard_2_1 mongosh --port 27019 --quiet <<EOF

rs.initiate(
    {
      _id : "shard_2_rep",
      members: [
       // { _id : 0, host : "shard_1_1:27018" },
       // { _id : 1, host : "shard_1_2:27118" },
       // { _id : 2, host : "shard_1_3:27218" }
        { _id : 3, host : "shard_2_1:27019" },
        { _id : 4, host : "shard_2_2:27119" },
        { _id : 5, host : "shard_2_3:27219" }
      ]
    }
  );
EOF
```
Инициализация роутера
```bash
docker compose exec -T mongos_router mongosh --port 27020 --quiet <<EOF

sh.addShard( "shard_1/shard_1_1:27018");
sh.addShard( "shard_1/shard_1_2:27118");
sh.addShard( "shard_1/shard_1_3:27218");
sh.addShard( "shard_2/shard_2_1:27019");
sh.addShard( "shard_2/shard_2_2:27119");
sh.addShard( "shard_2/shard_2_3:27219");
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
Проверяем количество элементов в коллекции helloDoc на шарде №1 (на каждой из реплик должно быть одинаковое значение)
```bash
docker compose exec -T shard_1_1 mongosh --port 27018 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
docker compose exec -T shard_1_2 mongosh --port 27118 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
docker compose exec -T shard_1_3 mongosh --port 27218 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
```

Проверяем количество элементов в коллекции helloDoc на шарде №2(на каждой из реплик должно быть одинаковое значение)
```bash
docker compose exec -T shard_2_1 mongosh --port 27019 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
docker compose exec -T shard_2_2 mongosh --port 27119 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
docker compose exec -T shard_2_3 mongosh --port 27219 --quiet <<EOF
use somedb;
db.helloDoc.countDocuments();
EOF
```

**Результат:** количество элементов на `shard_1_*` +  `shard_2_*` = 1000