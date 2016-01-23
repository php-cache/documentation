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

$item = $pool->getItem('aaron', ['developer', 'awesome']);
$item->set('foobar');
$pool->save($item);

$item = $pool->getItem('the_king_of_Sweden', ['awesome', 'king']);
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

// Remove everything tagged with 'awesome'
$pool->clear(['awesome']);
$pool->getItem('tobias', ['developer', 'speaker'])->isHit(); // true
$pool->getItem('aaron', ['developer', 'awesome'])->isHit(); // false
$pool->getItem('the_king_of_Sweden', ['awesome', 'king'])->isHit(); // false

// To clear everything
$pool->clear();
```

## Implemantation notes

There is a rare case where you might find issues with this implementation. Some tags cached in memory for performance, 
this might lead to uncleared cache if you have long running request. The following example will show the issue. 

```php
// Request A, T: 0
$item = $pool->getItem('key', ['tag']);
$item->set('value');
$pool->save($item);
$pool->isHit('key', ['tag']); // true

// Request B, T: 1
$pool->clear(['tag']);
$pool->isHit('key', ['tag']); // false

// Request C, T: 2
$pool->isHit('key', ['tag']); // false

// Request A, T: 2
$pool->isHit('key', ['tag']); // true
```

If request A is a background task that you are running for hours/days then you have a problem that the cache tag never
will be cleared. 