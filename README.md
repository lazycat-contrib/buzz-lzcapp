# Buzz for LazyCat

This repository packages [Buzz](https://github.com/block/buzz) as the LazyCat LPK `community.lazycat.app.buzz`.

The package runs the Buzz relay with PostgreSQL, Redis, and MinIO. The launcher entry exposes Buzz's invite pages and optional Git repository browser; Buzz desktop and mobile clients connect to the relay through the LazyCat application domain.

## Installation settings

Prepare these stable values before installation:

- A 64-character hexadecimal Nostr public key for the relay owner.
- A stable 64-character hexadecimal Nostr private key for the relay.
- A stable Git Hook HMAC secret of at least 32 characters.

Buzz uses Nostr identity keys rather than a username/password login. The remaining database, Redis, and MinIO credentials are generated as app-scoped stable secrets.

## Build

```bash
lzc-cli project release -o dist/buzz.lpk
lzc-cli lpk info dist/buzz.lpk
```

## Automatic updates

`.github/workflows/lazycat.yml` checks `ghcr.io/block/buzz:main` every day. When the Linux amd64 digest changes, it:

1. Copies only the Buzz relay image to the LazyCat Registry.
2. Increments the package patch version from the current `0.4.x` value.
3. Builds and verifies the LPK.
4. Creates a `v<version>` tag and GitHub Release with `community.lazycat.app.buzz-v<version>.lpk`.
5. Submits the verified package to the official LazyCat store.

The PostgreSQL, Redis, MinIO, and MinIO Client images are fixed to their initially copied LazyCat Registry references and are not managed by the update workflow.

Configure the GitHub Actions secret `LAZYCAT_TOKEN` for this repository (or authorize the repository to use the organization-level secret) before running the workflow.
