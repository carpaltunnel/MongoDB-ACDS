# JSON Schema Enforcement in MongoDB

https://www.mongodb.com/docs/upcoming/core/schema-validation/

While MongoDB is schemaless, validators allow us to enforce schema rules on our collections if desired.  We've already talked about how to create a JSON schema that we can use for input validation and documentation.  We can use that same schema to enforce validation at the database level with a few minor changes.  If all of your database manipulation is going through your API, which applies validation, then this can be redundant.  However, it does help provide validation in situations where the API is not used.  It is up to you to decide if database level schema enforcement and validation is required.

## MongoDB Validators
While it is possible to add validators to existing collections, it's typically a good idea to add them at the same time the collection is created.

To do this, we will add the `validator` property to the options object when we create our collection and use the `$jsonSchema` operator to indicate the schema to be validated against.

Let's take a look at the JSON schema we created in the previous lesson, modify it slightly for use with Mongo, and then create a new collection that uses it for validation.  The primary difference you'll notice is the usage of `bsonType` to define the type, rather than simply `type`.  We must remove the `format` property since it is unsupported by Mongo.  Additionally, the `integer` type is not supported so we need to change it to `number`.  All of our types must [conform to a valid BSON type](https://www.mongodb.com/docs/manual/reference/bson-types/).  

```json
{
  "bsonType": "object",
  "properties": {
    "id": {
      "description": "Unique identifier for the widget",
      "bsonType": "string",
    },
    "name": {
      "description": "Name of the widget that will be displayed to users",
      "bsonType": "string",
      "minLength": 3
    },
    "color": {
      "description": "Item color",
      "bsonType": "string",
      "enum": ["red", "green", "blue"]
    },
    "weight": {
      "description": "Item weight in kilograms",
      "bsonType": "number",
      "minimum": 1,
      "maximum": 25
    },
    "tags": {
      "description": "Item tags for searching",
      "bsonType": "array"
    },
    "tracking": {
      "type": "object",
      "properties": {
        "lastUpdated": {
          "description": "ISO8601 Date this widget was last updated",
          "bsonType": "date"
        },
        "lastUpdatedBy": {
          "description": "User ID that last updated the widget",
          "bsonType": "string"
        }
      }
    }
  },
  "required": ["id", "name"]
}
```

Now, we'll create a new `widgets` collection that uses this schema as a validator.

```javascript
db.createCollection("widgets", {
  validator: {
    $jsonSchema: {
      "bsonType": "object",
      "properties": {
        "id": {
          "description": "Unique identifier for the widget",
          "bsonType": "string",
        },
        "name": {
          "description": "Name of the widget that will be displayed to users",
          "bsonType": "string",
          "minLength": 3
        },
        "color": {
          "description": "Item color",
          "bsonType": "string",
          "enum": ["red", "green", "blue"]
        },
        "weight": {
          "description": "Item weight in kilograms",
          "bsonType": "number",
          "minimum": 1,
          "maximum": 25
        },
        "tags": {
          "description": "Item tags for searching",
          "bsonType": "array"
        },
        "tracking": {
          "type": "object",
          "properties": {
            "lastUpdated": {
              "description": "ISO8601 Date this widget was last updated",
              "bsonType": "date"
            },
            "lastUpdatedBy": {
              "description": "User ID that last updated the widget",
              "bsonType": "string"
            }
          }
        }
      },
      "required": ["id", "name"]
    }
  }
});
```

Be sure to check the return value to ensure that the query executed successfully.  Any errors in your JSON schema will be returned, or a simple `ok: 1.0` will be returned on success.

## Validation Action
By default, Mongo will error and refuse to insert documents that do not successfully pass the validator.  However, you can choose to allow inserts and only log validation failures.  To do this, add another property to your configuration options named `validationAction` with a value of `warn`, like so : 
```javascript
db.createCollection("widgets", {
  validator: {
    $jsonSchema: { ... }
  },
  validationAction: "warn"
});
```

You can also specify the value of `error` to indicate that inserts that do not pass validation should be blocked.  But, since that is the default, it is usually omitted.