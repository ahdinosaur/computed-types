<h1 align="center">FunVal</h1>

<h4 align="center">A minimalist library for data validation using plain functions ❤️.</h4>

<p align="center">
<a href="https://www.npmjs.org/package/funval"><img src="http://img.shields.io/npm/v/funval.svg" alt="View On NPM"></a>
<a href="https://travis-ci.org/neuledge/funval"><img src="https://travis-ci.org/neuledge/funval.svg?branch=master" alt="Build Status"></a>
<a href="https://david-dm.org/neuledge/funval"><img src="https://david-dm.org/neuledge/funval.svg" alt="Dependency Status"></a>
<a href="LICENSE"><img src="https://img.shields.io/npm/l/funval.svg" alt="License"></a>
</p>
<br>

```ts
declare function Validator<T, I>(input: I): T | Promise<T>;
```

**FunVal** is a minimalist validation library that seamlessly integrates with your existing
TypeScript schemas. Using only pure functions, **FunVal** knows how to validate your data and
automatically generates TypeScript interfaces to reduce code duplications and complexity.

### Main Features

- **Asynchronous & Synchronous Support** - Automaticly detected by the validators type.
- **Pure Javascript** - Works also without TypeScript.
- **Seamless Interfaces** - Create new validator using plain functions in seconds.
- **Function Composition** - Pipe multiple validators to generate new ones.
- **TypeScript Validation** - Detect errors during compile time. 

### Install

```bash
npm i funval
```

### Usage

```ts
import { Validate, Optional, Or, NonEmptyString, StringRange, Type } from 'funval';
import compose from 'compose-function';

const UserSchema = {
  name: Optional(String),
  username: compose(StringRange(3, 20), NonEmptyString),
  status: Or('active' as 'active', 'suspended' as 'suspended'),
  amount: Number,
};

const validator = Validate(UserSchema);

let user: Type<typeof UserSchema>;

try {
  user = validator({
    username: 'john1',
    // @ts-ignore TS2322: Type '"unregistered"' is not assignable to type '"active" | "suspended"'.
    status: 'unregistered',
    amount: 20.3,
  });
} catch (err) {
  console.error(err.message, err.paths);
  
  // Prints:
  // Expect value to equals "suspended" (given: "unregistered") [
  //   {
  //     path: [ 'status' ],
  //     message: 'Expect value to equals "suspended" (given: "unregistered")'
  //   }
  // ]
}
```


### Creating Validators

A validator is any function that can return a value. For example, here is an email address validator:

```ts
import * as EmailValidator from 'email-validator';

function Email(input: string): string {
  if (!EmailValidator.validate(input)) {
    throw new TypeError(`The given email "${input}" is invalid`);
  }

  return input;
}
```

You can use the `Email` validator on schemas by using:

```ts
const UserSchema = {
  email: Email,
};

const validator = Validate(UserSchema);
```

#### Asynchronous Validators

Asynchronous validators are supported by resolving to a `Promise` value:

```ts
import fetch from 'node-fetch';

async function AvailableUsername(input: string): Promise<string> {
  const res = await fetch(`/check-username?username=${encodeURIComponent(input)}`);

  if (!res.ok) {
    throw new TypeError(`Username "${input}" is already taken`);
  }

  return input;
}
```

**FunVal** automatically detects promise values and convert the return type of the `Validator` to promise as well: 
```ts
const UserSchema = {
  username: AvailableUsername,
};

const validator = await Validate(UserSchema);
```


### License

&copy; 2020 Moshe Simantov

Licensed under the [MIT](LICENSE).
