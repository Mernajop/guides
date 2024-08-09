# KYVE Node Guide

**Install Go**

KYVE is built using [Go](https://go.dev/dl/) version `1.20+`. Follow the instructions from the official website to install go.

In addition to Go, make sure you have [`git`](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [`make`](https://www.gnu.org/software/make/) installed.

**GitHub**

Clone and build KYVE using `git`:

```bash
git clone https://github.com/KYVENetwork/chain.git
cd chain
git fetch
git checkout tags/<tag>
make build ENV=mainnet
```

Here the `<tag>` is the latest version which you can get [here](https://github.com/KYVENetwork/chain/tags).

> [!NOTE]  
> You can find the compiled binary under `chain/build/kyved`.


After the download was done, verify that it was successful:

```bash
./kyved version
```

## Initialize Node[](https://docs.kyve.network/validators/chain_nodes/installation#initialize-node)

We need to initialize the node to create all the necessary validator and node configuration files

- Mainnet
- Kaon
- Korellia

**Save Chain ID**

We recommend saving the chain-id into your kyved's client.toml. This will make it so you do not have to manually pass in the chain-id flag for every CLI command.

```bash
./kyved config chain-id kyve-1
```

**Initialize**

Initialize node by providing your moniker and the chain id

```bash
./kyved init <your_custom_moniker> --chain-id kyve-1
```

> [!WARNING]  
> **IMPORTANT:** Monikers can contain only ASCII characters. Using Unicode characters will render your node unreachable.

Once the installation and initialization was successful you can proceed to the node configuration.
