## Схемы коллекций

### orders

```
db.createCollection(
  "orders", {
    validator: {
      $jsonSchema: {
        bsonType: "object",
        title: "Order object validation",
        properties: {
          client_id: { bsonType: "string" },
          timestamp: { bsonType: "timestamp" },
          items: {
            bsonType: "array",
            items: {
              bsonType: "object",
              properties: {
                product_id:   { bsonType: "objectId" },
                product_name: { bsonType: "string" },
                quantity:     { bsonType: "int" },
                price:        { bsonType: "decimal" }
              }
            }
          },
          status:   { bsonType: "string" },
          total:    { bsonType: "decimal" },
          geo_zone: { bsonType: "string" }
        }
      }
    }
  }
);

db.orders.createIndex({ "client_id": 1 });
```

### products

```
db.createCollection(
  "products", {
    validator: {
      $jsonSchema: {
        bsonType: "object",
        title: "Product object validation",
        properties: {
          name:       { bsonType: "string" },
          category:   { bsonType: "string" },
          price:      { bsonType: "decimal" },
          stock_per_zone: {
            bsonType: "array",
              items: {
              bsonType: "object",
              properties: {
                zone:     { bsonType: "string" },
                quantity: { bsonType: "int" }
              }
            }
          },
          attributes: {
            bsonType: "object",
            properties: {
              name:  { bsonType: "string" },
              value: {}
            }
          }
        }
      }
    }
  }
);

db.products.createIndex({ "category": 1, "price": 1 })
```

### carts

```
db.createCollection(
  "carts", {
    validator: {
      $jsonSchema: {
        bsonType: "object",
        title: "Cart object validation",
        properties: {
          user_id:    { bsonType: "string" },
          session_id: { bsonType: "string" },
          items: {
            bsonType: "array",
            items: {
              bsonType: "object",
              properties: {
                product_id: { bsonType: "objectId" },
                quantity:   { bsonType: "int" },
              }
            }
          },
          status:     { bsonType: "string", enum: ["active", "ordered", "abandoned"] },
          created_at: { bsonType: "timestamp" },
          updated_at: { bsonType: "timestamp" },
          expires_at: { bsonType: "timestamp" }
        }
      }
    }
  }
);

db.carts.createIndex({ "session_id": 1, "status": 1 })
db.carts.createIndex({ "user_id": 1, "status": 1 })
```

## Шардирование

### orders

Используем идентификатор клиента (`client_id`) в качестве ключа. Это позволит получать историю заказов каждого клиента с одного шарда.

Стратегия — хэшированное шардирование. Она позволит равномерно распределять заказы пользователей по шардам. Риск появления горячих шардов есть, но он достаточно низок: вероятно, что отдельные пользователи с большим количеством заказов будут распределены по шардам равномерно.

```
sh.shardCollection("db.orders", { "client_id": "hashed" })
```

### products

Используем идентификатор товара в качестве ключа. Недостаток подхода — получение данных при поиске по категориям и диапазону цен будет происходить с нескольких шардов.

Стратегия — хэшированное шардирование.

```
sh.shardCollection("db.products", { "_id": "hashed" })
```

### carts

Используем идентификатор корзины в качестве ключа. Это позволит равномерно распределить данные. Поскольку в один момент времени пользователь работает с одной корзиной, нет необходимости учитывать идентификатор пользователя при шардировании.

Стратегия — хэшированное шардирование.

```
sh.shardCollection("db.carts", { "_id": "hashed" })
```
