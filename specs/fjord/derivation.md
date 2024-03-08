# Deriving Payload Attributes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Deriving the Transaction List](#deriving-the-transaction-list)
  - [Network upgrade automation transactions](#network-upgrade-automation-transactions)
    - [Fjord](#fjord)
      - [GasPriceOracle Deployment - Fjord](#gaspriceoracle-deployment---fjord)
      - [GasPriceOracle Proxy Update - Fjord](#gaspriceoracle-proxy-update---fjord)
      - [GasPriceOracle Enable Fjord](#gaspriceoracle-enable-fjord)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Deriving the Transaction List

### Network upgrade automation transactions

#### Fjord

The Fjord hardfork activation block, contains the following transactions in this order:

- L1 Attributes Transaction
- User deposits from L1
- Network Upgrade Transactions
  - GasPriceOracle deployment
  - Update GasPriceOracle Proxy ERC-1967 Implementation Slot
  - GasPriceOracle Enable Fjord

To not modify or interrupt the system behavior around gas computation, this block will not include any sequenced
transactions by setting `noTxPool: true`.

##### GasPriceOracle Deployment - Fjord

The `GasPriceOracle` contract is upgraded to support the new Fjord L1 data fee computation. Post fork this contract
will use FastLZ to compute the L1 data fee.

To perform this upgrade, a deposit transaction is derived with the following attributes:

- `from`: `0x4210000000000000000000000000000000000003`
- `to`: `null`,
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `1,450,000`
- `data`: `0xTODO...` ([full bytecode](../static/bytecode/fjord-gas-price-oracle-deployment.txt))
- `sourceHash`: `0xTODO`
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: Gas Price Oracle Deployment"

This results in the Fjord GasPriceOracle contract being deployed to `0xTODO`,
to verify:

```bash
cast compute-address --nonce=0 0x4210000000000000000000000000000000000003
Computed Address: 0xTODO
```

Verify `sourceHash`:

```bash
❯ cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: Gas Price Oracle Deployment"))
# 0xTODO
```

Verify `data`:

```bash
git checkout TODO (update once merged)
pnpm clean && pnpm install && pnpm build
jq -r ".bytecode.object" packages/contracts-bedrock/forge-artifacts/GasPriceOracle.sol/GasPriceOracle.json
```

This transaction MUST deploy a contract with the following code hash
`0xTODO`.

##### GasPriceOracle Proxy Update - Fjord

This transaction updates the GasPriceOracle Proxy ERC-1967 implementation slot to point to the new GasPriceOracle
deployment.

A deposit transaction is derived with the following attributes:

- `from`: `0x0000000000000000000000000000000000000000`
- `to`: `0x420000000000000000000000000000000000000F` (Gas Price Oracle Proxy)
- `mint`: `0`
- `value`: `0`
- `gasLimit`: `50,000`
- `data`: `0xTODO`
- `sourceHash`: `0xTODO`
  computed with the "Upgrade-deposited" type, with `intent = "Fjord: Gas Price Oracle Proxy Update"`

Verify data:

```bash
cast concat-hex $(cast sig "upgradeTo(address)") $(cast abi-encode "upgradeTo(address)" 0xTODO)
# 0xTODO
```

Verify `sourceHash`:

```bash
cast keccak $(cast concat-hex 0x0000000000000000000000000000000000000000000000000000000000000002 $(cast keccak "Fjord: Gas Price Oracle Proxy Update"))
# 0xTODO
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
