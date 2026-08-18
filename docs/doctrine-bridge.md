# PSR-6 to Doctrine Cache bridge

The Doctrine bridge exposes a PSR-6 pool through the legacy `Doctrine\Common\Cache\Cache` API. Use it only when a dependency still requires Doctrine Cache.

## Installation

```bash
composer require cache/psr-6-doctrine-bridge:^4.0
```

Version 2 requires PHP 8.2, Doctrine Cache 2.2, and a PSR-6 v3 implementation.

## Usage

```php
use Cache\Adapter\PHPArray\ArrayCachePool;
use Cache\Bridge\Doctrine\DoctrineCacheBridge;

$pool = new ArrayCachePool();
$cache = new DoctrineCacheBridge($pool);

$cache->save('report.current', ['ready' => true], 60);
$cache->contains('report.current');
$report = $cache->fetch('report.current');
$cache->delete('report.current');

$samePool = $cache->getCachePool();
```

The bridge normalizes characters that Doctrine Cache accepts but PSR-6 reserves.
