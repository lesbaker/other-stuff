# Culebra Cheatsheet

Personal cheatsheet for the Culebra programming language

## Data structures

### Array

#### Mutating operations (changes array in-place)

| Signature                           | Description                           |
|-------------------------------------|---------------------------------------|
| `a.push(x: Any) -> Nil`             | Append `x` to the end.                |
| `a.pop() -> Any`                    | Remove and return the last element. `nil` if empty. |
| `a.extend(other: Array) -> Nil`     | Append every element of `other`. `a.extend(a)` appends the elements `a` had on entry. For `Tuple` / `Set` sources use spread (§9). |
| `a.insert(i: Long, x: Any) -> Nil`  | Insert `x` at index `i`, such that `0 <= i <= size`. Elements at `i + 1` shift right one. Negative indices count from the end, like `a[i]`; Inserting at index `size` same as `push()`. Out of range indices raise `IndexError`. |
| `a.remove_at(i: Long) -> Any`       | Remove the element at `i` such that `0 <= i < size`. Elements at `i + 1` shift left one. Negative indices count from the end. Out of range indices raise `IndexError`. Returns the removed element. |
| `a.reverse() -> Nil`                | Reverse in place.                     |
| `a.sort(reverse: Bool = false) -> Nil` | Stable-sort in natural order; elements compare by the same rule as `<`, so an Object's `__lt__` / `cmp` is honored (a `Path` array sorts) and incomparable elements throw (leaving the array as it was). Keyword-only `reverse: true` sorts descending (still stable). |
| `a.sort_by(key: Function, reverse: Bool = false) -> Nil` | Stable-sort in place using `key(x)` as the comparison key (ascending). `key` must take one parameter; the keys compare by the same rule as `<`, so a key can be a number, a String, a Tuple (to order by several things at once: `\|n\| (n.bar, n.beat)`) or an Object with `cmp` / `__lt__`. Keyword-only `reverse: true` sorts descending (still stable). |

#### Pure operations (contents remain same)

| Signature                                   | Description                           |
|---------------------------------------------|---------------------------------------|
| `a.size() -> Long`                          | Number of elements.                   |
| `a.empty() -> Bool`                         | Whether `size() == 0`.                |
| `a.presence() -> Array \| Nil`              | `a` unchanged (same Array, not a copy) if non-empty, else `nil` — pairs with `??`/`?.` for "use it if there's anything there" (`xs.presence()?.join(",") ?? default`). |
| `a.get(i: Long, fallback: Any) -> Any`      | The element at `i` (negative counts from the end, like `a[i]`), or `fallback` if out of range. Read-only, never throws. |
| `a.slice(start: Long, end: Long) -> Array`  | Shallow subarray `[start, end)`. Same clamping as `String.slice`. |
| `a.join(sep: String) -> String`             | Concatenate elements via `to_string` (strings unquoted), separated by `sep`. |
| `a.contains(v: Any) -> Bool`                | Whether `v == elem` for some element — by `==`'s own rule, an element's `__eq__` / `eq` included. |
| `a.index_of(v: Any) -> Long`                | Index of first equal element, else `-1`. |
| `a.map(f: Function) -> Array`               | New array of `f(x)` for each element. `f` must take one parameter. |
| `a.filter(f: Function) -> Array`            | New array of elements for which `f(x)` is truthy. `f` must take one parameter. |
| `a.for_each(f: Function) -> Nil`            | Call `f(x)` for each element for side effects. `f` must take one parameter. |
| `a.reduce(init: Any, f: Function) -> Any`   | Fold: start with `init`, apply `acc = f(acc, x)` for each element, return final `acc`. `f` must take two parameters. |
| `a.find(f: Function) -> Any`                | First element for which `f(x)` is truthy, else `nil`. `f` must take one parameter. |
| `a.any(f: Function) -> Bool`                | `true` if `f(x)` is truthy for any element, else `false`. `f` must take one parameter. |
| `a.all(f: Function) -> Bool`                | `true` if `f(x)` is truthy for every element (or if empty), else `false`. `f` must take one parameter. |
| `a.flat_map(f: Function) -> Array`          | Concatenate `f(x)` for each element; each `f(x)` must be an `Array`. `f` must take one parameter. |
| `a.sum() -> Long \| Float`                  | Sum of all elements, each `Long` or `Float`. Stays `Long` while every element is a `Long`, becomes `Float` once any element is one. Empty → `0`. |
| `a.product() -> Long \| Float`              | Product of all elements, promoting like `sum`. Empty → `1`. |
| `a.min() -> Any`                            | Smallest element. Elements compare by the same rule as `<` (§5), so numbers, Strings, Tuples and Objects with `cmp` / `__lt__` all work, and a pair `<` refuses throws. The element itself is returned, so its own type survives, and ties keep the earlier element. Throws on empty. |
| `a.max() -> Any`                            | Largest element. Same rule as `min`. |
| `a.min_by(f: Function) -> Any`              | Element whose key `f(x)` is smallest; `f` must take one parameter, and the keys compare by the same rule as `<` — a Tuple key orders by several things at once. Each key is computed once, and ties keep the earlier element. Throws on empty. |
| `a.max_by(f: Function) -> Any`              | Element whose key `f(x)` is largest. Same rules as `min_by`. |
| `a.to_set() -> Set`                         | Fresh `Set` of the elements in first-seen order, duplicates dropped. This is how a `Set` is built from a collection; `Set(a, b)` builds one from values. Unhashable elements throw. |
| `a.to_object() -> Object`                   | Fresh `Object` from `(key, value)` tuples — the inverse of `Object.iter()`, so a table can be built as an expression instead of a `mut` + loop. Keys keep first-seen order and a repeat overwrites in place (last value, first position). Entries are **mutable**, like `group_by`'s and any other key inserted at runtime. An element that is not a 2-tuple raises `TypeError`; an unhashable key throws like any other. |
| `a.group_by(f: Function) -> Object`         | Buckets elements into Arrays keyed by `f(x)`, in first-seen key order; `f` must take one parameter and return a hashable key. |
| `a.partition(p: Function) -> Tuple`         | One-pass split into `(matching, non_matching)`, order preserved in both halves. `p` must take one parameter. Destructures: `let (yes, no) = xs.partition(p)`. |
| `a.unzip() -> Tuple`                        | Split `(a, b)` pairs into `(Array, Array)` — the inverse of `zip`. Each element must be a 2-element `Tuple` or a `{first, second}` Object (either pair spelling is accepted); anything else raises `TypeError`. Destructures: `let (xs, ys) = pairs.unzip()`. |
| `a.sorted(reverse: Bool = false) -> Array` | Like `sort` but returns a new sorted Array, leaving the receiver unchanged — so it chains (`xs.sorted().join(",")`). `reverse: true` for stable descending. |
| `a.sorted_by(key: Function, reverse: Bool = false) -> Array` | Like `sort_by` but returns a new sorted Array, leaving the receiver unchanged — so it chains (`xs.sorted_by(f).join(",")`). `reverse: true` for stable descending. |

### Deque
| Member | Returns |
|---|---|
| `d.push_back(x)` | `Nil` |
| `d.push_front(x)` | `Nil` |
| `d.pop_back()` | `Any` — removed element, `nil` if empty |
| `d.pop_front()` | `Any` — removed element, `nil` if empty |
| `d.peek_back()` | `Any` — `nil` if empty, does not remove |
| `d.peek_front()` | `Any` — `nil` if empty, does not remove |
| `d.size()` | `Long` |
| `d.empty()` | `Bool` |
| `d.to_array()` | `Array` — snapshot, front to back |

