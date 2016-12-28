#provide-mssql

JSON model layer for Microsoft SQL Server (via Tedious) database in Express
apps.

##Installation

`npm i --save provide-mssql`

##Usage

Model (called from Express handler--for which consider [provide-handler](https://www.npmjs.com/package/provide-handler)):
```
model = require 'provide-mssql'

class TicketModel
  table: # DB table definition
    name: 'ticket' # table name in DB
    primary_key: '_id'
    schema: # whitelist of valid table columns (with datatypes) for queries
      _id: 'Int'
      customer_id: 'VarChar'
      received_date: 'Date'

  all: (filters...) =>
    # JSON object will be built into SQL query...
    params =
      select:
        customer: ['name: customer_name', 'region']
      join: [
        ['@.customer_id', 'customer._id']
      ]
    yield @request params

  find_by_customer: (id) =>
    params =
      select: {}
      join: [
        ['@.customer_id', 'customer._id']
      ]
      where: [
        ['@.customer_id', model.eq id]
      ]
    yield @request params


module.exports = model.provide TicketModel
```

Notice how we define a `primary_key` in the schema--this is to exclude that
column (`_id` in this example) from the fields affected by INSERT/UPDATE
queries, as we don't actually want to change that value.

Also note that model methods use "fat-arrow" to preserve context (so `this`
refers to the local class).

See `test/mssql.coffee` for more query examples. If you run into a query that
won't build properly, just pass it as a string (with optional params):

```
model_method_with_challenging query: (item_id) =>
  query = """SELECT *
    FROM table1
    LEFT JOIN table2 ON table1.field1 = table2.field1
      AND table1.field2 = table2.field2
    WHERE table1.id = @id"""

  yield @request query, @build_param 'id', 'Int', item_id
```

##Testing

`npm test`
