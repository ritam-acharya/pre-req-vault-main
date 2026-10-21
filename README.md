# Pre-Req Vault

An [Anchor](https://www.anchor-lang.com/) program for depositing and withdrawing
SOL from a user-owned program-derived vault on Solana. A successful withdrawal
also calls the external prerequisite-registration program to initialize the
user's prerequisite account.

## Features

- Creates one vault state and SOL vault for each wallet.
- Deposits SOL into the user's vault.
- Withdraws SOL from the vault through a PDA-signed system transfer.
- Initializes the external prerequisite account during withdrawal.
- Closes the vault state and transfers the remaining vault balance to the user.

## Program details

| Network | Program ID |
| --- | --- |
| Devnet | `4AobTerp496tbor2FgW8z949A7HGjnkJF7yQEhBpwV3g` |
| Localnet | `DBoobRVqT7PaAhq8obqo95723CHyWvcCLRUKvQ5wYWE4` |

The external prerequisite-registration program used by `withdraw` is deployed
at `TRBZyQHB3m68FGeVsqTK39Wm4xejadjVhP5MAZaKWDM`. Its IDL is checked into
[`idls/registration.json`](./idls/registration.json).

## How it works

For a wallet `user`, the program derives:

```text
vault state = PDA(["state", user])
vault       = PDA(["vault", vault state])
```

The `VaultState` account stores the bump seeds required to validate and sign
subsequent vault operations.

### Instructions

| Instruction | Description |
| --- | --- |
| `initialize` | Creates the user's vault state and derives the vault PDA. |
| `deposit(amount)` | Transfers `amount` lamports from the user to the vault. |
| `withdraw(amount)` | Transfers `amount` lamports from the vault to the user, then initializes the external prerequisite account. |
| `close` | Transfers all remaining vault lamports to the user and closes the vault state account. |

Amounts are specified in lamports. One SOL is `1_000_000_000` lamports.

## Prerequisites

Install and configure:

- Rust with the stable toolchain
- Solana CLI
- Anchor CLI compatible with the project's Anchor version
- Node.js and Corepack
- pnpm

Configure a Solana wallet at `~/.config/solana/id.json`, or update the wallet
path in [`Anchor.toml`](./Anchor.toml). The default provider cluster is
Devnet.

Check that the required tools are available:

```bash
rustc --version
solana --version
anchor --version
node --version
pnpm --version
```

## Installation

```bash
pnpm install
```

## Build

Build the Solana program and generate the Anchor artifacts:

```bash
anchor build
```

The generated IDL and TypeScript types are written under `target/`.

## Testing

The configured TypeScript integration tests run against the provider configured
in `Anchor.toml`:

```bash
anchor test
```

The test suite initializes a vault, deposits 1 SOL, withdraws 0.5 SOL, and
closes the vault. Because the default cluster is Devnet and the local validator
is skipped, make sure the configured wallet is funded and that the deployed
programs are reachable.

To run the Rust/LiteSVM tests directly:

```bash
pnpm exec ts-mocha -p ./tsconfig.json -t 1000000 "tests/**/*.ts"
cargo test
```

Formatting can be checked or applied with:

```bash
pnpm run lint
pnpm run lint:fix
```

## Project structure

```text
.
├── programs/pre-req-vault/src/   # Rust program instructions and state
├── programs/pre-req-vault/tests/ # Rust program tests
├── tests/                        # TypeScript integration tests
├── idls/registration.json        # IDL for the prerequisite CPI dependency
├── Anchor.toml                   # Anchor networks, provider, and scripts
└── package.json                  # TypeScript tooling and scripts
```

## Deployment

Set the desired Solana cluster and wallet in `Anchor.toml`, then build and
deploy:

```bash
anchor build
anchor deploy
```

After deployment, verify that the program ID in `Anchor.toml` matches the
program ID declared in
[`programs/pre-req-vault/src/lib.rs`](./programs/pre-req-vault/src/lib.rs).

## License

This project currently declares the ISC license in `package.json`.