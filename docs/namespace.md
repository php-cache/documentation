# Namespaced and prefixed PSR-6 cache pools

`NamespacedCachePool` isolates one logical cache inside any PSR-6 pool. It prevents key collisions and limits `clear()` to one namespace.

## Installation

```bash
composer require cache/namespaced-cache:^2.0
```

## Usage

```php
use Cache\Adapter\PHPArray\ArrayCachePool;
use Cache\Namespaced\NamespacedCachePool;

$sharedPool = new ArrayCachePool();
$billing = NamespacedCachePool::create($sharedPool, 'billing');
$catalog = NamespacedCachePool::create($sharedPool, 'catalog');

$billingItem = $billing->getItem('summary');
$billingItem->set(['total' => 1200]);
$billing->save($billingItem);

$catalogItem = $catalog->getItem('summary');
$catalogItem->set(['products' => 48]);
$catalog->save($catalogItem);

$billing->clear();
$billing->hasItem('summary');
$catalog->hasItem('summary');
```

The final two calls return `false` and `true`. Clearing `billing` does not remove `catalog` entries.

The decorator uses native hierarchy when the wrapped pool implements `HierarchicalPoolInterface`. Other PSR-6 pools use generation keys for the namespace and hierarchical paths.

The decorator persists a random generation before it derives a storage key. Clearing a namespace or hierarchy path advances the matching generation.

Generation records are typed. If an unrelated value occupies the first internal metadata key, the decorator probes alternate keys instead of overwriting it.

Old entries then become unreachable. The backend can remove them through expiration or eviction.

If generation metadata disappears, the decorator persists a new random value. It never reuses a fixed missing value that could expose entries hidden by an earlier clear.

The decorator throws `CacheException` if it cannot persist generation metadata.

## Hierarchical deletion

Public keys that start with `|` keep hierarchy behavior on every wrapped pool. Deleting `|parent` invalidates that key and its descendants.

Deleting the hierarchy root `|` invalidates every hierarchical key in the namespace. It leaves ordinary keys available.

Both `deleteItem()` and `deleteItems()` apply these hierarchy rules.

## Preserving tag support

Use `NamespacedCachePool::create()` instead of the public constructor. The factory returns a taggable decorator when the wrapped pool supports tags. Its internal tag indexes are scoped to the namespace, so invalidating a tag in one namespace does not delete entries from another namespace. Cache items still return the public tag names from `getPreviousTags()`.

Taggable cache items keep `setTags()`. The returned pool also forwards `invalidateTag()` and `invalidateTags()` to the wrapped pool.

The public constructor creates the basic `NamespacedCachePool` type. Use it only when the caller does not need tag invalidation.

`PrefixedCachePool::create()` follows the same rule. It preserves native tags from its wrapped PSR-6 pool.

## Storage keys

Version 2 encodes namespace components with reversible `_xHH_` byte encoding. ASCII letters, digits, `_`, and `.` remain unchanged. Every other byte is encoded, including each byte in non-ASCII text. Literal lowercase `_x` sequences are encoded too.

Public key components preserve ordinary backend-supported bytes, including `-`, `%`, and non-ASCII text. Only structural `|`, `!`, and literal lowercase `_x` sequences are encoded.

Before upgrading or rolling back, clear affected namespaces when a namespace contains bytes outside `[A-Za-z0-9_.]` or a lowercase `_x` sequence. Clear them when a public key contains `|`, `!`, or a lowercase `_x` sequence.

Version 2 also scopes internal tag indexes to each namespace. Clear namespaced caches containing tagged items before upgrading or rolling back.

Public hierarchy keys use a separate storage path in version 2. Clear namespaced caches containing hierarchy keys before upgrading or rolling back.

Hierarchy-backed namespaces use keys such as `|billing|summary`. Generic PSR-6 pools use fixed-length hash keys that meet PSR-6 key rules.

## Choosing a prefix instead

Use `PrefixedCachePool` when you need readable storage keys and can clear the complete wrapped pool. Its `clear()` method cannot isolate one prefix.

Prefixes use reversible `_xHH_` byte encoding. Bytes outside `[A-Za-z0-9_.]` and literal lowercase `_x` sequences are encoded before joining the public key.

Before upgrading or rolling back, clear an affected prefixed cache when its prefix contains bytes outside `[A-Za-z0-9_.]` or a lowercase `_x` sequence.

Both decorators accept any PSR-6 pool. Namespaces add isolated clearing, while prefixes only prevent ordinary key collisions.
