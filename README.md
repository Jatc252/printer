# Printer [![NPM version][NPMIMGURL]][NPMURL]

[NPMIMGURL]: https://img.shields.io/npm/v/@putout/printer.svg?style=flat&longCache=true
[NPMURL]: https://npmjs.org/package/@putout/printer "npm"

Prints [**Babel AST**](https://github.com/coderaiser/estree-to-babel) to readable **JavaScript**.

- ☝️ Similar to **Recast**, but simpler and easier in maintenance, since it supports only **Babel**.
- ☝️ As opinionated as **Prettier**, but has more user-friendly output and works directly with **AST**.
- ☝️ Like **ESLint** but works directly with **Babel AST**.
- ☝️ Easily extandable with help of [Overrides](h#overrides).

Supports:
- ✅ **ES2023**;
- ✅ **JSX**;
- ✅ **TypeScript**;

## Install

```
npm i @putout/printer
```

## API

```js
const {print} = require('@putout/printer');
const {parse} = require('@babel/parser');
const ast = parse('const a = (b, c) => {const d = 5; return a;}');

print(ast);

// returns
`
const a = (b, c) => {
    const d = 5;
    return a;
};
`;
```

## Overrides

When you need to extend syntax of `@putout/printer` just pass a function which receives:

- `path`, Babel Path
- `print`, a function to output result of printing into token array;

When `path` contains to dashes `__` and name, it is the same as: `print(path.get('right'))`, and this is
actually `traverse(path.get('right'))` shortened to simplify read and process.

Here is how you can override `AssignmentPattern`:

```js
const ast = parse('const {a = 5} = b');

print(ast, {
    format: {
        indent: '    ',
    },
    visitors: {
        AssignmentPattern(path, {print}) {
            print(' /* [hello world] */= ');
            print('__right');
        },
    },
});

// returns
'const {a /* [hello world] */= 5} = b;\n';
```

### `print`

Used in previous example `print` can be used for a couple purposes:
- to print `string`;
- to print `node` when `object` passed;
- to print `node` when `string` started with `__`;

```js
print(ast, {
    visitors: {
        AssignmentPattern(path, {print, maybe}) {
            maybe.print.newline(path.parentPath.isCallExpression());
            print(' /* [hello world] */= ');
            print('__right');
        },
    },
});
```

### `maybe`

When you need some condition use `maybe`. For example, to add newline only when parent node is `CallExpression` you
can use `maybe.print.newline(condition)`:

```js
print(ast, {
    visitors: {
        AssignmentPattern(path, {print, maybe}) {
            maybe.print.newline(path.parentPath.isCallExpression());
            print(' /* [hello world] */= ');
            print('__right');
        },
    },
});
```

### `write`

When are you going to output string you can use low-level function `write`:

```js
print(ast, {
    visitors: {
        BlockStatement(path, {write}) {
            write('hello')
        },
    },
});
```


### `indent`

When you need to add indentation use `indent`, for example when you output body,
you need to increment indentation, and then decrement it back:

```js
print(ast, {
    visitors: {
        BlockStatement(path, {write, indent}) {
            write('{')
            indent.inc();
            indent();
            write('some;')
            indent.dec();
            write('{');
        },
    },
});
```

### `traverse`

When are you needing to traverse node, you can use `traverse`:

```js
print(ast, {
    visitors: {
        AssignmentExpression(path, {traverse}) {
            traverse(path.get('left'));
        },
    },
});
```

This is the same as `print('__left')` but more low-level, and supports only objects.

## License

MIT
