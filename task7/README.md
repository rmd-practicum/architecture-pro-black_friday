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
          name:       { bsonType: "string" }
          category:   { bsonType: "string" }
          price:      { bsonType: "decimal" }
          stock_per_zone: { 
            bsonType: "object",
            properties: {
              zone:     { bsonType: "string" },
              quantity: { bsonType: "int" }
            }
          }
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

db.products.createIndex({ "category": 1 })
db.products.createIndex({ "price": 1 })
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
