# Predeploys

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [GasPriceOracle](#gaspriceoracle)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## GasPriceOracle

Following the Fjord upgrade, the values used for L1 fee computation are:

- baseFeeScalar
- blobBaseFeeScalar
- costIntercept
- costFastlzCoef
- costTxSizeCoef
- decimals

These values are managed by the `SystemConfig` contract on the L1. The`decimals` remains hardcoded
to 6.
