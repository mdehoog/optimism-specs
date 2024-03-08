# Predeploys

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [GasPriceOracle](#gaspriceoracle)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## GasPriceOracle

Following the Fjord upgrade, three additional values used for L1 fee computation are:

- costIntercept
- costFastlzCoef
- costTxSizeCoef

These values are hard-coded constants in the `GasPriceOracle` contract.
