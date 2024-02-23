# System Config

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [System config contents (version 0)](#system-config-contents-version-0)
  - [Scalars](#scalars)
    - [Fjord `scalar` (`uint256`) change](#fjord-scalar-uint256-change)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## System config contents (version 0)

### Scalars

#### Fjord `scalar` (`uint256`) change

After Fjord activation, the `scalar` attribute encodes additional scalar information, in a versioned encoding scheme.

The `scalar` is encoded as big-endian `uint256`, interpreted as `bytes32`, and composed as following:

\*Byte ranges are indicated with `[` (inclusive) and `)` (exclusive).

- `0`: scalar-version byte
- `[1, 32)`: depending scalar-version:
  - Scalar-version `0`:
    - `[1, 28)`: padding, should be zero.
    - `[28, 32)`: big-endian `uint32`, encoding the L1-fee `baseFeeScalar`
    - This version implies the L1-fee `blobBaseFeeScalar` is set to 0.
    - In the event there are non-zero bytes in the padding area, `baseFeeScalar` must be set to MaxUint32.
    - This version is compatible with the pre-Ecotone `scalar` value (assuming a `uint32` range).
    - This version implies that `costIntercept` and `costFastlzCoef` are set to 0, and `costTxSizeCoef` is set to `1e6`.
  - Scalar-version `1`:
    - `[1, 24)`: padding, must be zero.
    - `[24, 28)`: big-endian `uint32`, encoding the `blobBaseFeeScalar`
    - `[28, 32)`: big-endian `uint32`, encoding the `baseFeeScalar`
    - This version is meant to configure the EIP-4844 blob fee component, once blobs are used for data-availability.
    - This version implies that `costIntercept` and `costFastlzCoef` are set to 0, and `costTxSizeCoef` is set to `1e6`.
  - Scalar-version `2`:
    - `[1, 12)`: padding, should be zero.
    - `[12, 16)`: big-endian `int32`, encoding the `costTxSizeCoef`
    - `[16, 20)`: big-endian `int32`, encoding the `costFastlzCoef`
    - `[20, 24)`: big-endian `int32`, encoding the `costIntercept`
    - `[24, 28)`: big-endian `uint32`, encoding the `blobBaseFeeScalar`
    - `[28, 32)`: big-endian `uint32`, encoding the `baseFeeScalar`
  - Other scalar-version values: unrecognized.
    OP-Stack forks are recommended to utilize the `>= 128` scalar-version range and document their `scalar` encoding.

Invalid and unrecognized scalar event-data should be ignored,
and the last valid configuration should continue to be utilized.

The scalar values are incorporated into the L2 through the
[Fjord L1 attributes deposit transaction calldata](deposits.md#l1-attributes---fjord).
