# Symfony AdapterBundle

AdapterBundle creates PSR-6 cache services from Symfony configuration. Version 2 requires PHP 8.2, `psr/cache` 3, and Symfony 6.4, 7, or 8.

## Installation

Install the bundle and the adapter package your app needs:

```bash
composer require cache/adapter-bundle:^2.0 cache/redis-adapter:^2.0
```

Register the bundle in `config/bundles.php` if Symfony Flex has not registered it:

```php
return [
    Cache\AdapterBundle\CacheAdapterBundle::class => ['all' => true],
];
```

The bundle checks each optional adapter dependency when it builds a provider. Install `cache/cache:^2.0` if the app needs the complete adapter collection.

## Configuring providers

Define providers under `cache_adapter.providers`:

```yaml
cache_adapter:
  providers:
    default:
      factory: cache.factory.redis
      options:
        dsn: '%env(REDIS_URL)%'
        pool_namespace: application
      aliases:
        - app.cache
```

This configuration registers `cache.provider.default` and the public alias `app.cache`.

A provider named `default` becomes the default provider. Otherwise, the first configured provider becomes the default. The bundle exposes that provider through both `cache` and `php_cache` aliases.

Applications can map the PSR-6 interface to the default provider:

```yaml
services:
  Psr\Cache\CacheItemPoolInterface: '@cache'
```

## Supported factories

| Factory service | Options |
| --- | --- |
| `cache.factory.apcu` | None |
| `cache.factory.array` | `pool_namespace` |
| `cache.factory.chain` | `services`, `skip_on_failure` |
| `cache.factory.filesystem` | `flysystem_service` |
| `cache.factory.memcache` | `host`, `port`, `redundant_servers` |
| `cache.factory.memcached` | `persistent_id`, `host`, `port`, `pool_namespace`, `redundant_servers`, `driver_options` |
| `cache.factory.mongodb` | `dsn`, `host`, `port`, `database`, `collection` |
| `cache.factory.namespaced` | `service`, `namespace` |
| `cache.factory.predis` | `dsn`, `host`, `port`, `scheme`, `pool_namespace`, `persistent` |
| `cache.factory.prefixed` | `service`, `prefix` |
| `cache.factory.redis` | `dsn`, `host`, `port`, `pool_namespace`, `database` |
| `cache.factory.void` | None |

Options that contain service IDs accept values such as `@app.cache_filesystem`. The bundle converts them to Symfony service references.

`namespace` and `pool_namespace` must contain at least one character. Invalid values fail during container configuration instead of when the provider is first used.

Redis supports password-only and ACL DSNs. Percent-encode reserved characters in either credential:

```text
redis://alice:p%40ss%3Aword@cache.example:6379/0
```

MongoDB defaults to database `application` and collection `cache`. A database in its DSN overrides the `database` option.

## Constructor and runtime fallback

Use `fallback_provider` when the default provider can fail while Symfony creates it. The value can be a provider name or a complete service ID.

The bundle matches bare values against configured provider names first. This supports names that contain dots, such as `warm.tier`.

Service IDs can include an optional leading `@`. The bundle follows aliases and rejects any fallback that leads to the default provider.

It also rejects `cache`, `php_cache`, `cache.provider.default_fallback`, and circular alias chains.

Install the Chain and Void adapters for this configuration:

```bash
composer require cache/chain-adapter:^2.0 cache/void-adapter:^2.0
```

```yaml
cache_adapter:
  fallback_provider: void

  providers:
    redis:
      factory: cache.factory.redis
      options:
        dsn: '%env(REDIS_URL)%'

    void:
      factory: cache.factory.void

    default:
      factory: cache.factory.chain
      options:
        services:
          - '@cache.provider.redis'
          - '@cache.provider.void'
        skip_on_failure: true
```

The bundle resolves the default provider through lazy closures. If its construction throws, the `cache` and `php_cache` aliases resolve to `cache.provider.void`.

Custom aliases on the default provider also resolve to the fallback. Fetching `cache.provider.default` directly bypasses this constructor fallback.

`fallback_provider` does not catch errors from later cache operations. The chain's `skip_on_failure` option handles those errors and removes the failed member.

Each chain member must implement `Cache\Adapter\Common\PhpCachePool`. Add `VoidCachePool` last when every failed read must become a cache miss.

The resulting `CachePoolChain` implements PSR-6 and PSR-16. Apps can use `get()`, `set()`, and the PSR-16 bulk methods on the same provider.

## Decorating cache providers

The Namespaced factory accepts any PSR-6 service. It uses native hierarchy when available and generation keys for other pools.

```yaml
cache_adapter:
  providers:
    shared:
      factory: cache.factory.array

    billing:
      factory: cache.factory.namespaced
      options:
        service: '@cache.provider.shared'
        namespace: billing
```

The Namespaced and Prefixed factories preserve native tag support from the wrapped pool. Their cache items keep tags, and each provider forwards tag invalidation.

Clearing a namespaced provider affects only its namespace. Clearing a prefixed provider clears the complete wrapped pool.

## Upgrading from version 1

AdapterBundle 2 removes the APC factory. Replace `cache.factory.apc` with `cache.factory.apcu` and install `cache/apcu-adapter:^2.0`.

Version 2 also removes every `cache.factory.doctrine_*` service. Replace those providers with a supported native adapter or register an external PSR-6 service directly.

Update renamed options in existing configuration:

* Replace the MongoDB `namespace` option with `database` and `collection`.
* Replace the Predis `schema` option with `scheme`.
* Use modern DSNs for Redis, Predis, and MongoDB where practical.

The underlying cache 4 packages change APCu payloads, Redis tag indexes, namespaced tag indexes, and hierarchy storage paths. Do not run old and new workers against the same affected store.

Clear a namespaced store when a namespace contains bytes outside `[A-Za-z0-9_.]` or lowercase `_x`. Also clear it when a public key contains `|`, `!`, or lowercase `_x`.

Clear namespaced stores containing tagged or hierarchy items. Clear a prefixed store when its prefix contains bytes outside `[A-Za-z0-9_.]` or lowercase `_x`.

Use this sequence for an upgrade or rollback:

1. Stop or drain every worker that uses the affected cache.
2. Clear each affected APCu, Redis, Predis, namespaced, or prefixed store.
3. Deploy the target version and restart the workers.

Report bundle problems on the [AdapterBundle issue tracker](https://github.com/php-cache/adapter-bundle/issues).
