# Cepha

An experimental templating library designed specifically for generating source code.

It's a JavaScript library, but it's designed especially to be nice to use with [Civet](https://github.com/danielx/civet).

## Example

```js
import { cepha } from "cepha"

// You have some data
const model = "User"
const fields = [
  { name: "name",    type: "string",  defaultValue: "Unnamed" },
  { name: "isAdmin", type: "boolean", defaultValue: true },
]

// Generate some code from that data
modelDefinition := cepha().string (c) =>
  c`class ${model} extends Model {` => // the opened "{" gets automatically closed
    c`id: number` // indentation is maintained according to nesting

    for { name, type, defaultValue } of fields
      c`override get ${name}(): ${type} {` =>
        c`return super.${name} ?? ${JSON.stringify defaultValue}`

console.log modelDefinition
```

```typescript
class User extends Model {
  id: number
  override get name(): string {
    return super.name ?? "Unnamed"
  }
  override get isAdmin(): boolean {
    return super.isAdmin ?? true
  }
}
```

## Features

### Emit code imperatively

```typescript
cepha().string (c) =>

  c`squaring numbers:`

  for n of [1..5]
    c`the square of ${n} is ${n**2}`
```

```
squaring numbers:
the square of 1 is 1
the square of 2 is 4
the square of 3 is 9
the square of 4 is 16
the square of 5 is 25
```

### Nested blocks, indentation-aware, automatic closing of brackets
Block can be nested and the indentation is maintained.

When nesting blocks, any opening brackets at the end of the parent block's string will be closed.

```typescript
cepha().string (c) =>
  c`class User extends Model {` =>
    c`id: number`
    c`constructor(` =>
      c`readonly name,`
      c`readonly isAdmin,`
    c`{` =>
      c`super()`
```

```typescript
class User extends Model {
  id: number
  constructor(
    readonly name,
    readonly isAdmin,
  )
  {
    super()
  }
}
```

### Deferred code blocks

Use the `defer` helper to defer code block evaluation until after the rest of the block have been evaluated.

When a deferred block is finally evaluated, it's contents are inserted where the deferred block was.

```typescript
models := {
  User:    { parentClass: "Model" },
  Post:    { parentClass: "Model" },
  Comment: { parentClass: "Model" },
  Layout:  { parentClass: "CmsObject" },
}

cepha().string (c, { defer }) =>

  imports := new Set<string>() // add items to this as we go
  defer => // this gets evaluated after the rest of the document, so `imports` will be populated
    for importName of imports
      c`import { ${importName} } from './${importName}'`

  c``
  for [modelName, { parentClass }] of Object.entries(models)
    imports.add parentClass
    c`class ${modelName} extends ${parentClass} {}`
```

```typescript
import { Model } from './Model'
import { CmsObject } from './CmsObject'

class User extends Model {}
class Post extends Model {}
class Comment extends Model {}
class Layout extends CmsObject {}
```
