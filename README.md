# docker-images

## Introduction

Repo with superrosko docker images.

## Images

The repository keeps one Dockerfile per image tag:

- `php/<version>-fpm` for PHP-FPM images.
- `node/<version>-alpine` for Node.js Alpine images.
- `alpine/<version>-alpine` for Alpine utility images.

## Build

Build an image from the repository root and tag it with the same directory
name:

```bash
docker build -t superrosko/php:8.5.7-fpm php/8.5.7-fpm
docker build -t superrosko/node:26.4.0-alpine node/26.4.0-alpine
docker build -t superrosko/alpine:3.22.0-alpine alpine/3.22.0-alpine
```

Run a quick smoke check after building:

```bash
docker run --rm superrosko/php:8.5.7-fpm php -v
docker run --rm superrosko/php:8.5.7-fpm php -m
docker run --rm superrosko/node:26.4.0-alpine node -v
```

## Docker Hub

Login to Docker Hub:

```bash
docker login
```

Build and push the PHP 8.5.7 FPM image:

```bash
docker build -t superrosko/php:8.5.7-fpm php/8.5.7-fpm
docker push superrosko/php:8.5.7-fpm
```

The same command shape works for the other image families:

```bash
docker build -t superrosko/node:26.4.0-alpine node/26.4.0-alpine
docker push superrosko/node:26.4.0-alpine

docker build -t superrosko/alpine:3.22.0-alpine alpine/3.22.0-alpine
docker push superrosko/alpine:3.22.0-alpine
```

For repeatable releases, set the image name once:

```bash
IMAGE=superrosko/php:8.5.7-fpm
CONTEXT=php/8.5.7-fpm

docker build -t "$IMAGE" "$CONTEXT"
docker run --rm "$IMAGE" php -v
docker push "$IMAGE"
```

## Tagging

Image tags match the source directory name. For example,
`php/8.5.7-fpm/Dockerfile` is built as:

```text
superrosko/php:8.5.7-fpm
```

Use exact upstream versions in tags instead of moving aliases so builds stay
predictable.

## PHP 8.5.7 FPM

The `superrosko/php:8.5.7-fpm` image is based on `php:8.5.7-fpm` and includes:

- System tools: `zip`, `unzip`, `git`, `curl`, `locales`, `procps`, `dnsutils`, `nano`.
- PHP extensions: `opcache`, `pdo_mysql`, `pdo_pgsql`, `bcmath`, `pcntl`, `zip`, `gmp`, `intl`, `gd`.
- PECL extensions: `xdebug`, `redis`, `memcached`, `mongodb`.
- Composer installed as `/usr/local/bin/composer`.

## Credits

- [Dmitriy Bespalov][link-author]

## License

The MIT License (MIT). Please see [License File][link-license] for more information.


[link-author]: https://github.com/superrosko
[link-license]: LICENSE.md
