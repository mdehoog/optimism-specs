# Deposits

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [L1 Attributes Deposited Transaction](#l1-attributes-deposited-transaction)
  - [L1 Attributes Deposited Transaction Calldata](#l1-attributes-deposited-transaction-calldata)
    - [L1 Attributes - Fjord](#l1-attributes---fjord)
- [Special Accounts on L2](#special-accounts-on-l2)
  - [L1 Attributes Predeployed Contract](#l1-attributes-predeployed-contract)
    - [Fjord L1Block upgrade](#fjord-l1block-upgrade)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## L1 Attributes Deposited Transaction

### L1 Attributes Deposited Transaction Calldata

#### L1 Attributes - Fjord

On the Fjord activation block, and if Fjord is not activated at Genesis,
the L1 Attributes Transaction includes a call to `setL1BlockValuesEcotone()`
because the L1 Attributes transaction precedes the [Fjord Upgrade Transactions][fjord-upgrade-txs],
meaning that `setL1BlockValuesFjord` is not guaranteed to exist yet.

Every subsequent L1 Attributes transaction should include a call to the `setL1BlockValuesFjord()` function.
The input args are no longer ABI encoded function parameters,
but are instead packed into 5 32-byte aligned segments and one 12-byte segment (starting after the function selector).
Each signed and unsigned integer argument is encoded as big-endian using a number of bytes corresponding to the
underlying type. The overall calldata layout is as follows:

[fjord-upgrade-txs]: derivation.md#fjord

| Input arg         | Type    | Calldata bytes | Segment |
|-------------------|---------|----------------|---------|
| {0x850c16d8}      |         | 0-3            | n/a     |
| baseFeeScalar     | uint32  | 4-7            | 1       |
| blobBaseFeeScalar | uint32  | 8-11           |         |
| sequenceNumber    | uint64  | 12-19          |         |
| l1BlockTimestamp  | uint64  | 20-27          |         |
| l1BlockNumber     | uint64  | 28-35          |         |
| basefee           | uint256 | 36-67          | 2       |
| blobBaseFee       | uint256 | 68-99          | 3       |
| l1BlockHash       | bytes32 | 100-131        | 4       |
| batcherHash       | bytes32 | 132-163        | 5       |
| l1CostIntercept   | int32   | 164-167        | 6       |
| l1CostFastlzCoef  | int32   | 168-171        |         |
| l1CostTxSizeCoef  | int32   | 172-175        |         |

Total calldata length MUST be exactly 176 bytes.

In the first L2 block after the Fjord activation block, the Fjord L1 attributes are first used.

The pre-Fjord values are migrated over 1:1.
Blocks after the Fjord activation block contain all pre-Fjord values 1:1,
and also set the following new attributes:

- The `l1CostIntercept` is set to `0`.
- The `l1CostFastlzCoef` is set to `0`.
- The `l1CostTxSizeCoef` is set to `1e6`.

## Special Accounts on L2

### L1 Attributes Predeployed Contract

- With the Fjord upgrade, the predeploy additionally stores:
  - `l1CostIntercept` (`int32`): system configurable to set the L1 cost formula intercept
  - `l1CostFastlzCoef` (`int32`): system configurable to set the L1 cost formula FastLZ coefficient
  - `l1CostTxSizeCoef` (`int32`): system configurable to set the L1 cost formula transaction size coefficient

#### Fjord L1Block upgrade

The L1 Attributes Predeployed contract, `L1Block.sol`, is upgraded as part of the Fjord upgrade.
The version is incremented to `1.3.0`, and one existing slot begins to store additional data:

- `l1CostIntercept` (`int32`): system configurable to set the L1 cost formula intercept
- `l1CostFastlzCoef` (`int32`): system configurable to set the L1 cost formula FastLZ coefficient
- `l1CostTxSizeCoef` (`int32`): system configurable to set the L1 cost formula transaction size coefficient

The function called by the L1 attributes transaction depends on the network upgrade:

- Before the Fjord activation:
  - `setL1BlockValuesEcotone` is called, following the pre-Fjord L1 attributes rules.
- At the Fjord activation block:
  - `setL1BlockValuesEcotone` function MUST be called, except if activated at genesis.
    The contract is upgraded later in this block, to support `setL1BlockValuesFjord`.
- After the Fjord activation:
  - `setL1BlockValuesEcotone` function is deprecated and MUST never be called.
  - `setL1BlockValuesFjord` MUST be called with the new Fjord attributes.

`setL1BlockValuesFjord` uses a tightly packed encoding for its parameters, which is described in
[L1 Attributes - Fjord](#l1-attributes---fjord).
