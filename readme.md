# TypeScript Tricks

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
