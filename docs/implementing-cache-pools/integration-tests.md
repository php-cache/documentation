# PSR-6 Integration tests 

To make sure your implementation of PSR-6 is correct you should use this test suite. This will work for **any** PSR-6
cache pool. The tests will make sure your pool work according to the PSR-6 specification. 

## Usage

Install the current stable version of this library.

```bash
composer require --dev cache/integration-tests
```

Create a test that looks like this: 
```php
use Cache\IntegrationTests\CachePoolTest;

class PoolIntegrationTest extends CachePoolTest
{
    public function createCachePool()
    {
        return new CachePool();
    }
}
```


## Versioning

The integration tests package follow semantic versioning like all the other packages in PHP-Cache. For each new test
added we update the minor version. If there is a bugfix in an existing test, bump the minor version. 

This means that you should specify a fix version of the integration tests in your require-dev.
You should not use `dev-master`. Instead you should specify `0.7.0` or even better `^0.7.0`. 

## Skipping tests

You are able to skip some tests that your implementation cannot support. To skip a test, add the test name and a reason 
to a class property like this: 

```php
use Cache\IntegrationTests\CachePoolTest;

class VoidAdapterIntegrationTest extends CachePoolTest
{
    protected $skippedTests = [
      'testGetItem' => 'Void adapter does not save items.',
      'testIsHit'   => 'Void adapter does not save items.',
    ];

    public function createCachePool()
    {
        return new CachePool();
    }
}
```


## Other integration tests

### Test a tagging pool

```php
use Cache\IntegrationTests\TaggableCachePoolTest;

class PoolIntegrationTest extends TaggableCachePoolTest
{
    public function createCachePool()
    {
        return new CachePool();
    }
}
```

### Test a hierarchical pool

```php
use Cache\IntegrationTests\HierarchicalCachePoolTest;

class PoolIntegrationTest extends HierarchicalCachePoolTest
{
    public function createCachePool()
    {
        return new CachePool();
    }
}
```