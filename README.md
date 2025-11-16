<p align="center">
    <a href="https://github.com/elephant-php/vo-ttl" target="_blank">
        <img src="elephant-php.webp" height="150px" alt="Yii">
    </a>
    <h1 align="center">TTL - Time To Live</h1>
    <br>
</p>


Immutable Value Object Time To Live for cache.


## Requirements

- PHP 8.1 or higher.
  
## Installation

The package could be installed with [Composer](https://getcomposer.org):

```shell
    composer require elephant-php/vo-ttl
```

## Configuration

Abstract example for your cache

```php
/** @var \Psr\SimpleCache\CacheInterface $psrCache */
$cache = new Cache();
```

## `Ttl` object

`Ttl` is a simple immutable value object that represents cache time-to-live (TTL) in seconds.
It eliminates magic numbers (like 60 * 60 or 3600), improves readability, and provides convenient factory methods.

Below are examples on how to use it.

If you're using PSR-16 cache adapter directly:

- TTL must be an integer number of seconds or `null` for infinite lifetime.
- Always use `->toSeconds()` when using `Ttl` object.

```php
use Elephant\Ttl\Ttl;

$cache = new Cache();

$cache->set('key1', 'value1', Ttl::seconds(30)->toSeconds()); // 30 seconds
$cache->set('key2', 'value2', Ttl::minutes(15)->toSeconds()); // 15 minutes
$cache->set('key3', 'value3', Ttl::hours(2)->toSeconds());    // 2 hours
$cache->set('key4', 'value4', Ttl::days(1)->toSeconds());     // 1 day

// Complex durations
$ttl = Ttl::create(sec: 30, min: 10, hour: 1); // 1 hour 10 minutes 30 seconds
$cache->set('key', 'value', $ttl->toSeconds());

// Infinity / no expiration
$cache->set('key6', 'value6', Ttl::forever()); // shorthand for null
$cache->set('key7', 'value7', Ttl::from(null));
```

### Creating and Normalizing TTL

The Ttl::from() method normalizes various TTL representations (Ttl, DateInterval, int, string, or null) into a Ttl object.
```php
$ttl = Ttl::from(new DateInterval('PT45M')); // 45 minutes
$ttl = Ttl::from(10); // 10 seconds
$ttl = Ttl::from('12'); // 12 seconds
$ttl = Ttl::from(null); // Infinity / no expiration
$ttl = Ttl::from(Ttl::seconds(500));

$ttl = Ttl::create(sec: 30, min: 15);

// From DateInterval
$ttl = Ttl::fromInterval(new DateInterval('PT45M'));
$cache->set('key', 'value', $ttl->toSeconds());

// Ttl::forever() is just a shorthand for `null` TTL (no expiration)
$cache->set('key', 'value', Ttl::forever()->toSeconds());
// or
$cache->set('key', 'value', Ttl::from(null)->toSeconds());
```

### When using `Ttl` with Yii cache wrapper:

- You can pass a `Ttl` object in the constructor as the default value.
- You can pass it to methods like `getOrSet()` which expect integer number of seconds or `null`.
```php
use Your\Cache\Cache;
use Your\Cache\ArrayCache;

use Elephant\Ttl\Ttl;

$cache = new Cache(new ArrayCache(), Ttl::minutes(5)); // default TTL
$cache->getOrSet('key', 'value'); // // Uses default TTL

// Custom TTL
$cache->getOrSet('key2', fn() => 'value2', Ttl::seconds(30)->toSeconds());
$cache->getOrSet('key3', fn() => 'value3', Ttl::forever()->toSeconds()); // No expiration
```
### Additional Features
Checking Infinite TTL: 

Use isForever() to check if a TTL represents "forever" (i.e., no expiration). It returns true when the TTL value is null.

```php
if (Ttl::from(null)->isForever()) {
    // No expiration
}
````

### Accessing TTL Value
Use toSeconds() to get the TTL in seconds (int) or null for "forever". The public $value property can be accessed directly (e.g., Ttl::seconds(30)->value), but toSeconds() is preferred for clarity.

```php
$ttl = Ttl::seconds(60);
$seconds = $ttl->toSeconds(); // Returns 60
$seconds = $ttl->value; // Also 60
```

### Invalid TTL values
```php
$ttl = Ttl::from('abc'); // Converts to 0 (expired)
$ttl = Ttl::from(1.5);   // TypeError: invalid TTL type
```


## License

MIT License

Please see [`LICENSE`](./LICENSE.md) for more information.
