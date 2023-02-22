# SQLAlchemy Tutorial

---


## Connection

There are two ways to pass the parameters for creating the connection:
Using a URI/URL string (the classic way) or by passing an instance of `sqlalchemy.URL`:

Passing the string:
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

url = 'mysql://jake:1234@127.0.0.1:3307/ecommerce_shop'  # 'driver_name://username:password@address/database_name'
engine = create_engine(url=url, echo=True)
Session = sessionmaker(bind=engine)
```


```python
from sqlalchemy import create_engine, URL
from sqlalchemy.orm import sessionmaker

url = URL.create(
        drivername='mysql',
        username='jake',
        password='1234',
        host='127.0.0.1:3307',
        database='ecommerce_shop'
    )
engine = create_engine(url=url, echo=True)
Session = sessionmaker(bind=engine)
```

The `engine` object is like the connection to the database,
and the `Session` is the multi-use cursor which we'll use to perform all the SQL transactions/queries.


## Creating Tables

There are two ways to create the tables for the database,
the first is to execute a `CREATE` string (the traditional way), and the second is to create a class (ORM)
that we can use for creating the table, inserting records, and performing `SELECT` queries.

Passing a string:
```python
from sqlalchemy import text

s = """
CREATE TABLE IF NOT EXISTS clients (
    id INTEGER,
    name VARCHAR(255),
    address VARCHAR(255),
    is_premium_client BOOLEAN
)
"""

with Session() as session:
    session.execute(text(s))
    session.commit()
```
Output:
```
2023-02-22 12:27:18,507 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 12:27:18,507 INFO sqlalchemy.engine.Engine
CREATE TABLE IF NOT EXISTS clients (
    id INTEGER PRIMARY KEY,
    name VARCHAR(255),
    address VARCHAR(255),
    is_premium_client BOOLEAN
)
2023-02-22 12:27:18,507 INFO sqlalchemy.engine.Engine [generated in 0.00012s] ()
2023-02-22 12:27:18,508 INFO sqlalchemy.engine.Engine COMMIT
```

Or using
```python
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy import Column, INTEGER, VARCHAR, BOOLEAN


# create a class that inherits from DeclarativeBase, which all our table-classes will inherit from
class Base(DeclarativeBase):
    pass


class Client(Base):
    __tablename__ = 'clients'

    id = Column('id', INTEGER, primary_key=True)
    name = Column('name', VARCHAR(255))
    address = Column('address', VARCHAR(255))
    is_premium_client = Column('is_premium_client', BOOLEAN)

    def __init__(self, id: int, name: str, address: str, is_premium_client: bool, **kwargs):
        super().__init__(**kwargs)
        self.id = id
        self.name = name
        self.address = address
        self.is_premium_client = is_premium_client

    def __repr__(self) -> str:
        return f'Client(id={self.id}, name={self.name}, address={self.address})'


# when you call `create_all()` it will create all thr tables that inherit from `Base` (only if they don't already exist)
Base.metadata.create_all(bind=engine)
```
This code will generate the following SQL transactions
```
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine PRAGMA main.table_info("clients")
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine [raw sql] ()
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine PRAGMA temp.table_info("clients")
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine [raw sql] ()
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine
CREATE TABLE clients (
	id INTEGER NOT NULL,
	name VARCHAR(255),
	address VARCHAR(255),
	is_premium_client BOOLEAN,
	PRIMARY KEY (id)
)
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine [no key 0.00005s] ()
2023-02-22 14:01:50,832 INFO sqlalchemy.engine.Engine COMMIT
```


## Inserting Records

To insert new records we can either pass a string with SQL query, or create an instance of
the class and pass that directly:

Passing a string:
```python
s = """
INSERT INTO clients (id, name, address, is_premium_client)
VALUES (3176, "John Doe", "2910 Kyle Street Julesburg, NE", false)
"""

with Session() as session:
    session.execute(text(s))
    session.commit()
```
Output:
```
2023-02-22 14:11:10,525 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 14:11:10,525 INFO sqlalchemy.engine.Engine
INSERT INTO clients (id, name, address, is_premium_client)
VALUES (3176, "John Doe", "2910 Kyle Street Julesburg, NE", false)
2023-02-22 14:11:10,526 INFO sqlalchemy.engine.Engine [generated in 0.00023s] ()
2023-02-22 14:11:10,526 INFO sqlalchemy.engine.Engine COMMIT
```

By creating an instance of the class-table:
```python
new_record = Client(id=3176, name='John Doe', address='2910 Kyle Street Julesburg, NE', is_premium_client=False)

with Session() as session:
    session.add(new_record)
    session.commit()
