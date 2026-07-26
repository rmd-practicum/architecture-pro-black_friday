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
          timestamp: { bsonType: "date" },
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
            bsonType: "array",
            items: {
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
          created_at: { bsonType: "date" },
          updated_at: { bsonType: "date" },
          expires_at: { bsonType: "date" }
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
sh.shardCollection("somedb.orders", { "client_id": "hashed" })
```

### products

Используем идентификатор товара в качестве ключа. Недостаток подхода — получение данных при поиске по категориям и диапазону цен будет происходить с нескольких шардов. Тем не менее, получение деталей отдельного товара, а так же обновление остатков, будет эффективно.

Стратегия — хэшированное шардирование.

```
sh.shardCollection("somedb.products", { "_id": "hashed" })
```

### carts

Используем идентификатор сессии в качестве ключа. При этом для авторизованных пользователей предпочтительно получать корзину по идентификатору сессии, а не пользователя — в противном случае, поиск корзины будет осуществляться на всех шардах.

Стратегия — хэшированное шардирование.

```
sh.shardCollection("somedb.carts", { "_id": "hashed" })
```
