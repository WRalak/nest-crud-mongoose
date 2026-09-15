# NestJS + Mongoose CRUD (Products)

A minimal, production-shaped CRUD API using NestJS and MongoDB via Mongoose.

## Setup

```bash
npm install
cp .env.example .env
# edit .env if your MongoDB isn't on localhost:27017
npm run start:dev
```

Requires a running MongoDB instance (local install, Docker, or Atlas). To spin one up quickly with Docker:

```bash
docker run -d -p 27017:27017 --name mongo mongo:7
```

## Project structure

```
src/
  main.ts                          # bootstrap, global ValidationPipe
  app.module.ts                    # ConfigModule + MongooseModule root connection
  products/
    schemas/product.schema.ts      # Mongoose schema (@Schema/@Prop)
    dto/create-product.dto.ts      # validation rules for POST
    dto/update-product.dto.ts      # PartialType of create dto for PATCH
    products.controller.ts         # REST routes
    products.service.ts            # Mongoose Model queries
    products.module.ts             # wires schema + controller + service
```

## Endpoints

| Method | Path            | Body                                   | Description        |
|--------|-----------------|-----------------------------------------|---------------------|
| POST   | /products       | `{ name, price, description?, stock?, isActive? }` | Create a product |
| GET    | /products       | –                                       | List all products   |
| GET    | /products/:id   | –                                       | Get one product     |
| PATCH  | /products/:id   | any subset of create fields              | Update a product    |
| DELETE | /products/:id   | –                                       | Delete a product (204) |

### Example requests

```bash
# Create
curl -X POST http://localhost:3000/products \
  -H "Content-Type: application/json" \
  -d '{"name":"Keyboard","price":49.99,"stock":10}'

# List
curl http://localhost:3000/products

# Get one
curl http://localhost:3000/products/<id>

# Update
curl -X PATCH http://localhost:3000/products/<id> \
  -H "Content-Type: application/json" \
  -d '{"price":39.99}'

# Delete
curl -X DELETE http://localhost:3000/products/<id>
```

## Notes / how it fits together

- **Schema** (`product.schema.ts`) defines the Mongo document shape with `@Schema({ timestamps: true })`, so `createdAt`/`updatedAt` are added automatically.
- **DTOs** use `class-validator` decorators; `main.ts` registers a global `ValidationPipe` with `whitelist: true` (strips unknown fields) and `transform: true` (turns plain JSON into typed DTO instances).
- **Service** injects the Mongoose `Model<ProductDocument>` via `@InjectModel(Product.name)` and does the actual `find`/`findById`/`findByIdAndUpdate`/`findByIdAndDelete` calls, throwing `NotFoundException`/`BadRequestException` where appropriate.
- **Module** registers the schema with `MongooseModule.forFeature(...)` so it's scoped to the `products` feature module, while `AppModule` opens the single shared connection with `MongooseModule.forRootAsync(...)` reading `MONGODB_URI` from `.env`.

## Extending this

To add another resource (e.g. `users`), copy the `products/` folder pattern: schema → dto → service → controller → module, then import the new module in `app.module.ts`. Use `nest g resource <name>` if you have the Nest CLI installed globally and want it scaffolded for you.
