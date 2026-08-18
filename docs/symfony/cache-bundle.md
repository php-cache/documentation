# Symfony CacheBundle

CacheBundle connects PSR-6 pools to Symfony sessions, routing, logging, and the web profiler. Version 2 requires PHP 8.2, `psr/cache` 3, and Symfony 6.4, 7, or 8.

Use [AdapterBundle](adapter-bundle.md) to create cache providers from configuration. CacheBundle can also use any PSR-6 service registered by the app.

## Installation

```bash
composer require cache/cache-bundle:^2.0
```

Register the bundle in `config/bundles.php` if Symfony Flex has not registered it:

```php
return [
    Cache\CacheBundle\CacheBundle::class => ['all' => true],
];
```

Inspect the complete configuration reference at any time:

```bash
bin/console config:dump-reference cache
```

## Session cache

Enable Symfony sessions and select a PSR-6 pool:

```yaml
framework:
  session: true

cache:
  session:
    enabled: true
    service_id: cache.provider.default
    use_tagging: true
    prefix: session_
    ttl: 7200
    lock_factory: lock.factory
    lock_ttl: 300
```

CacheBundle registers `cache.service.session` and uses it as Symfony's `session.handler` service.

The default prefix is `session_`. When `ttl` is omitted, the session handler uses 86400 seconds.

With tagging enabled, session entries receive the `session` tag. This lets `cache:flush session` invalidate sessions without clearing unrelated entries in a shared pool.

CacheBundle also registers `cache.session_lock` with the configured Symfony lock factory. Requests with the same session ID wait for an exclusive lock.

The handler holds the lock from session validation or reading through the final write. It releases the lock during `close()`, `destroy()`, or storage error handling.

`lock_factory` defaults to Symfony's `lock.factory` service. Symfony uses a local semaphore when supported and falls back to a local file lock.

Local locks protect processes on one host. Configure a shared lock store when several hosts can handle the same session:

```yaml
framework:
  session: true
  lock:
    session: '%env(SESSION_LOCK_DSN)%'

cache:
  session:
    enabled: true
    service_id: cache.provider.default
    lock_factory: lock.session.factory
    lock_ttl: 300
```

Set `SESSION_LOCK_DSN` to a store that every app host can reach. See the [Symfony Lock documentation](https://symfony.com/doc/current/lock.html) for supported stores and DSNs.

`lock_ttl` is the lock lease in seconds and defaults to 300. For expiring stores, make it longer than the longest expected session request.

## Router cache

Router caching decorates Symfony's `router` service:

```yaml
cache:
  router:
    enabled: true
    service_id: cache.provider.default
    ttl: 604800
    use_tagging: true
    prefix: routes.
```

The default TTL is 604800 seconds. The default prefix is empty.

Cached route matches receive the `router` and `match` tags. Generated URLs receive the `router` and `generate` tags.

Cache keys include the request context that can affect matching or URL generation, including scheme, host, ports, base URL, path, query data, and context parameters. Results cannot cross tenants or hosts that share one pool.

Router caching adds cache work to every match and generation call. Measure the real app before enabling it.

## Logging

Enable logging for provider services that expose `setLogger()`:

```yaml
cache:
  logging:
    enabled: true
    logger: logger.cache
```

The default logger service is `logger`. CacheBundle applies it to services tagged with `cache.provider`.

## Web profiler data collector

The data collector traces services tagged with `cache.provider` and adds their calls and hit ratios to the Symfony profiler.

```yaml
cache:
  data_collector:
    enabled: true
```

When `enabled` is omitted, CacheBundle follows `kernel.debug`. Turn off the collector explicitly when debugging without cache traces.

Version 2 uses service decoration for tracing. It no longer generates subclasses of cache provider classes. The decorator preserves tag-aware pools, traces tag invalidation and failed operations, and resets its call buffer between requests in long-running workers.

## Clearing caches

The `cache:flush` command accepts these cache types:

```bash
bin/console cache:flush session
bin/console cache:flush router
bin/console cache:flush symfony
bin/console cache:flush provider cache.provider.default
bin/console cache:flush all
```

Run `bin/console cache:flush` without arguments for an interactive confirmation before clearing all configured caches.

The `session` and `router` types invalidate their matching tag when the pool supports tags. Otherwise, they clear the entire configured pool. The `provider` type always clears the specified pool. Configured pools can remain private because the command receives a generated service locator.

The `provider` type also accepts a public alias that resolves to a service tagged with `cache.provider`.

The `symfony` type runs Symfony's `cache:clear` command. The `all` type clears session and router caches, then runs `cache:clear`.

## Upgrading from version 1

CacheBundle 2 removes the Doctrine, annotation, serializer, and validation integrations. Remove those sections from `cache` configuration and configure each subsystem through its maintained Symfony or Doctrine integration.

Session storage now requires Symfony Lock. CacheBundle creates the required lock and passes it to the PSR-6 session handler.

Apps that construct `Psr6SessionHandler` or `Psr16SessionHandler` directly must pass a `SessionLockInterface` as the second constructor argument. The options array is now the third argument.

The `cache:flush doctrine` type no longer exists. Supported types are `all`, `session`, `router`, `symfony`, and `provider`.

Remove the old `logging.level` option. Version 2 accepts only the logger service ID.

If app code invalidates router entries directly, replace the old `routing` tag with `router`. Route entries also carry either `match` or `generate`.

The underlying cache 4 packages change APCu payloads, Redis tag indexes, namespaced tag indexes, and hierarchy storage paths. Do not run old and new workers against the same affected store.

Clear a namespaced store when a namespace contains bytes outside `[A-Za-z0-9_.]` or lowercase `_x`. Also clear it when a public key contains `|`, `!`, or lowercase `_x`.

Clear namespaced stores containing tagged or hierarchy items. Clear a prefixed store when its prefix contains bytes outside `[A-Za-z0-9_.]` or lowercase `_x`.

Use this sequence for an upgrade or rollback:

1. Stop or drain every worker that uses the affected cache.
2. Clear each affected APCu, Redis, Predis, namespaced, or prefixed store.
3. Deploy the target version and restart the workers.

Report bundle problems on the [CacheBundle issue tracker](https://github.com/php-cache/cache-bundle/issues).
