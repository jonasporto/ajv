# ajv - Another JSON Schema Validator

Currently the fastest JSON Schema validator for node.js and browser.

It uses [doT templates](https://github.com/olado/doT) to generate super-fast validating functions.

[![Build Status](https://travis-ci.org/epoberezkin/ajv.svg?branch=master)](https://travis-ci.org/epoberezkin/ajv)
[![npm version](https://badge.fury.io/js/ajv.svg)](https://www.npmjs.com/package/ajv)
[![Code Climate](https://codeclimate.com/github/epoberezkin/ajv/badges/gpa.svg)](https://codeclimate.com/github/epoberezkin/ajv)
[![Coverage Status](https://coveralls.io/repos/epoberezkin/ajv/badge.svg?branch=master&service=github)](https://coveralls.io/github/epoberezkin/ajv?branch=master)


## Contents

- [Features](#features)
- [Performance](#performance)
- [Getting started](#getting-started)
- [Using in browser](#using-in-browser)
- Validation
  - [Keywords](#validation-keywords)
  - [Formats](#formats)
  - [$data reference](#data-reference)
  - [Defining custom keywords](#defining-custom-keywords)
  - [Asynchronous schema compilation](#asynchronous-compilation)
  - [Asynchronous validation](#asynchronous-validation)
- Modifying data during validation
  - [Filtering data](#filtering-data)
  - [Assigning defaults](#assigning-defaults)
  - [Coercing data types](#coercing-data-types)
- API
  - [Methods](#api)
  - [Options](#options)
  - [Validation errors](#validation-errors)
- [Packages using Ajv](#some-packages-using-ajv)
- [CLI, Tests, Contributing, History, License](#command-line-interface)


## Features

- ajv implements full [JSON Schema draft 4](http://json-schema.org/) standard:
  - all validation keywords (see [JSON-Schema validation keywords](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md))
  - full support of remote refs (remote schemas have to be added with `addSchema` or compiled to be available)
  - support of circular references between schemas
  - correct string lengths for strings with unicode pairs (can be turned off)
  - [formats](#formats) defined by JSON Schema draft 4 standard and custom formats (can be turned off)
  - [validates schemas against meta-schema](#api-validateschema)
- supports [browsers](#using-in-browser) and nodejs 0.10-5.0
- [asynchronous loading](#asynchronous-compilation) of referenced schemas during compilation
- "All errors" validation mode with [option allErrors](#options)
- [error messages with parameters](#validation-errors) describing error reasons to allow creating custom error messages
- i18n error messages support with [ajv-i18n](https://github.com/epoberezkin/ajv-i18n) package (version >= 1.0.0)
- [filtering data](#filtering-data) from additional properties
- [assigning defaults](#assigning-defaults) to missing properties and items
- NEW: [coercing data](#coercing-data-types) to the types specified in `type` keywords
- [custom keywords](#defining-custom-keywords)
- keywords `switch`, `constant`, `contains`, `patternGroups`, `patternRequired`, `formatMaximum` / `formatMinimum` and `exclusiveFormatMaximum` / `exclusiveFormatMinimum` from [JSON-schema v5 proposals](https://github.com/json-schema/json-schema/wiki/v5-Proposals) with [option v5](#options)
- [v5 meta-schema](https://raw.githubusercontent.com/epoberezkin/ajv/master/lib/refs/json-schema-v5.json#) for schemas using v5 keywords
- NEW: [v5 $data reference](#data-reference) to use values from the validated data as values for the schema keywords
- NEW: [asynchronous validation](#asynchronous-validation) of custom formats and keywords

Currently ajv is the only validator that passes all the tests from [JSON Schema Test Suite](https://github.com/json-schema/JSON-Schema-Test-Suite) (according to [json-schema-benchmark](https://github.com/ebdrup/json-schema-benchmark), apart from the test that requires that `1.0` is not an integer that is impossible to satisfy in JavaScript).


## Performance

ajv generates code to turn JSON schemas into javascript functions that are efficient for v8 optimization.

Currently ajv is the fastest validator according to these benchmarks:

- [json-schema-benchmark](https://github.com/ebdrup/json-schema-benchmark) - 70% faster than the second place
- [jsck benchmark](https://github.com/pandastrike/jsck#benchmarks) - 20-190% faster
- [z-schema benchmark](https://rawgit.com/zaggino/z-schema/master/benchmark/results.html)
- [themis benchmark](https://cdn.rawgit.com/playlyfe/themis/master/benchmark/results.html)


## Install

```
npm install ajv
```


## <a name="usage"></a>Getting started

Try it in the node REPL: https://tonicdev.com/npm/ajv


The fastest validation call:

```javascript
var Ajv = require('ajv');
var ajv = Ajv(); // options can be passed, e.g. {allErrors: true}
var validate = ajv.compile(schema);
var valid = validate(data);
if (!valid) console.log(validate.errors);
```

or with less code

```javascript
// ...
var valid = ajv.validate(schema, data);
if (!valid) console.log(ajv.errors);
// ...
```

or

```javascript
// ...
ajv.addSchema(schema, 'mySchema');
var valid = ajv.validate('mySchema', data);
if (!valid) console.log(ajv.errorsText());
// ...
```

See [API](#api) and [Options](#options) for more details.

ajv compiles schemas to functions and caches them in all cases (using schema stringified with [json-stable-stringify](https://github.com/substack/json-stable-stringify) as a key), so that the next time the same schema is used (not necessarily the same object instance) it won't be compiled again.

The best performance is achieved when using compiled functions returned by `compile` or `getSchema` methods (there is no additional function call).

__Please note__: every time validation function or `ajv.validate` are called `errors` property is overwritten. You need to copy `errors` array reference to another variable if you want to use it later (e.g., in the callback). See [Validation errors](#validation-errors)


## Using in browser

You can require ajv directly from the code you browserify - in this case ajv will be a part of your bundle.

If you need to use ajv in several bundles you can create a separate UMD bundle using `npm run bundle` script (thanks to [siddo420](https://github.com/siddo420)).

Then you need to load ajv in the browser:
```html
<script src="ajv.min.js"></script>
```

This bundle can be used with different module systems or creates global `Ajv` if no module system is found.

The browser bundle is available on [cdnjs](https://cdnjs.com/libraries/ajv).

Ajv was tested with these browsers:

[![Sauce Test Status](https://saucelabs.com/browser-matrix/epoberezkin.svg)](https://saucelabs.com/u/epoberezkin)


## Validation keywords

Ajv supports all validation keywords from draft 4 of JSON-schema standard:

- [type](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#type)
- [for numbers](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#keywords-for-numbers) - maximum, minimum, exclusiveMaximum, exclusiveMinimum, multipleOf
- [for strings](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#keywords-for-strings) - maxLength, minLength, pattern, format
- [for arrays](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#keywords-for-arrays) - maxItems, minItems, uniqueItems, items, additionalItems
- [for objects](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#keywords-for-objects) - maxProperties, minproperties, required, properties, patternProperties, additionalProperties, dependencies
- [compound keywords](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#keywords-for-all-types) - enum, not, oneOf, anyOf, allOf

With option `v5: true` Ajv also supports all validation keywords and [$data reference](#data-reference) from [v5 proposals](https://github.com/json-schema/json-schema/wiki/v5-Proposals) for JSON-schema standard:

- [switch](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#switch-v5-proposal) - conditional validation with a sequence of if/then clauses
- [contains](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#contains-v5-proposal) - check that array contains a valid item
- [constant](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#constant-v5-proposal) - check that data is equal to some value
- [patternGroups](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#patterngroups-v5-proposal) - a more powerful alternative to patternProperties
- [patternRequired](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#patternrequired-v5-proposal) - like `required` but with patterns that some property should match.
- [formatMaximum, formatMinimum, exclusiveFormatMaximum, exclusiveFormatMinimum](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md#formatmaximum--formatminimum-and-exclusiveformatmaximum--exclusiveformatminimum-v5-proposal) - setting limits for date, time, etc.

See [JSON-Schema validation keywords](https://github.com/epoberezkin/ajv/blob/master/KEYWORDS.md) for more details.


## Formats

The following formats are supported for string validation with "format" keyword:

- _date_: full-date according to [RFC3339](http://tools.ietf.org/html/rfc3339#section-5.6).
- _time_: time with optional time-zone.
- _date-time_: date-time from the same source (time-zone is mandatory). `date`, `time` and `date-time` validate ranges in `full` mode and only regexp in `fast` mode (see [options](#options)).
- _uri_: full uri with optional protocol.
- _email_: email address.
- _hostname_: host name acording to [RFC1034](http://tools.ietf.org/html/rfc1034#section-3.5).
- _ipv4_: IP address v4.
- _ipv6_: IP address v6.
- _regex_: tests whether a string is a valid regular expression by passing it to RegExp constructor.
- _uuid_: Universally Unique IDentifier according to [RFC4122](http://tools.ietf.org/html/rfc4122).
- _json-pointer_: JSON-pointer according to [RFC6901](https://tools.ietf.org/html/rfc6901).
- _relative-json-pointer_: relative JSON-pointer according to [this draft](http://tools.ietf.org/html/draft-luff-relative-json-pointer-00).

There are two modes of format validation: `fast` and `full`. This mode affects formats `date`, `time`, `date-time`, `uri`, `email`, and `hostname`. See [Options](#options) for details.

You can add additional formats and replace any of the formats above using [addFormat](#api-addformat) method.

You can find patterns used for format validation and the sources that were used in [formats.js](https://github.com/epoberezkin/ajv/blob/master/lib/compile/formats.js).


## $data reference

With `v5` option you can use values from the validated data as the values for the schema keywords. See [v5 proposal](https://github.com/json-schema/json-schema/wiki/$data-(v5-proposal)) for more information about how it works.

`$data` reference is supported in the keywords: constant, enum, format, maximum/minimum, exclusiveMaximum / exclusiveMinimum, maxLength / minLength, maxItems / minItems, maxProperties / minProperties, formatMaximum / formatMinimum, exclusiveFormatMaximum / exclusiveFormatMinimum, multipleOf, pattern, required, uniqueItems.

The value of "$data" should be a [relative JSON-pointer](http://tools.ietf.org/html/draft-luff-relative-json-pointer-00).

Examples.

This schema requires that the value in property `smaller` is less or equal than the value in the property larger:

```javascript
var schema = {
  "properties": {
    "smaller": {
      "type": number,
      "maximum": { "$data": "1/larger" }
    },
    "larger": { "type": number }
  }
};

var validData = {
  smaller: 5,
  larger: 7
};
```

This schema requires that the properties have the same format as their field names:

```javascript
var schema = {
  "additionalProperties": {
    "type": "string",
    "format": { "$data": "0#" }
  }
};

var validData = {
  'date-time': '1963-06-19T08:30:06.283185Z',
  email: 'joe.bloggs@example.com'
}
```

`$data` reference is resolved safely - it won't throw even if some property is undefined. If `$data` resolves to `undefined` the validation succeeds (with the exclusion of `constant` keyword). If `$data` resolves to incorrect type (e.g. not "number" for maximum keyword) the validation fails.


## Defining custom keywords

Starting from version 2.0.0 ajv supports custom keyword definitions.

The advantages of using custom keywords are:

- allow creating validation scenarios that cannot be expressed using JSON-Schema
- simplify your schemas
- help bringing a bigger part of the validation logic to your schemas
- make your schemas more expressive, less verbose and closer to your application domain
- implement custom data processors that modify your data and/or create side effects while the data is being validated

The concerns you have to be aware of when extending JSON-schema standard with custom keywords are the portability and understanding of your schemas. You will have to support these custom keywords on other platforms and to properly document these keywords so that everybody can understand them in your schemas.

You can define custom keywords with [addKeyword](#api-addkeyword) method. Keywords are defined on the `ajv` instance level - new instances will not have previously defined keywords.

Ajv allows defining keywords with:
- validation function
- compilation function
- macro function
- inline compilation function that should return code (as string) that will be inlined in the currently compiled schema.

Example. `range` and `exclusiveRange` keywords using compiled schema:

```javascript
ajv.addKeyword('range', { type: 'number', compile: function (sch, parentSchema) {
  var min = sch[0];
  var max = sch[1];

  return parentSchema.exclusiveRange === true
          ? function (data) { return data > min && data < max; }
          : function (data) { return data >= min && data <= max; }
} });

var schema = { "range": [2, 4], "exclusiveRange": true };
var validate = ajv.compile(schema);
console.log(validate(2.01)); // true
console.log(validate(3.99)); // true
console.log(validate(2)); // false
console.log(validate(4)); // false
```

See [Defining custom keywords](https://github.com/epoberezkin/ajv/blob/master/CUSTOM.md) for details.


## Asynchronous compilation

During asynchronous compilation remote references are loaded using supplied function. See `compileAsync` method and `loadSchema` [option](#options).

Example:

```javascript
var ajv = Ajv({ loadSchema: loadSchema });

ajv.compileAsync(schema, function (err, validate) {
	if (err) return;
	var valid = validate(data);
});

function loadSchema(uri, callback) {
	request.json(uri, function(err, res, body) {
		if (err || res.statusCode >= 400)
			callback(err || new Error('Loading error: ' + res.statusCode));
		else
			callback(null, body);
	});
}
```

__Please note__: [Option](#options) `missingRefs` should NOT be set to `"ignore"` or `"fail"` for asynchronous compilation to work.


## Asynchronous validation

Example in node REPL: https://tonicdev.com/esp/ajv-asynchronous-validation

Starting from version 3.5.0 you can define custom formats and keywords that perform validation asyncronously by accessing database or some service. You should add `async: true` in the keyword or format defnition (see [addFormat](#api-addformat) and [addKeyword](#api-addkeyword)).

If your schema uses asynchronous formats/keywords or refers to some schema that contains them it should have `"$async": true` keyword so that Ajv can compile it correctly. If asynchronous format/keyword or reference to asynchronous schema is used in the schema without `$async` keyword Ajv will throw an exception during schema compilation.

__Please note__: all asynchronous subschemas that are referenced from the current or other schemas should have `"$async": true` keyword as well, otherwise the schema compilation will fail.

Validation function for an asynchronous custom format/keyword should return a promise that resolves to `true` or `false`. Ajv compiles asynchronous schemas to either [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) (default) that can be optionally transpiled with [regenerator](https://github.com/facebook/regenerator) or to [es7 async function](http://tc39.github.io/ecmascript-asyncawait/) that can be transpiled with [nodent](https://github.com/MatAtBread/nodent) or with regenerator as well. You can also supply any other transpiler as a function. See [Options](#options).

The compiled validation function has `async: true` property (if the schema is asynchronous), so you can differentiate these functions if you are using both syncronous and asynchronous schemas.

If you are using generators, the compiled validation function can be either wrapped with [co](https://github.com/tj/co) (default) or returned as generator function, that can be used directly, e.g. in [koa](http://koajs.com/) 1.0. `co` is a small library, it is included in Ajv (both as npm dependency and in the browser bundle).

Generator functions are currently supported in Chrome, Firefox and node.js (0.11+); if you are using Ajv in other browsers or in older versions of node.js you should use one of available transpiling options. All provided async modes use global Promise class. If your platform does not have Promise you should use a polyfill that defines it.

Validation result will be a promise that resolves to `true` or rejects with an exception `Ajv.ValidationError` that has the array of validation errors in `errors` property.


Example:

```javascript
/**
 * without "async" and "transpile" options (or with option {async: true})
 * Ajv will choose the first supported/installed option in this order:
 * 1. native generator function wrapped with co
 * 2. es7 async functions transpiled with nodent
 * 3. es7 async functions transpiled with regenerator
 */

var ajv = Ajv();

ajv.addKeyword('idExists', {
  async: true,
  type: 'number',
  validate: checkIdExists
});


function checkIdExists(schema, data) {
  return knex(schema.table)
  .select('id')
  .where('id', data)
  .then(function (rows) {
    return !!rows/length; // true if record is found
  });
}

var schema = {
  "$async": true,
  "properties": {
    "userId": {
      "type": "integer",
      "idExists": { "table": "users" }
    },
    "postId": {
      "type": "integer",
      "idExists": { "table": "posts" }
    }
  }
};

var validate = ajv.compile(schema);

validate({ userId: 1, postId: 19 }))
.then(function (valid) {
  // "valid" is always true here
  console.log('Data is valid');
})
.catch(function (err) {
  if (!(err instanceof Ajv.ValidationError)) throw err;
  // data is invalid
  console.log('Validation errors:', err.errors);
});

```

### Using transpilers with asyncronous validation functions.

To use a transpiler you should separately install it (or load its bundle in the browser).

Ajv npm package includes minified browser bundles of regenerator and nodent in dist folder.


#### Using nodent

```javascript
var ajv = Ajv({ /* async: 'es7', */ transpile: 'nodent' });
var validate = ajv.compile(schema); // transpiled es7 async function
validate(data).then(successFunc).catch(errorFunc);
```

`npm install nodent` or use `nodent.min.js` from dist folder of npm package.


#### Using regenerator

```javascript
var ajv = Ajv({ /* async: 'es7', */ transpile: 'regenerator' });
var validate = ajv.compile(schema); // transpiled es7 async function
validate(data).then(successFunc).catch(errorFunc);
```

`npm install regenerator` or use `regenerator.min.js` from dist folder of npm package.


#### Using other transpilers

```javascript
var ajv = Ajv({ async: 'es7', transpile: transpileFunc });
var validate = ajv.compile(schema); // transpiled es7 async function
validate(data).then(successFunc).catch(errorFunc);
```

See [Options](#options).


#### Comparison of async modes

|mode|transpile<br>speed*|run-time<br>speed*|bundle<br>size|
|---|:-:|:-:|:-:|
|generators<br>(native)|-|1.0|-|
|es7.nodent|1.35|1.1|183Kb|
|es7.regenerator|1.0|2.7|322Kb|
|regenerator|1.0|3.2|322Kb|

\* Relative performance in node v.4, smaller is better.

[nodent](https://github.com/MatAtBread/nodent) has several advantages:

- much smaller browser bundle than regenerator
- almost the same performance of generated code as native generators in nodejs and the latest Chrome
- much better performace than native generators in other browsers

[regenerator](https://github.com/facebook/regenerator) is a more widely adopted alternative.


## Filtering data

With [option `removeAdditional`](#options) (added by [andyscott](https://github.com/andyscott)) you can filter data during the validation.

This option modifies original data.

Example:

```javascript
var ajv = Ajv({ removeAdditional: true });
var schema = {
  "additionalProperties": false,
  "properties": {
    "foo": { "type": "number" },
    "bar": {
      "additionalProperties": { "type": "number" },
      "properties": {
        "baz": { "type": "string" }
      }
    }
  }
}

var data = {
  "foo": 0,
  "additional1": 1, // will be removed; `additionalProperties` == false
  "bar": {
    "baz": "abc",
    "additional2": 2 // will NOT be removed; `additionalProperties` != false
  },
}

var validate = ajv.compile(schema);

console.log(validate(data)); // true
console.log(data); // { "foo": 0, "bar": { "baz": "abc", "additional2": 2 }
```

If `removeAdditional` option in the example above were `"all"` then both `additional1` and `additional2` properties would have been removed.

If the option were `"failing"` then property `additional1` would have been removed regardless of its value and property `additional2` would have been removed only if its value were failing the schema in the inner `additionalProperties` (so in the example above it would have stayed because it passes the schema, but any non-number would have been removed).


## Assigning defaults

With [option `useDefaults`](#options) Ajv will assign values from `default` keyword in the schemas of `properties` and `items` (when it is the array of schemas) to the missing properties and items.

This option modifies original data.


Example 1 (`default` in `properties`):

```javascript
var ajv = Ajv({ useDefaults: true });
var schema = {
  "type": "object",
  "properties": {
    "foo": { "type": "number" },
    "bar": { "type": "string", "default": "baz" }
  },
  "required": [ "foo", "bar" ]
};

var data = { "foo": 1 };

var validate = ajv.compile(schema);

console.log(validate(data)); // true
console.log(data); // { "foo": 1, "bar": "baz" }
```

Example 2 (`default` in `items`):

```javascript
var schema = {
  "type": "array",
  "items": [
    { "type": "number" },
    { "type": "string", "default": "foo" }
  ]
}

var data = [ 1 ];

var validate = ajv.compile(schema);

console.log(validate(data)); // true
console.log(data); // [ 1, "foo" ]
```

`default` keywords in other cases are ignored:

- not in `properties` or `items` subschemas
- in schemas inside `anyOf`, `oneOf` and `not` (see #42)
- in `if` subschema of v5 `switch` keyword
- in schemas generated by custom macro keywords


## Coercing data types

When you are validating user inputs all your data properties are usually strings. The option `coerceTypes` allows you to have your data types coerced to the types specified in your schema `type` keywords, both to pass the validation and to use the correctly typed data afterwards.

This option modifies original data.

__Please note__: if you pass a scalar value to the validating function its type will be coerced and it will pass the validation, but the value of the variable you pass won't be updated because scalars are passed by value.


Example:

```javascript
var ajv = Ajv({ coerceTypes: true });
var schema = {
  "type": "object",
  "properties": {
    "foo": { "type": "number" },
    "bar": { "type": "boolean" }
  },
  "required": [ "foo", "bar" ]
};

var data = { "foo": "1", "bar": "false" };

var validate = ajv.compile(schema);

console.log(validate(data)); // true
console.log(data); // { "foo": 1, "bar": false }
```

The coercion rules, as you can see from the example, are different from JavaScript both to validate user input as expected and to have the coercion reversible (to correctly validate cases where different types are defined in subschemas of "anyOf" and other compound keywords).

See [Coercion rules](https://github.com/epoberezkin/ajv/blob/master/COERCION.md) for details.


## API

##### Ajv(Object options) -&gt; Object

Create ajv instance.

All the instance methods below are bound to the instance, so they can be used without the instance.


##### .compile(Object schema) -&gt; Function&lt;Object data&gt;

Generate validating function and cache the compiled schema for future use.

Validating function returns boolean and has properties `errors` with the errors from the last validation (`null` if there were no errors) and `schema` with the reference to the original schema.

Unless the option `validateSchema` is false, the schema will be validated against meta-schema and if schema is invalid the error will be thrown. See [options](#options).


##### .compileAsync(Object schema, Function callback)

Asyncronous version of `compile` method that loads missing remote schemas using asynchronous function in `options.loadSchema`. Callback will always be called with 2 parameters: error (or null) and validating function. Error will be not null in the following cases:

- missing schema can't be loaded (`loadSchema` calls callback with error).
- the schema containing missing reference is loaded, but the reference cannot be resolved.
- schema (or some referenced schema) is invalid.

The function compiles schema and loads the first missing schema multiple times, until all missing schemas are loaded.

See example in [Asynchronous compilation](#asynchronous-compilation).


##### .validate(Object schema|String key|String ref, data) -&gt; Boolean

Validate data using passed schema (it will be compiled and cached).

Instead of the schema you can use the key that was previously passed to `addSchema`, the schema id if it was present in the schema or any previously resolved reference.

Validation errors will be available in the `errors` property of ajv instance (`null` if there were no errors).

__Please note__: every time this method is called the errors are overwritten so you need to copy them to another variable if you want to use them later.

If the schema is asynchronous (has `$async` keyword on the top level) this method returns a Promise. See [Asynchronous validation](#asynchronous-validation).


##### .addSchema(Array&lt;Object&gt;|Object schema [, String key])

Add schema(s) to validator instance. From version 1.0.0 this method does not compile schemas (but it still validates them). Because of that change, dependencies can be added in any order and circular dependencies are supported. It also prevents unnecessary compilation of schemas that are containers for other schemas but not used as a whole.

Array of schemas can be passed (schemas should have ids), the second parameter will be ignored.

Key can be passed that can be used to reference the schema and will be used as the schema id if there is no id inside the schema. If the key is not passed, the schema id will be used as the key.


Once the schema is added, it (and all the references inside it) can be referenced in other schemas and used to validate data.

Although `addSchema` does not compile schemas, explicit compilation is not required - the schema will be compiled when it is used first time.

By default the schema is validated against meta-schema before it is added, and if the schema does not pass validation the exception is thrown. This behaviour is controlled by `validateSchema` option.


##### .addMetaSchema(Object schema [, String key])

Adds meta schema that can be used to validate other schemas. That function should be used instead of `addSchema` because there may be instance options that would compile a meta schema incorrectly (at the moment it is `removeAdditional` option).

There is no need to explicitly add draft 4 meta schema (http://json-schema.org/draft-04/schema and http://json-schema.org/schema) - it is added by default, unless option `meta` is set to `false`. You only need to use it if you have a changed meta-schema that you want to use to validate your schemas. See `validateSchema`.

With option `v5: true` [meta-schema that includes v5 keywords](https://raw.githubusercontent.com/epoberezkin/ajv/master/lib/refs/json-schema-v5.json) also added.


##### <a name="api-validateschema"></a>.validateSchema(Object schema) -&gt; Boolean

Validates schema. This method should be used to validate schemas rather than `validate` due to the inconsistency of `uri` format in JSON-Schema standard.

By default this method is called automatically when the schema is added, so you rarely need to use it directly.

If schema doesn't have `$schema` property it is validated against draft 4 meta-schema (option `meta` should not be false) or against [v5 meta-schema](https://raw.githubusercontent.com/epoberezkin/ajv/master/lib/refs/json-schema-v5.json#) if option `v5` is true.

If schema has `$schema` property then the schema with this id (that should be previously added) is used to validate passed schema.

Errors will be available at `ajv.errors`.


##### .getSchema(String key) -&gt; Function&lt;Object data&gt;

Retrieve compiled schema previously added with `addSchema` by the key passed to `addSchema` or by its full reference (id). Returned validating function has `schema` property with the reference to the original schema.


##### .removeSchema(Object schema|String key|String ref)

Remove added/cached schema. Even if schema is referenced by other schemas it can be safely removed as dependent schemas have local references.

Schema can be removed using key passed to `addSchema`, it's full reference (id) or using actual schema object that will be stable-stringified to remove schema from cache.


##### <a name="api-addformat"></a>.addFormat(String name, String|RegExp|Function|Object format)

Add custom format to validate strings. It can also be used to replace pre-defined formats for ajv instance.

Strings are converted to RegExp.

Function should return validation result as `true` or `false`.

If object is passed it should have properties `validate`, `compare` and `async`:

- _validate_: a string, RegExp or a function as described above.
- _compare_: an optional comparison function that accepts two strings and compares them according to the format meaning. This function is used with keywords `formatMaximum`/`formatMinimum` (from [v5 proposals](https://github.com/json-schema/json-schema/wiki/v5-Proposals) - `v5` option should be used). It should return `1` if the first value is bigger than the second value, `-1` if it is smaller and `0` if it is equal.
- _async_: an optional `true` value if `validate` is an asynchronous function; in this case it should return a promise that resolves with a value `true` or `false`.

Custom formats can be also added via `formats` option.


##### <a name="api-addkeyword"></a>.addKeyword(String keyword, Object definition)

Add custom validation keyword to ajv instance.

Keyword should be a valid JavaScript identifier.

Keyword should be different from all standard JSON schema keywords and different from previously defined keywords. There is no way to redefine keywords or to remove keyword definition from the instance.

Keyword definition is an object with the following properties:

- _type_: optional string or array of strings with data type(s) that the keyword applies to. If not present, the keyword will apply to all types.
- _validate_: validating function
- _compile_: compiling function
- _macro_: macro function
- _inline_: compiling function that returns code (as string)
- _async_: an optional `true` value if the validation function is asynchronous (whether it is compiled or passed in _validate_ property); in this case it should return a promise that resolves with a value `true` or `false`. This option is ignored in case of "macro" and "inline" keywords.

_validate_, _compile_, _macro_ and _inline_ are mutually exclusive, only one should be used at a time.

__Please note__: If the keyword is validating data type that is different from the type(s) in its definition, the validation function will not be called (and expanded macro will not be used), so there is no need to check for data type inside validation function or inside schema returned by macro function (unless you want to enforce a specific type and for some reason do not want to use a separate `type` keyword for that). In the same way as standard keywords work, if the keyword does not apply to the data type being validated, the validation of this keyword will succeed.

See [Defining custom keywords](#defining-custom-keywords) for more details.


##### .errorsText([Array&lt;Object&gt; errors [, Object options]]) -&gt; String

Returns the text with all errors in a String.

Options can have properties `separator` (string used to separate errors, ", " by default) and `dataVar` (the variable name that dataPaths are prefixed with, "data" by default).


## Options

Defaults:

```javascript
{
  // validation and reporting options:
  v5:               false,
  allErrors:        false,
  verbose:          false,
  jsonPointers:     false,
  uniqueItems:      true,
  unicode:          true,
  format:           'fast',
  formats:          {},
  schemas:          {},
  // referenced schema options:
  missingRefs:      true,
  loadSchema:       undefined, // function(uri, cb) { /* ... */ cb(err, schema); },
  // options to modify validated data:
  removeAdditional: false,
  useDefaults:      false,
  coerceTypes:      false,
  // asynchronous validation options:
  async:            undefined,
  transpile:        undefined,
  // advanced options:
  meta:             true,
  validateSchema:   true,
  addUsedSchema:    true,
  inlineRefs:       true,
  passContext:      false,
  loopRequired:     Infinity,
  multipleOfPrecision: false,
  errorDataPath:    'object',
  messages:         true,
  beautify:         false,
  cache:            new Cache
}
```

##### Validation and reporting options

- _v5_: add keywords `switch`, `constant`, `contains`, `patternGroups`, `patternRequired`, `formatMaximum` / `formatMinimum` and `exclusiveFormatMaximum` / `exclusiveFormatMinimum` from [JSON-schema v5 proposals](https://github.com/json-schema/json-schema/wiki/v5-Proposals). With this option added schemas without `$schema` property are validated against [v5 meta-schema](https://raw.githubusercontent.com/epoberezkin/ajv/master/lib/refs/json-schema-v5.json#). `false` by default.
- _allErrors_: check all rules collecting all errors. Default is to return after the first error.
- _verbose_: include the reference to the part of the schema (`schema` and `parentSchema`) and validated data in errors (false by default).
- _jsonPointers_: set `dataPath` propery of errors using [JSON Pointers](https://tools.ietf.org/html/rfc6901) instead of JavaScript property access notation.
- _uniqueItems_: validate `uniqueItems` keyword (true by default).
- _unicode_: calculate correct length of strings with unicode pairs (true by default). Pass `false` to use `.length` of strings that is faster, but gives "incorrect" lengths of strings with unicode pairs - each unicode pair is counted as two characters.
- _format_: formats validation mode ('fast' by default). Pass 'full' for more correct and slow validation or `false` not to validate formats at all. E.g., 25:00:00 and 2015/14/33 will be invalid time and date in 'full' mode but it will be valid in 'fast' mode.
- _formats_: an object with custom formats. Keys and values will be passed to `addFormat` method.
- _schemas_: an array or object of schemas that will be added to the instance. If the order is important, pass array. In this case schemas must have IDs in them. Otherwise the object can be passed - `addSchema(value, key)` will be called for each schema in this object.


##### Referenced schema options

- _missingRefs_: handling of missing referenced schemas. Option values:
  - `true` (default) - if the reference cannot be resolved during compilation the exception is thrown. The thrown error has properties `missingRef` (with hash fragment) and `missingSchema` (without it). Both properties are resolved relative to the current base id (usually schema id, unless it was substituted).
  - `"ignore"` - to log error during compilation and always pass validation.
  - `"fail"` - to log error and successfully compile schema but fail validation if this rule is checked.
- _loadSchema_: asynchronous function that will be used to load remote schemas when the method `compileAsync` is used and some reference is missing (option `missingRefs` should NOT be 'fail' or 'ignore'). This function should accept 2 parameters: remote schema uri and node-style callback. See example in [Asynchronous compilation](#asynchronous-compilation).


##### Options to modify validated data

- _removeAdditional_: remove additional properties - see example in [Filtering data](#filtering-data). This option is not used if schema is added with `addMetaSchema` method. Option values:
  - `false` (default) - not to remove additional properties
  - `"all"` - all additional properties are removed, regardless of `additionalProperties` keyword in schema (and no validation is made for them).
  - `true` - only additional properties with `additionalProperties` keyword equal to `false` are removed.
  - `"failing"` - additional properties that fail schema validation will be removed (where `additionalProperties` keyword is `false` or schema).
- _useDefaults_: replace missing properties and items with the values from corresponding `defaults` keywords. Default behaviour is to ignore `default` keywords. This option is not used if schema is added with `addMetaSchema` method. See example in [Assigning defaults](#assigning-defaults).
- _coerceTypes_: change data type of data to match `type` keyword. See the example in [Coercing data types](#coercing-data-types) and [coercion rules](https://github.com/epoberezkin/ajv/blob/master/COERCION.md).


##### Asynchronous validation options

- _async_: determines how Ajv compiles asynchronous schemas (see [Asynchronous validation](#asynchronous-validation)) to functions. Option values:
  - `"*"` / `"co*"` - compile to generator function ("co*" - wrapped with `co.wrap`). If generators are not supported and you don't provide `transpile` option, the exception will be thrown when Ajv instance is created.
  - `"es7"` - compile to es7 async function. Unless your platform supports them you need to provide `transpile` option. Currently only MS Edge 13 with flag supports es7 async functions according to [compatibility table](http://kangax.github.io/compat-table/es7/)).
  - `true` - if transpile option is not available Ajv will choose the first supported/installed async/transpile modes in this order: "co*" (native generator with co.wrap), "es7"/"nodent", "es7"/"regenerator" during the creation of the Ajv instance. If none of the options is available the exception will be thrown.
  - `undefined`- Ajv will choose the first available async mode in the same way as with `true` option but when the first asynchronous schema is compiled.
- _transpile_: determines whether Ajv transpiles compiled asynchronous validation function. Option values:
  - `"nodent"` - transpile with [nodent](https://github.com/MatAtBread/nodent). If nodent is not installed, the exception will be thrown. nodent can only transpile es7 async functions; it will enforce this mode.
  - `"regenerator"` - transpile with [regenerator](https://github.com/facebook/regenerator). If regenerator is not installed, the exception will be thrown.
  - a function - this function should accept the code of validation function as a string and return transpiled code. This option allows you to use any other transpiler you prefer.


##### Advanced options

- _meta_: add [meta-schema](http://json-schema.org/documentation.html) so it can be used by other schemas (true by default).
- _validateSchema_: validate added/compiled schemas against meta-schema (true by default). `$schema` property in the schema can either be http://json-schema.org/schema or http://json-schema.org/draft-04/schema or absent (draft-4 meta-schema will be used) or can be a reference to the schema previously added with `addMetaSchema` method. Option values:
  - `true` (default) -  if the validation fails, throw the exception.
  - `"log"` - if the validation fails, log error.
  - `false` - skip schema validation.
- _addUsedSchema_: by default methods `compile` and `validate` add schemas to the instance if they have `id` property that doesn't start with "#". If `id` is present and it is not unique the exception will be thrown. Set this option to `false` to skip adding schemas to the instance and the `id` uniqueness check when these methods are used. This option does not affect `addSchema` method.
- _inlineRefs_: Affects compilation of referenced schemas. Option values:
  - `true` (default) - the referenced schemas that don't have refs in them are inlined, regardless of their size - that substantially improves performance at the cost of the bigger size of compiled schema functions.
  - `false` - to not inline referenced schemas (they will be compiled as separate functions).
  - integer number - to limit the maximum number of keywords of the schema that will be inlined.
- _passContext_: pass validation context to custom keyword functions. If this option is `true` and you pass some context to the compiled validation function with `validate.call(context, data)`, the `context` will be available as `this` in your custom keywords. By default `this` is Ajv instance.
- _loopRequired_: by default `required` keyword is compiled into a single expression (or a sequence of statements in `allErrors` mode). In case of a very large number of properties in this keyword it may result in a very big validation function. Pass integer to set the number of properties above which `required` keyword will be validated in a loop - smaller validation function size but also worse performance.
- _multipleOfPrecision_: by default `multipleOf` keyword is validated by comparing the result of division with parseInt() of that result. It works for dividers that are bigger than 1. For small dividers such as 0.01 the result of the division is usually not integer (even when it should be integer, see issue #84). If you need to use fractional dividers set this option to some positive integer N to have `multipleOf` validated using this formula: `Math.abs(Math.round(division) - division) < 1e-N` (it is slower but allows for float arithmetics deviations).
- _errorDataPath_: set `dataPath` to point to 'object' (default) or to 'property' (default behavior in versions before 2.0) when validating keywords `required`, `additionalProperties` and `dependencies`.
- _messages_: Include human-readable messages in errors. `true` by default. `false` can be passed when custom messages are used (e.g. with [ajv-i18n](https://github.com/epoberezkin/ajv-i18n)).
- _beautify_: format the generated function with [js-beautify](https://github.com/beautify-web/js-beautify) (the validating function is generated without line-breaks). `npm install js-beautify` to use this option. `true` or js-beautify options can be passed.
- _cache_: an optional instance of cache to store compiled schemas using stable-stringified schema as a key. For example, set-associative cache [sacjs](https://github.com/epoberezkin/sacjs) can be used. If not passed then a simple hash is used which is good enough for the common use case (a limited number of statically defined schemas). Cache should have methods `put(key, value)`, `get(key)` and `del(key)`.


## Validation errors

In case of validation failure Ajv assigns the array of errors to `.errors` property of validation function (or to `.errors` property of ajv instance in case `validate` or `validateSchema` methods were called). In case of [asynchronous validation](#asynchronous-validation) the returned promise is rejected with the exception of the class `Ajv.ValidationError` that has `.errors` poperty.


### Error objects

Each error is an object with the following properties:

- _keyword_: validation keyword.
- _dataPath_: the path to the part of the data that was validated. By default `dataPath` uses JavaScript property access notation (e.g., `".prop[1].subProp"`). When the option `jsonPointers` is true (see [Options](#options)) `dataPath` will be set using JSON pointer standard (e.g., `"/prop/1/subProp"`).
- _schemaPath_: the path (JSON-pointer as a URI fragment) to the schema of the keyword that failed validation.
- _params_: the object with the additional information about error that can be used to create custom error messages (e.g., using [ajv-i18n](https://github.com/epoberezkin/ajv-i18n) package). See below for parameters set by all keywords.
- _message_: the standard error message (can be excluded with option `messages` set to false).
- _schema_: the schema of the keyword (added with `verbose` option).
- _parentSchema_: the schema containing the keyword (added with `verbose` option)
- _data_: the data validated by the keyword (added with `verbose` option).


### Error parameters

Properties of `params` object in errors depend on the keyword that failed validation.

- `maxItems`, `minItems`, `maxLength`, `minLength`, `maxProperties`, `minProperties` - property `limit` (number, the schema of the keyword).
- `additionalItems` - property `limit` (the maximum number of allowed items in case when `items` keyword is an array of schemas and `additionalItems` is false).
- `additionalProperties` - property `additionalProperty` (the property not used in `properties` and `patternProperties` keywords).
- `patternGroups` (with v5 option) - properties:
  - `pattern`
  - `reason` ("minimum"/"maximum"),
  - `limit` (max/min allowed number of properties matching number)
- `dependencies` - properties:
  - `property` (dependent property),
  - `missingProperty` (required missing dependency - only the first one is reported currently)
  - `deps` (required dependencies, comma separated list as a string),
  - `depsCount` (the number of required dependedncies).
- `format` - property `format` (the schema of the keyword).
- `maximum`, `minimum` - properties:
  - `limit` (number, the schema of the keyword),
  - `exclusive` (boolean, the schema of `exclusiveMaximum` or `exclusiveMinimum`),
  - `comparison` (string, comparison operation to compare the data to the limit, with the data on the left and the limit on the right; can be "<", "<=", ">", ">=")
- `multipleOf` - property `multipleOf` (the schema of the keyword)
- `pattern` - property `pattern` (the schema of the keyword)
- `required` - property `missingProperty` (required property that is missing).
- `patternRequired` (with v5 option) - property `missingPattern` (required pattern that did not match any property).
- `type` - property `type` (required type(s), a string, can be a comma-separated list)
- `uniqueItems` - properties `i` and `j` (indices of duplicate items).
- `$ref` - property `ref` with the referenced schema URI.
- custom keywords (in case keyword definition doesn't create errors) - property `keyword` (the keyword name).


## Some packages using Ajv

- [osprey-method-handler](https://github.com/mulesoft-labs/osprey-method-handler) - Express middleware for validating requests and responses based on a RAML method object, used in [osprey](https://github.com/mulesoft/osprey) - validating API proxy generated from a RAML definition
- [jsoneditor](https://github.com/josdejong/jsoneditor) - A web-based tool to view, edit, format, and validate JSON http://jsoneditoronline.org
- [ripple-lib](https://github.com/ripple/ripple-lib) - A JavaScript API for interacting with [Ripple](https://ripple.com) in Node.js and the browser
- [restbase](https://github.com/wikimedia/restbase) - Distributed storage with REST API & dispatcher for backend services built to provide a low-latency & high-throughput API for Wikipedia / Wikimedia content
- [hippie-swagger](https://github.com/CacheControl/hippie-swagger) - [Hippie](https://github.com/vesln/hippie) wrapper that provides end to end API testing with swagger validation
- [react-form-controlled](https://github.com/seeden/react-form-controlled) - React controlled form components with validation
- [rabbitmq-schema](https://github.com/tjmehta/rabbitmq-schema) - A schema definition module for RabbitMQ graphs and messages
- [@query/schema](https://www.npmjs.com/package/@query/schema) - stream filtering with a URI-safe query syntax parsing to JSON Schema
- [grunt-jsonschema-ajv](https://github.com/SignpostMarv/grunt-jsonschema-ajv) - Grunt plugin for validating files against JSON-Schema


## Command line interface

Simple JSON-schema validation can be done from command line using [ajv-cli](https://github.com/jessedc/ajv-cli) package. At the moment it does not support referenced schemas.


## Tests

```
npm install
git submodule update --init
npm test
```

## Contributing

All validation functions are generated using doT templates in [dot](https://github.com/epoberezkin/ajv/tree/master/lib/dot) folder. Templates are precompiled so doT is not a run-time dependency.

`npm run build` - compiles templates to [dotjs](https://github.com/epoberezkin/ajv/tree/master/lib/dotjs) folder.

`npm run watch` - automatically compiles templates when files in dot folder change


## Changes history

See https://github.com/epoberezkin/ajv/releases

__Please note__: [Changes in version 3.0.0](https://github.com/epoberezkin/ajv/releases/tag/3.0.0).


## License

[MIT](https://github.com/epoberezkin/ajv/blob/master/LICENSE)