```
Output:
```
2023-02-22 14:19:25,593 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 14:19:25,594 INFO sqlalchemy.engine.Engine INSERT INTO clients (id, name, address, is_premium_client) VALUES (?, ?, ?, ?)
2023-02-22 14:19:25,594 INFO sqlalchemy.engine.Engine [generated in 0.00031s] (3176, 'John Doe', '2910 Kyle Street Julesburg, NE', 0)
2023-02-22 14:19:25,595 INFO sqlalchemy.engine.Engine COMMIT
```

### Adding Multiple Records

Using the table-class we can easily add multiple records:
```python
new_records = [
    Client(id=95814, name='Naomi Vazquez', address='471 Timber Ridge Road Sacramento, CA', is_premium_client=False),
    Client(id=60089, name='Marion White', address='1474 John Calvin Drive Buffalo Grove, IL', is_premium_client=True),
    Client(id=43215, name='Bernice Stiver', address='4091 Quilly Lane Columbus, OH', is_premium_client=False),
    Client(id=76401, name='Hilda Rios', address='4649 Beeghley Street Stephenville, TX', is_premium_client=True),
]
with Session() as session:
    session.add_all(new_records)
    session.commit()
```
```
2023-02-22 14:24:26,970 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 14:24:26,971 INFO sqlalchemy.engine.Engine INSERT INTO clients (id, name, address, is_premium_client) VALUES (?, ?, ?, ?)
2023-02-22 14:24:26,971 INFO sqlalchemy.engine.Engine [generated in 0.00015s] [(95814, 'Naomi Vazquez', '471 Timber Ridge Road Sacramento, CA', 0), (60089, 'Marion White', '1474 John Calvin Drive Buffalo Grove, IL', 1), (43215, 'Bernice Stiver', '4091 Quilly Lane Columbus, OH', 0), (76401, 'Hilda Rios', '4649 Beeghley Street Stephenville, TX', 1)]
2023-02-22 14:24:26,971 INFO sqlalchemy.engine.Engine COMMIT
```


## Selecting records

The equivalent of a `SELECT * FROM table` query, is:
```python
from sqlalchemy import select

with Session() as session:
    all_records = session.scalars(select(Client))
    session.commit()

    for x in all_records:
        print(x)
```
The SQL generated:
```
2023-02-22 14:33:40,597 INFO sqlalchemy.engine.Engine BEGIN (implicit)
2023-02-22 14:33:40,598 INFO sqlalchemy.engine.Engine SELECT clients.id, clients.name, clients.address, clients.is_premium_client
FROM clients
2023-02-22 14:33:40,598 INFO sqlalchemy.engine.Engine [cached since 373.3s ago] ()
2023-02-22 14:33:40,598 INFO sqlalchemy.engine.Engine COMMIT
```
And our records:
```
Client(id=3176, name=John Doe, address=2910 Kyle Street Julesburg, NE)
Client(id=95814, name=Naomi Vazquez, address=471 Timber Ridge Road Sacramento, CA)
Client(id=60089, name=Marion White, address=1474 John Calvin Drive Buffalo Grove, IL)
Client(id=43215, name=Bernice Stiver, address=4091 Quilly Lane Columbus, OH)
Client(id=76401, name=Hilda Rios, address=4649 Beeghley Street Stephenville, TX)
```

`SELECT * FROM clients WHERE is_premium_client IS true`
```python
with Session() as session:
    stmt = select(Client).where(Client.is_premium_client == True)
    all_records = session.scalars(stmt)
    session.commit()

    for x in all_records:
        print(x)
```
```
Client(id=60089, name=Marion White, address=1474 John Calvin Drive Buffalo Grove, IL)
Client(id=76401, name=Hilda Rios, address=4649 Beeghley Street Stephenville, TX)
```



`SELECT * FROM clients WHERE id IN (3176, 60089, 43215)`
```python
with Session() as session:
    stmt = select(Client).where(Client.id.in_([3176, 60089, 43215]))
    all_records = session.scalars(stmt)
    session.commit()

    for x in all_records:
        print(x)
```
```
Client(id=3176, name=John Doe, address=2910 Kyle Street Julesburg, NE)
Client(id=60089, name=Marion White, address=1474 John Calvin Drive Buffalo Grove, IL)
Client(id=43215, name=Bernice Stiver, address=4091 Quilly Lane Columbus, OH)
```


`SELECT * FROM clients WHERE id
(3176, 60089, 43215)`
```python
with Session() as session:
    stmt = select(Client).where(Client.)
    all_records = session.scalars(stmt)
    session.commit()

    for x in all_records:
        print(x)
```
```
Client(id=3176, name=John Doe, address=2910 Kyle Street Julesburg, NE)
Client(id=60089, name=Marion White, address=1474 John Calvin Drive Buffalo Grove, IL)
Client(id=43215, name=Bernice Stiver, address=4091 Quilly Lane Columbus, OH)
```

