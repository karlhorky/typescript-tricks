# TypeScript Tricks

A collection of useful TypeScript tricks

## `DeepImmutable` aka `DeepReadonly` Generic

Deep immutable (readonly) generic type for specifying multi-level data structures that cannot be modified.

**Example:**

```ts
let deepX: DeepImmutable<{y: {a: number}}> = {y: {a: 1}};
deepX.y.a = 2; // Fails as expected!
```

**Credit:** [@nieltg](https://github.com/nieltg) in [Microsoft/TypeScript#13923 (comment)](https://github.com/Microsoft/TypeScript/issues/13923#issuecomment-402901005)

```ts
type Primitive = undefined | null | boolean | string | number | Function

type Immutable<T> =
  T extends Primitive ? T :
    T extends Array<infer U> ? ReadonlyArray<U> :
      T extends Map<infer K, infer V> ? ReadonlyMap<K, V> : Readonly<T>

type DeepImmutable<T> =
  T extends Primitive ? T :
    T extends Array<infer U> ? DeepImmutableArray<U> :
      T extends Map<infer K, infer V> ? DeepImmutableMap<K, V> : DeepImmutableObject<T>

interface DeepImmutableArray<T> extends ReadonlyArray<DeepImmutable<T>> {}
interface DeepImmutableMap<K, V> extends ReadonlyMap<DeepImmutable<K>, DeepImmutable<V>> {}
type DeepImmutableObject<T> = {
  readonly [K in keyof T]: DeepImmutable<T[K]>
}
```

## Empty Object Type

To verify that an object has no keys, use `Record<string, never>`:

```ts
type EmptyObject = Record<string, never>;

const a: EmptyObject = {}; // ✅
const b: EmptyObject = { z : 'z' }; // ❌ Type 'string' is not assignable to type 'never'
```

[Playground](https://www.typescriptlang.org/play?#code/C4TwDgpgBAogtmUB5ARgKwgY2FAvFAJSwHsAnAEwB4BnYUgSwDsBzAGikYgDcJSA+ANwAoIZmKNaUAIYAuWAmTosOfAG8AvgKgB6bVECg5KPGSUc+IhCoM2PFFVQAXlDkByBy6iadewDLkQA)

## `JSON.stringify()` an Object with Regular Expression Values

`JSON.stringify()` on an object with regular expressions as values will behave in an unual way:

```ts
JSON.stringify({
  name: 'update',
  urlRegex: /^\/cohorts\/[^/]+$/,
})
// '{"name":"update","urlRegex":{}}'
```

Use a custom replacer function to call `.toString()` on the RegExp:

```ts
export function stringifyObjectWithRegexValues(obj: Record<string, unknown>) {
  return JSON.stringify(obj, (key, value) => {
    if (value instanceof RegExp) {
      return value.toString();
    }
    return value;
  });
}
```

This will return a visible representation of the regular expression:

```ts
stringifyObjectWithRegexValues({
  name: 'update',
  urlRegex: /^\/cohorts\/[^/]+$/,
})
// '{"name":"update","urlRegex":"/^\\\\/cohorts\\\\/[^/]+$/"}'
```

## `Opaque` Generic

A generic type that allows for checking based on the name of the type ("opaque" type checking) as opposed to the data type ("transparent", the default in TypeScript).

**Example:**

```ts
type Username = Opaque<"Username", string>;
type Password = Opaque<"Password", string>;

function createUser(username: Username, password: Password) {}
const getUsername = () => getFormInput('username') as Username;
const getPassword = () => getFormInput('password') as Password;

createUser(
  getUsername(),
  getUsername(),  // Error: Argument of type 'Opaque<"Username", string>' is not assignable to
                  // parameter of type 'Opaque<"Password", string>'.
);
```

**Credit:**

- [@stereobooster](https://twitter.com/stereobooster) in [Pragmatic types: opaque types and how they could have saved Mars Climate Orbiter](https://dev.to/stereobooster/pragmatic-types-opaque-types-and-how-they-could-have-saved-mars-climate-orbiter-1551)
- [@phpnode](https://twitter.com/phpnode) in [Stronger JavaScript with Opaque Types](https://codemix.com/opaque-types-in-javascript/)

```ts
type Opaque<K, T> = T & { __TYPE__: K };
```

## `Prettify` Generic

A generic type that shows the final "resolved" type without indirection or abstraction.

```ts
const users = [
  { id: 1, name: "Jane" },
  { id: 2, name: "John" },
] as const;

type User = (typeof users)[number];

type LiteralToBase<T> = T extends string
  ? string
  : T extends number
  ? number
  : T extends boolean
  ? boolean
  : T extends null
  ? null
  : T extends undefined
  ? undefined
  : T extends bigint
  ? bigint
  : T extends symbol
  ? symbol
  : T extends object
  ? object
  : never;

type Widen<T> = {
  [K in keyof T]: T[K] extends infer U ? LiteralToBase<U> : never;
};

export type Prettify<Type> = Type extends {}
  ? Type extends infer Obj
    ? Type extends Date
      ? Date
      : { [Key in keyof Obj]: Prettify<Obj[Key]> } & {}
    : never
  : Type;

type WideUser = Widen<User>;
//   ^? Widen<{ readonly id: 1; readonly name: "Jane"; }> | Widen<{ readonly id: 2; readonly name: "John"; }>

type PrettyWideUser = Prettify<Widen<User>>;
//   ^? { readonly id: number; readonly name: string; } | { readonly id: number; readonly name: string; }
```

**Credit:**

- [Matt Pocock](https://twitter.com/mattpocockuk) in [`Prettify` type helper tweet](https://twitter.com/mattpocockuk/status/1622730173446557697)
- [Aaron](https://twitter.com/nowlena) in [`Prettify` type helper tweet reply](https://twitter.com/nowlena/status/1622967286188630020)

```ts
export type Prettify<Type> = Type extends {}
  ? Type extends infer Obj
    ? Type extends Date
      ? Date
      : { [Key in keyof Obj]: Prettify<Obj[Key]> } & {}
    : never
  : Type;
```

## `Spread` Generic

A generic type that allows for [more soundness](https://github.com/microsoft/TypeScript/pull/28553#issuecomment-440004598) while using object spreads and `Object.assign`.

```ts
type A = {
    a: boolean;
    b: number;
    c: string;
};

type B = {
    b: number[];
    c: string[] | undefined;
    d: string;
    e: number | undefined;
};

type AB = Spread<A, B>;

// type AB = {
//    a: boolean;
//    b: number[];
//    c: string | string[];
//    d: string;
//    e: number | undefined;
//};
```

**Credit:**

- [@ahejlsberg](https://github.com/ahejlsberg) in [Microsoft/TypeScript#21316 (comment)](https://github.com/Microsoft/TypeScript/pull/21316#issuecomment-359574388)

```ts
type Diff<T, U> = T extends U ? never : T;  // Remove types from T that are assignable to U

// Names of properties in T with types that include undefined
type OptionalPropertyNames<T> =
    { [K in keyof T]: undefined extends T[K] ? K : never }[keyof T];

// Common properties from L and R with undefined in R[K] replaced by type in L[K]
type SpreadProperties<L, R, K extends keyof L & keyof R> =
    { [P in K]: L[P] | Diff<R[P], undefined> };

// Type of { ...L, ...R }
type Spread<L, R> =
    // Properties in L that don't exist in R
    & Pick<L, Diff<keyof L, keyof R>>
    // Properties in R with types that exclude undefined
    & Pick<R, Diff<keyof R, OptionalPropertyNames<R>>>
    // Properties in R, with types that include undefined, that don't exist in L
    & Pick<R, Diff<OptionalPropertyNames<R>, keyof L>>
    // Properties in R, with types that include undefined, that exist in L
    & SpreadProperties<L, R, OptionalPropertyNames<R> & keyof L>;
```

## Related

For higher quality utility types, you may have better luck with:

- https://github.com/millsp/ts-toolbelt
- https://github.com/gcanti/typelevel-ts
- https://github.com/pelotom/type-zoo
- https://github.com/kgtkr/typepark
- https://github.com/tycho01/typical
- https://github.com/piotrwitek/utility-types
