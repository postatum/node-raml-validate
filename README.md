# RAML Validate

[![NPM version][npm-image]][npm-url]
[![Build status][travis-image]][travis-url]
[![Test coverage][coveralls-image]][coveralls-url]

Strict and pluginable validation of [RAML parameters](https://github.com/raml-org/raml-spec/blob/master/raml-0.8.md#named-parameters).

## Installation

```shell
npm install raml-validate --save
```

## Usage

You must require the module and call it as a function to get a validation instance back.

```javascript
var validate = require('raml-validate')();

// Create a user model schema.
var user = validate({
  username: {
    type: 'string',
    minLength: 5,
    maxLength: 50,
    required: true
  },
  password: {
    type: 'string',
    minLength: 5,
    maxLength: 50,
    required: true
  }
});

// Validate a user model.
user({
  username: 'blakeembrey',
  password: 'super secret password'
}); //=> { valid: true, errors: [] }
```

**Module does not currently support [wild-card parameters](https://github.com/raml-org/raml-spec/blob/master/raml-0.8.md#headers)**

### Getting validation errors

All validation errors can be retrieved from the `errors` property on the returned object. If `valid === false`, the errors will be set to an array. This can be useful for generating error messages for the client.

```javascript
[
  {
    valid: false,
    key: 'password',
    value: 'test',
    rule: 'minLength',
    attr: 5
  }
]
```

### Required validation

If the validation does not set `required` to be true, a `null` or `undefined` value will be valid.

### Repeated validation

The module has core support for repeated properties in the form of an array. If the validation is set to `repeat`, but does not receive an array - validation will fail with a `repeat` error.

### Multiple types

The module supports multiple types according to the [RAML spec](https://github.com/raml-org/raml-spec/blob/master/raml-0.8.md#named-parameters-with-multiple-types). When multiple types are specified, it'll run the validation against the matching type.

```javascript
validate({
  file: [{
    type: 'string'
  }, {
    type: 'file'
  }]
});
```

If any of the types are set to `repeat`, it'll only run that validation object when every value in the array is of the correct type - otherwise you will receive a type error.

### Adding new types

New type validations can be added by setting the corresponding property on the `validate.TYPES` object. For example, adding file validation to support buffers can be added by doing:

```javascript
validate.TYPES.file = function (value) {
  return Buffer.isBuffer(value);
};
```

The function must accept the value as the first parameter and return a boolean depending on success or failure.

### Adding new rules

New rules can be added by setting the corresponding property on the `validate.RULES` object. For example, to add file size support we can do the following:

```javascript
validate.RULES.minFileSize = function (size) {
  return function (value) {
    return value.length <= size;
  };
};
```

The function must accept the validation value as its only parameter and is expected to return another function that implements the validation logic. The returned function must accept the value as the first argument, and can optionally accept the key and model as the second and third arguments. This is useful for implementing a rule such as `requires`, where both parameters may be optional; however, when set, depend on eachother being set.

```javascript
validate.RULES.requires = function (property) {
  return function (value, key, object) {
    return value != null && object[property] != null;
  };
};
```

## License

Apache 2.0

[npm-image]: https://img.shields.io/npm/v/raml-validate.svg?style=flat
[npm-url]: https://npmjs.org/package/raml-validate
[travis-image]: https://img.shields.io/travis/mulesoft/node-raml-validate.svg?style=flat
[travis-url]: https://travis-ci.org/mulesoft/node-raml-validate
[coveralls-image]: https://img.shields.io/coveralls/mulesoft/node-raml-validate.svg?style=flat
[coveralls-url]: https://coveralls.io/r/mulesoft/node-raml-validate?branch=master
