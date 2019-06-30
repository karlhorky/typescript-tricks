# TypeScript Tricks

## `DeepImmutable` aka `DeepReadonly`

Deep immutable (readonly) type for specifying multi-level data structures that cannot be modified.

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
