# type-flag <a href="https://npm.im/type-flag"><img src="https://badgen.net/npm/v/type-flag"></a> <a href="https://npm.im/type-flag"><img src="https://badgen.net/npm/dm/type-flag"></a> <a href="https://packagephobia.now.sh/result?p=type-flag"><img src="https://packagephobia.now.sh/badge?p=type-flag"></a>

Parse CLI argv flags with first-class type support. Think [minimist](https://github.com/substack/minimist), but typed!

### Features
- ⚡️ Typed flags
- 🎨 Custom types
- 🎭 Alias support
- 🐣 Tiny

<sub>Support this project by ⭐️ starring and sharing it. [Follow me](https://github.com/privatenumber) to see what other cool projects I'm working on! ❤️</sub>

## 🚀 Install

```bash
npm i type-flag
```

## 🚦 Quick usage

### Setup

_cli.ts_
```ts
import typeFlag from 'type-flag'

const parsed = typeFlag(process.argv.slice(2), {

    // Define flags here...

    someString: String,

    someNumber: {
        type: Number,
        alias: 'n' // Set an alias (eg. -n <number>)
    },

    someBoolean: {
        type: Boolean,
        alias: 'b'
    }
})
```

`parsed` resolves to the following type:
```ts
{
    flags: {
        someString: string[];
        someNumber: boolean[];
        someBoolean: number[];
    };
    unknownFlags: {
        [flagName: string]: (string | boolean)[];
    };
    _: string[];
}
```

### Usage

#### kebab-case flags mapped to camelCase
When passing in the flags, they can be in kebab-case and will automatically map to the camelCase equivalent.
```sh
$ node ./cli --someString hello --some-string world
```

#### Unknown flags
When unrecognized flags are passed in, they are either interpreted as a string or boolean depending on usage.

Note, unknown flags are not converted to camelCase.

```sh
$ node ./cli --unknown-flag --unknown-flag 2
```

This outputs the following:
```ts
{
	unknownFlags: {
		'unknown-flag': [true, '2']
	},
	...
}
```

#### Arguments
All argument values are passed into the `_` property.

Notice how the "`value"` is parsed as an argument because the boolean flag doesn't accept a value.

Arguments after `--` will not be parsed.

```sh
$ node ./cli --boolean value --string "hello world" "another value" -- --string "goodbye world"
```

This outputs the following:
```ts
{
	_: ['value', 'another value', '--string', 'goodbye world']
	...
}
```

## 👨🏻‍🏫 Examples

### Using a custom type
Basic types can be set using [built-in functions in JavaScript](https://www.w3schools.com/js/js_object_constructors.asp#:~:text=Built-in%20JavaScript%20Constructors), but sometimes you want to a new type, narrow the type, or add validation.

To create a new type, simply declare a function that accepts a string argument and returns the parsed value with the expected type.

In this example, the `format` flag is enforced to be either `cjs` or `esm`.
```ts
const possibleJsFormats = ['cjs', 'esm'] as const;

type JsFormats = typeof possibleJsFormats[number];

function JsFormat(format: JsFormats) {
    if (!possibleJsFormats.includes(format)) {
        throw new Error(`Invalid format: "${format}"`);
    }

    return format;
};

const parsed = typeFlag(process.argv.slice(2), {
    format: JsFormat
})
```

`parsed` resolves to the following type:
```ts
const parsed: {
    flags: {
        format: JsFormats[];
    };
    ...
}
```

### Inverting a boolean
To invert a boolean flag, `false` must be passed in with the `=` operator.

```sh
$ node ./cli --boolean-flag=false
```

## ⚙️ API

### typeFlag(argv, flagSchema)

Returns an object with the shape:
```ts
{
    flags: {
        [flagName: string]: InferredType[];
    };
    unknownFlags: {
        [flagName: string]: (string | boolean)[];
    };
    _: string[];
}
```

#### argv
Type: `string[]`

The argv array to parse. Pass in `process.argv.slice(2)`.


#### flagSchema
Type:
```ts
type TypeFunction = (argvValue: any) => any;

type FlagSchema = {
    [flagName: string]: TypeFunction | {
        type?: TypeFunction; // defaults to String
        alias?: string;
    };
};
```


An object containing flag schema definitions. Where the key is the flag name, and the value is either the type function or an object containing the type function and/or alias.

## 🙋‍♀️ FAQ

### Why do I have to manually pass in `process.argv`?
Few reasons:
- Type-flag designed to be an utility tool that can be extended in other CLI tools
- To mock the simplicity & clarity of [minimist](https://github.com/substack/minimist)
- Explicitly passing it in makes it easier to intercept for pre-processing and also testing

### Why are the parsed flags in an array?
To minimize the scope of the library to parsing & types.

This way, the user can choose whether to accept an array of multiple values, or to pop the last element as a single value.

Support for validation, required flags, default values, etc. should be added in user-land.

### How is it different from minimist?

- Typed schema

	minimist assumes usage is correct and infers flag types based on how the flags are used.

	type-flag takes in flag schemas to determine how to parse flags.

	For example, given a string and boolean flag with no values passed in:
	```
	$ cli --string --boolean value
	```

	minimist unconfigured would interpret `--string` as a boolean, and `--boolean` as a string. The `string` and `boolean` options would need to specify the appropriate flags, but they would not be automatically typed.
	
	type-flag would interpret `--string` with an empty string passed in, `--boolean` to be `true`, and `value` to be passed in as an argument (in `--`).

- Combined aliases

	It's pretty common in CLIs to to combine aliases into one flag (eg. `-a -v -z` → `-avz`).
	
	This is supported in both minimist and type-flag, however, type-flag interprets the following differently: `-n9`

	minimist interprets that as `-n 9` or `-n=9`. But type-flag considers that `-9` may refer to a flag and interprets it as `-n -9`. A real example of this flag is in [`gzip`](https://linux.die.net/man/1/gzip#:~:text=-9), where `-9` is an alias for "best compression".
