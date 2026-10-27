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
docker build -t superrosko/php:8.5.12-fpm php/8.5.12-fpm
docker build -t superrosko/node:26.12.1-alpine node/26.12.1-alpine
docker build -t superrosko/alpine:3.22.0-alpine alpine/3.22.0-alpine
```

Run a quick smoke check after building:

```bash
docker run --rm superrosko/php:8.5.12-fpm php -v
docker run --rm superrosko/php:8.5.12-fpm php -m
docker run --rm superrosko/node:26.12.1-alpine node -v
```

## Docker Hub

Login to Docker Hub:

```bash
docker login
```

Build and push the PHP 8.5.12 FPM image:

```bash
docker build --pull -t superrosko/php:8.5.12-fpm php/8.5.12-fpm
docker push superrosko/php:8.5.12-fpm
```

Build, verify, and push every supported PHP image:

```bash
for VERSION in 8.2.35 8.3.36 8.4.27 8.5.12; do
  IMAGE="superrosko/php:${VERSION}-fpm"
  docker build --pull -t "$IMAGE" "php/${VERSION}-fpm"
  docker run --rm "$IMAGE" php -v
  docker push "$IMAGE"
done
```

The same command shape works for the other image families:

```bash
for VERSION in 22.23.4 24.22.1 26.12.1; do
  IMAGE="superrosko/node:${VERSION}-alpine"
  docker build --pull -t "$IMAGE" "node/${VERSION}-alpine"
  docker run --rm "$IMAGE" node -v
  docker push "$IMAGE"
done

docker build -t superrosko/alpine:3.22.0-alpine alpine/3.22.0-alpine
docker push superrosko/alpine:3.22.0-alpine
```

For repeatable releases, set the image name once:

```bash
IMAGE=superrosko/php:8.5.12-fpm
CONTEXT=php/8.5.12-fpm

docker build -t "$IMAGE" "$CONTEXT"
docker run --rm "$IMAGE" php -v
docker push "$IMAGE"
```

## Tagging

Image tags match the source directory name. For example,
`php/8.5.12-fpm/Dockerfile` is built as:

```text
superrosko/php:8.5.12-fpm
```

Use exact upstream versions in tags instead of moving aliases so builds stay
predictable.

## Node.js

Supported, security-patched image tags:

- `22.23.4-alpine`
- `24.22.1-alpine`
- `26.12.1-alpine`

All Node.js images include npm 11.18.0 and Git.

## PHP FPM

Supported, security-patched image tags:

- `8.2.35-fpm`
- `8.3.36-fpm`
- `8.4.27-fpm`
- `8.5.12-fpm`

All PHP images include:

- System tools: `zip`, `unzip`, `git`, `curl`, `locales`, `procps`, `dnsutils`, `nano`.
- PHP extensions: `opcache`, `pdo_mysql`, `pdo_pgsql`, `bcmath`, `pcntl`, `zip`, `gmp`, `intl`, `gd`.
- PECL extensions: `xdebug`, `redis`, `igbinary`, `msgpack`, `memcached`, `mongodb`.
- Composer installed as `/usr/local/bin/composer`.

## Security cleanup

### PHP2Shell

On each supported PHP branch, versions before `8.2.35`, `8.3.36`, `8.4.27`,
and `8.5.12` are affected by
[GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
and
[GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp).
The vulnerable and end-of-life Dockerfiles have been removed from this
repository.

After all consumers have moved to the patched tags, delete these old tags from
the `superrosko/php` repository on Docker Hub:

```text
7.4-fpm
7.4.13-fpm
8.0.0-fpm
8.0.6-fpm
8.1.6-fpm
8.2.13-fpm
8.3.20-fpm
8.5.7-fpm
```

Restart or recreate every running PHP-FPM container after rollout so no worker
continues running vulnerable PHP code.

### Broccoli2Shell / Node2Shell

Node.js versions before `22.23.4`, `24.22.1`, and `26.12.1` are affected by
[CVE-2026-111497](https://nodejs.org/en/blog/vulnerability/october-2026-security-releases),
an out-of-bounds write in the bundled Brotli decoder that can lead to remote
code execution. The vulnerable and end-of-life Node.js Dockerfiles have been
removed from this repository.

After all consumers have moved to the patched tags, delete these old tags from
the `superrosko/node` repository on Docker Hub:

```text
12.16.1-alpine
14.15.1-alpine
14.17.0-alpine
14.17.6-alpine
16.15.0-alpine
25.9.0-alpine
26.4.0-alpine
```

Restart or recreate every running Node.js container after rollout so no process
continues using the vulnerable decoder.

## Credits

- [Dmitriy Bespalov][link-author]

## License

The MIT License (MIT). Please see [License File][link-license] for more information.


[link-author]: https://github.com/superrosko
[link-license]: LICENSE.md
