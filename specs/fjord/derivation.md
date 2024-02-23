# Deriving Payload Attributes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Deriving the Transaction List](#deriving-the-transaction-list)
  - [Network upgrade automation transactions](#network-upgrade-automation-transactions)
    - [Fjord](#fjord)
      - [L1Block Deployment - Fjord](#l1block-deployment---fjord)
      - [GasPriceOracle Deployment - Fjord](#gaspriceoracle-deployment---fjord)
      - [L1Block Proxy Update - Fjord](#l1block-proxy-update---fjord)
      - [GasPriceOracle Proxy Update - Fjord](#gaspriceoracle-proxy-update---fjord)
      - [GasPriceOracle Enable Fjord](#gaspriceoracle-enable-fjord)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Deriving the Transaction List

### Network upgrade automation transactions

#### Fjord

The Fjord hardfork activation block, contains the following transactions in this order:

- L1 Attributes Transaction, using the pre-Fjord `setL1BlockValuesEcotone`
- User deposits from L1
- Network Upgrade Transactions
  - L1Block deployment
  - GasPriceOracle deployment
  - Update L1Block Proxy ERC-1967 Implementation Slot
  - Update GasPriceOracle Proxy ERC-1967 Implementation Slot
  - GasPriceOracle Enable Fjord

To not modify or interrupt the system behavior around gas computation, this block will not include any sequenced
transactions by setting `noTxPool: true`.

##### L1Block Deployment - Fjord

The `L1Block` contract is upgraded to store the new Fjord Fast LZ parameters.

To perform this upgrade, a deposit transaction is derived with the following attributes:

- `from`: `0x4210000000000000000000000000000000000002`
- `to`: `null`
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `420,000`
- `data`: `0x60806040523...` ([full bytecode](../static/bytecode/fjord-l1-block-deployment.txt))
- `sourceHash`: `0x402f75bf100f605f36c2e2b8d5544a483159e26f467a9a555c87c125e7ab09f3`,
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: L1 Block Deployment"

This results in the Fjord L1Block contract being deployed to `0xa919894851548179A0750865e7974DA599C0Fac7`, to verify:

```bash
cast compute-address --nonce=0 0x4210000000000000000000000000000000000002
Computed Address: 0xa919894851548179A0750865e7974DA599C0Fac7
```

Verify `sourceHash`:

```bash
cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: L1 Block Deployment"))
# 0x402f75bf100f605f36c2e2b8d5544a483159e26f467a9a555c87c125e7ab09f3
```

Verify `data`:

```bash
git checkout TODO (update once merged)
pnpm clean && pnpm install && pnpm build
jq -r ".bytecode.object" packages/contracts-bedrock/forge-artifacts/L1Block.sol/L1Block.json
```

This transaction MUST deploy a contract with the following code hash
`0x12e89c50902af815d85608f9a2a35579a74e9491077b94211c96f79ef265bf9c`.

##### GasPriceOracle Deployment - Fjord

The `GasPriceOracle` contract is upgraded to support the new Fjord L1 data fee computation. Post fork this contract
will use FastLZ to compute the L1 data fee.

To perform this upgrade, a deposit transaction is derived with the following attributes:

- `from`: `0x4210000000000000000000000000000000000003`
- `to`: `null`,
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `1,450,000`
- `data`: `0x60806040523...` ([full bytecode](../static/bytecode/fjord-gas-price-oracle-deployment.txt))
- `sourceHash`: `0x86122c533fdcb89b16d8713174625e44578a89751d96c098ec19ab40a51a8ea3`
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: Gas Price Oracle Deployment"

This results in the Fjord GasPriceOracle contract being deployed to `0xFf256497D61dcd71a9e9Ff43967C13fdE1F72D12`,
to verify:

```bash
cast compute-address --nonce=0 0x4210000000000000000000000000000000000003
Computed Address: 0xFf256497D61dcd71a9e9Ff43967C13fdE1F72D12
```

Verify `sourceHash`:

```bash
❯ cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: Gas Price Oracle Deployment"))
# 0x86122c533fdcb89b16d8713174625e44578a89751d96c098ec19ab40a51a8ea3
```

Verify `data`:

```bash
git checkout TODO (update once merged)
pnpm clean && pnpm install && pnpm build
jq -r ".bytecode.object" packages/contracts-bedrock/forge-artifacts/GasPriceOracle.sol/GasPriceOracle.json
```

This transaction MUST deploy a contract with the following code hash
`0xcb82de8a527fee307214950192bf0ff5b2701c6b6eda2fbd025cf6d4075fbe38`.

##### L1Block Proxy Update - Fjord

This transaction updates the L1Block Proxy ERC-1967 implementation slot to point to the new L1Block deployment.

A deposit transaction is derived with the following attributes:

- `from`: `0x0000000000000000000000000000000000000000`
- `to`: `0x4200000000000000000000000000000000000015` (L1Block Proxy)
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `50,000`
- `data`: `0x3659cfe6000000000000000000000000a919894851548179a0750865e7974da599c0fac7`
- `sourceHash`: `0x0fefb8cb7f44b866e21a59f647424cee3096de3475e252eb3b79fa3f733cee2d`
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: L1 Block Proxy Update"

Verify data:

```bash
cast concat-hex $(cast sig "upgradeTo(address)") $(cast abi-encode "upgradeTo(address)" 0xa919894851548179A0750865e7974DA599C0Fac7)
0x3659cfe6000000000000000000000000a919894851548179a0750865e7974da599c0fac7
```

Verify `sourceHash`:

```bash
cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: L1 Block Proxy Update"))
# 0x0fefb8cb7f44b866e21a59f647424cee3096de3475e252eb3b79fa3f733cee2d
```

##### GasPriceOracle Proxy Update - Fjord

This transaction updates the GasPriceOracle Proxy ERC-1967 implementation slot to point to the new GasPriceOracle
deployment.

A deposit transaction is derived with the following attributes:

- `from`: `0x0000000000000000000000000000000000000000`
- `to`: `0x420000000000000000000000000000000000000F` (Gas Price Oracle Proxy)
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `50,000`
- `data`: `0x3659cfe6000000000000000000000000ff256497d61dcd71a9e9ff43967c13fde1f72d12`
- `sourceHash`: `0x1e6bb0c28bfab3dc9b36ffb0f721f00d6937f33577606325692db0965a7d58c6`
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: Gas Price Oracle Proxy Update"`

Verify data:

```bash
cast concat-hex $(cast sig "upgradeTo(address)") $(cast abi-encode "upgradeTo(address)" 0xFf256497D61dcd71a9e9Ff43967C13fdE1F72D12)
0x3659cfe6000000000000000000000000ff256497d61dcd71a9e9ff43967c13fde1f72d12
```

Verify `sourceHash`:

```bash
cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: Gas Price Oracle Proxy Update"))
# 0x1e6bb0c28bfab3dc9b36ffb0f721f00d6937f33577606325692db0965a7d58c6
```

##### GasPriceOracle Enable Fjord

This transaction informs the GasPriceOracle to start using the Fjord gas calculation formula.

A deposit transaction is derived with the following attributes:

- `from`: `0xDeaDDEaDDeAdDeAdDEAdDEaddeAddEAdDEAd0001` (Depositer Account)
- `to`: `0x420000000000000000000000000000000000000F` (Gas Price Oracle Proxy)
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `80,000`
- `data`: `0x8e98b106`
- `sourceHash`: `0xbac7bb0d5961cad209a345408b0280a0d4686b1b20665e1b0f9cdafd73b19b6b`,
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: Gas Price Oracle Set Fjord"

Verify data:

```bash
cast sig "setFjord()"
0x8e98b106
```

Verify `sourceHash`:

```bash
cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: Gas Price Oracle Set Fjord"))
# 0xbac7bb0d5961cad209a345408b0280a0d4686b1b20665e1b0f9cdafd73b19b6b
```
