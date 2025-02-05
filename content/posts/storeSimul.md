+++
title = 'Store Simulation'
date = 2025-01-31T17:09:26+11:00
draft = true
+++

When you're early career and smart but not a genius it can be hard to find
motivated projects either to learn and to demonstrate past learning.  In my case
I wanted to demonstrate that I have a reasonable level of ability in managing
and using relational databases. Sure I've done 
a lot of practice questions, but these are unimpressive and only really address
writing queries, when there's a lot more to effective databaseing than just that.

So I decided to create a database for a large fictional store that would have
many suppliers, products, customers, employees etc. It could have ended with some
`create table...` statements but I thought it would be more fun to mimic some 
data generation processes. So there is a python process that creates fake customers
and inserts them into the database via an object relational mapping, a scala 
program that invents products and hooks into java database connectivity and so on.
This extended the project into a comparison of database connectivity patterns
and build tools.

Again, it could have ended there, with everything running locally and with a 
big database to play with but it seemed like a good opportunity to learn another
ubiquitous tool, namely docker. So each data generation process is containerized,
as is the database itself, ready to be deployed nowhere :)

## The Database
This is a no frills postgresql database, with tables like 
```sql
create table product
(
    id         serial primary key,
    name       text    not null,
    material   text,
    color      text,
    department text,
    inventory  integer not null
);
```
and 
```sql
create table supplier
(
    id      serial primary key,
    name    text not null,
    address text,
    phone   text not null,
    email   text not null,
    constraint check_email check ((email ~~ '%@%'::text))
);

create table supplier_products
(
    supplier_id integer not null references supplier (id),
    product_id  integer not null references product (id),
    price       bigint  not null,
    constraint check_price check ((price > 0)),
    primary key (supplier_id, product_id)
);
```
I am pleased to announce that these three tables alone contain 
**artificial and composite primary keys**, **foreign keys**, a
**many-to-one relationship** and 
some **data integrity constraints**.

### Free Gift!
Marketing decided it would be nice to send new customers a randomly
chosen welcome gift as soon as they create their account. This means
that we (as database administrators) need a trigger that executes 
after a row is inserted into the customers table. This looks like

```sql
create function send_welcome_product() returns trigger as
$welcome_product$
declare
    -- This language (plpgsql) requires you to declare your variables
    -- and their types before you can use them in the funcion body.
    product_id        integer;
    purchase_order_id integer;
begin
    -- Create a new order for this customer, keeping the order_id.
    insert into purchase_order(customer, address)
    values (NEW.id, NEW.primary_address)
    returning id into purchase_order_id;

    -- Select a random product id from products with non-zero inventory.
    select id
    into product_id
    from product
    where product.inventory >= 1
    order by random()
    limit 1;

    -- Decrement the inventory of that product.
    update product set inventory = inventory - 1 where id = product_id;

    -- Add this product to the new order.
    insert into purchase_order_products(purchase_order, product, quantity)
    values (purchase_order_id, product_id, 1);

    -- Log this on the console. 
    raise notice 'New customer: %', NEW.id;
    raise notice 'Created order %', purchase_order_id;
    raise notice 'Added product % to order', product_id;

    -- Return is ignored since this is an AFTER trigger, but the editor wants one.
    return NULL;
end;
$welcome_product$ language plpgsql;

-- The 'trigger' defines exactly when the above function should be executed.
create trigger welcome_product
    after insert
    on customer
    for each row
execute function send_welcome_product();
```
To me the syntax of plpgsql functions and the 'create trigger' command are
very simple once they're complete, but not necessarily so when you're actually
writing them.

## Data Generation
To populate the database, a haskell process invents suppliers, a python process 
invents products etc. Here is a table of the various components:

| Component | Language| DB Connector      |
|-----------|---------|-------------------|
| Suppliers | Haskell | postgresql-simple |
| Products  | Scala   | JDBC              |
| Customers | Python  | SQLAlchemy        |
| Orders    | Elixir  | Ecto              |
<!-- |           |         |                   | -->
<!-- |           |         |                   | -->
<!-- |           |         |                   | -->

I should point out that this is a totally kooky way to populate a database
with test data. There's no reason apart from experimentation and learning
to use so many different technologies to achieve quite a simple task.
The following section will briefly describe each of these.

### Customers

This is a python project using the `poetry` packaging and dependency management
tool. In the same category as `venv` (which it wraps), `conda` etc this tool 
tries to make python projects easier to share.

