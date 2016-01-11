# PSR-6 Integration tests 

To make sure your implementation of PSR-6 is correct you should use this test suite. 

### Usage

Install the dev-master version of this library.

```bash
composer require --dev cache/integration-tests:dev-master
```

Create a test that looks like this: 
```php
class PoolIntegrationTest extends CachePoolTest
{
    public function createCachePool()
    {
        return new CachePool();
    }
}
```

You could also test your tag implementation:
```php
class TagIntegrationTest extends TaggableCachePoolTest
{
    public function createCachePool()
    {
        return new CachePool();
    }
}
```

### Versioning

The integration tests package follow semantic versionioning like all the other packages in PHP-Cache. For each new test
added we update the minor version. If there is a bugfix in an existing test bump the minor version. 

This means that you should specify a fix version of the integration tests in your require-dev. You should not use `dev-master` or 
`~0.4.0`. Instead you should specify `0.4.0` or even better `^0.4.0`. 