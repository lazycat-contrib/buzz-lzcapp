# Buzz for LazyCat

This repository packages [Buzz](https://github.com/block/buzz) as the LazyCat LPK `community.lazycat.app.buzz`.

The package runs the Buzz relay with PostgreSQL, Redis, and MinIO. The launcher entry exposes Buzz's invite pages and optional Git repository browser; Buzz desktop and mobile clients connect to the relay through the LazyCat application domain.

## Installation settings

Prepare these two values before installation:

- `RELAY_OWNER_PUBKEY`: the raw 64-character hexadecimal public key of the human or community owner, not Base64 or an `npub1...` value. Keep the matching private key in the owner's Nostr client; never enter that private key in the LazyCat setup wizard.
- `BUZZ_RELAY_PRIVATE_KEY`: a separate, stable raw 64-character hexadecimal private key used only by the Buzz relay to sign relay and membership events, not Base64 or an `nsec1...` value. Do not reuse the owner's private key, and back it up securely.

`BUZZ_GIT_HOOK_HMAC_SECRET` is generated automatically as an app-scoped stable secret. The database, Redis, and MinIO credentials are managed the same way and do not require user input.

### Generate keys with Buzz

The upstream `buzz-admin` tool prints a Nostr public/secret keypair as 64-character hexadecimal values:

```bash
cargo run -p buzz-admin -- generate-key
```

Run it twice and keep the two identities separate:

1. Generate the owner keypair. Put its public key in `RELAY_OWNER_PUBKEY`, then immediately save its secret key in the owner's Nostr client or password manager. It is not stored by `buzz-admin` and cannot be recovered.
2. Generate another keypair for the relay. Put only this second secret key in `BUZZ_RELAY_PRIVATE_KEY` and back it up securely. Its public key is not required by the setup wizard.

Each human or agent joining Buzz needs its own additional Nostr keypair. Set the agent's `BUZZ_PRIVATE_KEY` to that agent's secret key, then register its public key as a relay member using the stable relay signing key:

```bash
BUZZ_RELAY_PRIVATE_KEY=<relay signing key> \
  cargo run -p buzz-admin -- add-member --pubkey <agent public key>
```

### Generate keys with `nak`

If the Buzz source tree and Rust toolchain are not available, [`nak`](https://github.com/fiatjaf/nak) can generate the same hexadecimal Nostr keys:

```bash
OWNER_PRIVATE_KEY="$(nak key generate)"
RELAY_OWNER_PUBKEY="$(printf '%s\n' "$OWNER_PRIVATE_KEY" | nak key public)"
BUZZ_RELAY_PRIVATE_KEY="$(nak key generate)"

printf 'RELAY_OWNER_PUBKEY=%s\n' "$RELAY_OWNER_PUBKEY"
printf 'BUZZ_RELAY_PRIVATE_KEY=%s\n' "$BUZZ_RELAY_PRIVATE_KEY"
```

Store `OWNER_PRIVATE_KEY` in the owner's Nostr client or password manager; do not enter it in LazyCat. Copy the two printed values into the corresponding setup fields, save the relay private key securely, then clear the temporary shell variables:

```bash
unset OWNER_PRIVATE_KEY RELAY_OWNER_PUBKEY BUZZ_RELAY_PRIVATE_KEY
```

The setup wizard requires raw 64-character hexadecimal values, not `npub` or `nsec` encodings.

If a client only shows an `npub1...` or `nsec1...` value, convert it locally with `nak decode '<npub or nsec>'`. Never paste an `nsec` or private key into chat, an issue, or deployment logs.

## Build

```bash
lzc-cli project release -o dist/buzz.lpk
lzc-cli lpk info dist/buzz.lpk
```

## Automatic updates

`.github/workflows/lazycat.yml` checks `ghcr.io/block/buzz:main` every day. When the Linux amd64 digest changes, it:

1. Updates only the Buzz relay image to the digest-pinned `ghcr.1ms.run/block/buzz:main` mirror after verifying that its Linux amd64 digest matches the source image.
2. Increments the package patch version from the current `0.4.x` value.
3. Builds and verifies the LPK.
4. Creates a `v<version>` tag and GitHub Release with `community.lazycat.app.buzz-v<version>.lpk`.
5. Submits the verified package only to the MiaoMiao private store.

The PostgreSQL, Redis, MinIO, and MinIO Client images are fixed to their initially copied LazyCat Registry references and are not managed by the update workflow.

Configure the GitHub Actions secrets `APPSTORE_URL` and `APPSTORE_TOKEN` for this repository, or authorize it to use the organization-level secrets, before running the workflow. `APP_ID` and `PRIVATE_STORE_GROUP_CODES` remain optional.
