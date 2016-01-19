# Implement cache tags

*Note: You will find the best performance when using a driver that automatically purges stale records. e.g. Memcache(d) or Redis*

## Usage

To use Tagging with your project, use a pool that implements the `TaggablePoolInterface`, and follow the code examples below.
We create three cache items and store them in the cache with different tags. The order of the tags does not matter. 

```php
// $pool is a PSR-6 cache that implements TaggablePoolInterface

$item = $pool->getItem('tobias', ['developer', 'speaker']);
$item->set('foobar');
$pool->save($item);

$item = $pool->getItem('aaron', ['developer', 'nice guy']);
$item->set('foobar');
$pool->save($item);

$item = $pool->getItem('the king of Sweden', ['nice guy', 'king']);
$item->set('foobar');
$pool->save($item);
```

The following code shows how tags work:

```php
$pool->getItem('tobias', ['developer', 'speaker'])->isHit(); // true
$pool->getItem('tobias', ['speaker', 'developer'])->isHit(); // true
$pool->getItem('tobias', ['developer'])->isHit(); // false
$pool->getItem('tobias', ['king'])->isHit(); // false
$pool->getItem('tobias')->isHit(); // false
```

You can clear the cache like so:

```php

// Remove everything tagged with 'nice guy'
$pool->clear(['nice guy']);
$pool->getItem('tobias', ['developer', 'speaker'])->isHit(); // true
$pool->getItem('aaron', ['developer', 'nice guy'])->isHit(); // false
$pool->getItem('the king of Sweden', ['nice guy', 'king'])->isHit(); // false

// To clear everything
$pool->clear();
```
