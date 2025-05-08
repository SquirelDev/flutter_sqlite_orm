# Flutter SQLite ORM

A Flutter SQLite ORM package similar to Odoo's ORM, providing an elegant abstraction over SQLite with support for complex relationship fields like Many2one, Many2many, and One2many.

## Features

- Elegant model definition with annotations
- Support for various field types (Integer, Char, Text, Boolean, Date, DateTime, etc.)
- Complex relationship fields (Many2one, One2many, Many2many)
- Powerful domain-based search API
- RecordSet API for working with sets of records
- Automatic database schema creation and migrations
- Code generation to minimize boilerplate
- Automatic field discovery without manual overrides

## Getting Started

Add the package to your pubspec.yaml:

```yaml
dependencies:
  flutter_sqlite_orm: ^0.1.0

dev_dependencies:
  build_runner: ^2.4.6
```

## Usage

### Define your models

Create your model files with the `part` directive for the generated code:

```dart
import 'package:flutter_sqlite_orm/flutter_sqlite_orm.dart';

part 'customer.g.dart';

@OrmModel(tableName: 'customers')
class Customer extends BaseModel with _$CustomerMixin {
  @Field()
  final CharField name = CharField('name', required: true);

  @Field()
  final CharField email = CharField('email');

  @Field()
  final One2many<Order> orders = One2many('orders', comodel: Order, inverse: 'customer_id');

  Customer();
}

// In order.dart
import 'package:flutter_sqlite_orm/flutter_sqlite_orm.dart';

part 'order.g.dart';

@OrmModel(tableName: 'orders')
class Order extends BaseModel with _$OrderMixin {
  @Field()
  final CharField name = CharField('name', required: true);

  @Field()
  final DateField date = DateField('date', required: true, defaultValue: DateTime.now);

  @Field()
  final Many2one<Customer> customer_id = Many2one('customer_id', comodel: Customer, required: true);

  Order();
}
```

### Generate the code

Run the build_runner to generate the necessary code:

```bash
flutter pub run build_runner build
```

This will generate the mixins that automatically handle field registration for you. The generated code includes:

1. **Abstract getters** for each field, which the model class must implement
2. **Automatic getFields() implementation** that includes all fields without manual overrides
3. **Factory methods** for creating model instances
4. **Table name getters** for automatic table creation

### Initialize the database

Register your models and initialize the ORM:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Register your models
  FlutterSqliteOrm.registerModel(Customer());
  FlutterSqliteOrm.registerModel(Order());

  // Initialize the ORM with automatic table creation
  await FlutterSqliteOrm.initialize(createTables: true);

  runApp(MyApp());
}
```

### CRUD Operations

```dart
// Create
final customer = await Customer.create({
  'name': 'John Doe',
  'email': 'john@example.com',
});

// Read
final customers = await BaseModel.search<Customer>([
  ['name', 'ilike', 'John'],
  ['active', '=', true],
]);

// Update
await customer.write({
  'name': 'John Smith',
});

// Delete
await customer.unlink();
```

## License

This project is licensed under the MIT License - see the LICENSE file for details.
