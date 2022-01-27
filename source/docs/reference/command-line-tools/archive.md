
A tool Run alongside a Substrate-backed chain to index all Blocks, State, and Extrinsic data into a PostgreSQL database.
[Go to documentation](https://github.com/paritytech/substrate-archive)

See useful examples: https://github.com/paritytech/substrate-archive/wiki/Useful-Queries
# Substrate Archive

A generic tool for creating a relational SQL database that mirrors all data from a running Substrate built blockchain.
This is useful for fetching and indexing:
- Blocks
- State 
- Extrinsics

## Installation 

To install and compile the the `substrate-archive-cli` program:
1. Install Postgres
`brew install postgresql`

1. Clone the `substrate-archive` directory using:
`git clone https://github.com/paritytech/substrate-archive.git`

1. Set up the PostgreSQL database
??
`sudo -u postgres psql -U postgres -d postgres -c "ALTER USER postgres WITH PASSWORD '123';"`

1. Start your Substrate node, with `pruning` set to archive. For example: 
`./target/release/node-template --pruning=archive`

1. Look at the current DBs:
    `psql -U postgres -hlocalhost -p6432`

1. Create a db (drop old dbs)
    `CREATE DATABASE`

1. Set `CHAIN_DATA_DB="full_path"`

1. Set up your `Archive.conf` file:
- make sure to set your base bath to primary DB
- tell it where the rocksdb is. State using CHAIN_DATA_DB
- secondary DB is an optimization
- postgres url (set to var if in prod)

1. (Optional) setup up logging and debugging.

1. Run a node template. Make sure you run it in `--release --dev base-path=/tmp/dir --pruning=archive`

1. Make a transaction with your node template.

1. Start up the `substrate-archive` node for your target chain:
    `cargo run --release --  -c archive-conf.toml --chain=polkadot`

1. Go to PGAdmin URL: `localhost:15643/browser/#`.

1. Look at the reference to start making your queries.


## {additional context - optional}

## Basic command usage

The basic syntax for running {`tool`} commands is:

`tool [subcommand] [flag]`

### Flags

You can use the following optional flags with the {`tool`} command.

### Subcommands

You can use the following subcommands with the {`tool`} command. 
For reference information and examples that illustrate using {`tool`} subcommands, select an appropriate command.

| Command | Description
| ------- | -----------
|  |  
|  |

### Output

Depending on the subcommand you specify, the output from the {`tool`} program displays some or all of the following information:

| This field | Contains
| ---------- | ----------
|  |
|  |

### Examples

To {_describe some use_}, run the following command:

`tool --use`

## {specific subcommand}

Use the {tool subcommand} command to {_what the subcommand does_}.

#### Basic usage

`tool subcommand [FLAGS] [OPTIONS] <signature> <uri>`

#### Flags

You can use the following optional flags with the {`tool subcommand`} command.

| Flag   | Description
| ------ | -----------
|        |

#### Options

You can use the following command-line options with the {`tool subcommand`} command.

| Option   | Description
| -------- | -----------
|          |
|          |

#### Examples

{2-3 examples on ways to use this subcommand and any relevant explanations}