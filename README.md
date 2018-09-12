[![Build Status](https://travis-ci.org/tlivings/enjoi.png)](https://travis-ci.org/tlivings/enjoi) [![NPM version](https://badge.fury.io/js/enjoi.png)](http://badge.fury.io/js/enjoi)

# enjoi

Converts a JSON schema to a Joi schema for object validation.

add Converts to `allow(null)`

add Converts to `Joi.object().pattern(regex,Joi.any())`

add Converts to `Joi.string().valid(["1","2"])`

add Converts to `Joi.string().replace("a","1")`

add Converts to `Joi.string().when("key",{is:true,then:Joi.required()})`  Support only string mode for the time being

add Converts to `Joi.boolean().truthy(['Y',1]).falsy(['n',0]).replace("a","1")`

add Converts to `Joi.array().items(Joi.string(),Joi.number())`
### Schema Support

`enjoi` is built against json-schema v4, but does not support all of json-schema.

Here is a list of some missing keyword support still being worked on:

- `object:patternProperties` - TBD.

### API

- `enjoi(schema [, options])`
    - `schema` - a JSON schema or a string type representation (such as `'integer'`).
    - `options` - an (optional) object of additional options such as `subSchemas` and custom `types`.
        - `subSchemas` - an (optional) object with keys representing schema ids, and values representing schemas.
        - `types` - an (optional) object  with keys representing type names and values representing a Joi schema.
        - `refineType(type, format)` - an (optional) function to call to apply to type based on the type and format of the JSON schema.
        - `strictMode` - make schemas `strict(value)` with a default value of `false`.

Example:

```javascript
const Joi = require('joi');
const Enjoi = require('enjoi');

const schema = Enjoi({
    type: 'object',
    properties: {
        firstName: {
            description: 'First name.',
            type: 'string'
        },
        lastName: {
            description: 'Last name.',
            type: 'string',
            enum:["zxc","qwe"]
        },
        age: {
            description: 'Age in years',
            type: 'integer',
            minimum: 1,
            nullable:true   //allow(null)
        }
    },
    'required': ['firstName', 'lastName']
});

Joi.validate({firstName: 'John', lastName: 'Doe', age: null}, schema, function (error, value) {
    error && console.log(error);
});
```

Can also call `validate` directly on the created schema.

```javascript
schema.validate({firstName: 'John', lastName: 'Doe', age: 45}, function (error, value) {
    error && console.log(error);
});
```

### Sub Schemas

Sub-schemas can be provided through the `subSchemas` option for `$ref` values to lookup against.

Example:

```javascript
const schema = Enjoi({
    type: 'object',
    properties: {
        a: {
            $ref: '#/b' // # is root schema
        },
        b: {
            type: 'string'
        },
        c: {
            $ref: '#sub/d' // #sub is 'sub' under subSchemas.
        }
    }
}, {
    subSchemas: {
        sub: {
            d: {
                'type': 'string'
            }
        }
    }
});
```

### Custom Types

Custom types can be provided through the `types` option.

```javascript
const schema = Enjoi({
    type: 'thing'
}, {
    types: {
        thing: Joi.any()
    }
});
```

### Refine Type

You can use the refine type function to help refine types based on `type` and `format`. This will allow transforming a type for lookup against the custom `types`.

```javascript
const schema = Enjoi({
    type: 'string',
    format: 'email'
}, {
    types: {
        email: Joi.string().email()
    },
    refineType(type, format) {
        if (type === 'string' && format === 'email') {
            return 'email';
        }
    }
});
```