For a small project like this the use of a full object relational mapping (ORM)
is overkill, but it's a common way to interact with a database so I wanted to 
explore it.  This style of database interaction requires you to declare an exact
mapping from the rows in a table to a python class that can represent these rows.
For a simple database you can write these mappings by hand. For larger projects
you would probably consider a tool that automatically writes these modules.
I have the feeling that this may not be the most pleasant process, especially
for real databases with lots of relations, constraints etc. Anyway in this case
the customers model looks like this:
```python
from sqlalchemy import String
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column

class Base(DeclarativeBase):
    pass

class Customer(Base):
    __tablename__                = "customer"
    id: Mapped[int]              = mapped_column(primary_key=True)
    name: Mapped[str]            = mapped_column(String())
    email: Mapped[str]           = mapped_column(String())
    primary_address: Mapped[str] = mapped_column(String())
    def __repr__(self) -> str:
        return f"Customer(id:{self.id}, name:{self.name}, email:{self.email})"
```
which is all pretty self explanatory. Once you get this out of the way 
the syntax for interacting with the database is simple:
```python
with Session(engine) as session:
    customer = Customer(name=faker.name(),
                        email=faker.ascii_email(),
                        primary_address=faker.address())
    session.add(customer)
    session.commit()
```
Running this in a loop will generate as many fake customers as you like.

### Products
Next we have a scala prject build with `sbt` that invents products for the 
store. We start with a model for a product and a method that creates random
ones, like a 'Gorgeous Rubber Table' in 'mint green', which belongs in the 
'Clothing, Health & Industrial' department.

This is nothing fancy, but nice in scala:

```scala
import com.github.javafaker.{Faker}
import java.util.Locale

case class Product(
    name: String,
    material: String,
    color: String,
    department: String,
    inventory: Int
)

object Product {
  private val faker = new Faker(new Locale("en-AU"))
  private val rand = new util.Random

  def random() = {
    new Product(
      faker.commerce().productName(),
      faker.commerce().material(),
      faker.commerce().color(),
      faker.commerce().department(),
      rand.nextInt(1000) + 1
    )
  }
}
```
I chose to use a basic JDBC connection for this part. This is because I hadn't
used the JDBC before, and I feel like you should start with basic tools before 
moving on to things that 'improve' on them. Also the alternatives (Slick etc) 
are quite complicated.

The JDBC is a fairly low level interface, so you essentially write the skeleton
of an SQL query, then take extraordinary care in filling in the parts that change.
Here is a method that will do just that:

```scala
def writeNewProduct(connection: Connection) = {
    val statement = connection.createStatement()
    val sql =
      "INSERT INTO product (name, material, color, department, inventory) VALUES (?, ?, ?, ?, ?)"
    val preparedStatement = connection.prepareStatement(sql)

    val product = Product.random()
    preparedStatement.setString(1, product.name)
    preparedStatement.setString(2, product.material)
    preparedStatement.setString(3, product.color)
    preparedStatement.setString(4, product.department)
    preparedStatement.setInt(5, product.inventory)

    val rowsInserted = preparedStatement.executeUpdate()
    if (rowsInserted > 0) {
      logger.info(s"New product: $product")
    }
}
```
For a real project I think I would look at more sophisticated solutions, but it's nice
to know that JBDC is always there for something quick.

### Suppliers
Inventing a new supplier means 
1. inventing the supplier itself;
1. creating some rows in the 'supplier supplies product' relation.

In Haskell, the way to read or write a row to a table is to create a data-type corresponding 
to that table, and then to implement the `FromRow` or `ToRow` typeclass respectively.
In our case, we have a very natural supplier type:
```haskell
data Supplier = Supplier
  { supplier_name :: Text,
    address       :: Text,
    phone         :: Text,
    email         :: Text
  }
```
 and if we want to be able to write one to the database we must specify how to transform it 
to a database row:
```haskell
instance ToRow Supplier where
  toRow Supplier {..} = [toField supplier_name, toField address, toField phone, toField email]
```
TODO: explain this a bit.

This function will write a supplier to the database and then 
return the autogenerated ID of the new row:
```haskell
writeNewSupplier :: Connection -> IO Int
writeNewSupplier conn = do
  -- This is a random new supplier. 
  supplier <- generateNonDeterministic fakeSupplier
  [Only id] <-
    query
      conn
      "INSERT INTO supplier (name, address, phone, email) \
      \ VALUES (?, ?, ?, ?) \
      \ RETURNING id"
      supplier
  return id
```
As always things look a little different in Haskell, but doing the work early (implementing 
the typeclasses above) lets you use a pretty compact syntax when its time to actually do IO.

### Orders