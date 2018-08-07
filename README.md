# Summary

> This is a very preliminary version of the audit.

## Audited contracts

Subjects of this audit are the files from the folder [`contracts/src/2.0.0/`](https://github.com/0xProject/0x-monorepo/tree/v2-prototype/packages/contracts/src/2.0.0) 
in commit `55dbb0ece06d17a9db7b93a0ffa274ff65298002`.

> Current version of audit only includes `protocol` subfolder.

## Documentation

System details are described in [specifications](https://github.com/0xProject/0x-protocol-specification/blob/master/v2/v2-specification.md).

## System description

In general, the system is well designed and provides broad functionality. 
It's well designed in terms of upgradability and can be easily paused in case if system security is compromised.
Major code refactoring was made and lots of new features were added since v1.


# General security tradeoffs

## Major

### Order reuse attack

Currently `Exchange` contract contains storage with all filled and canceled orders.
This creates a possibility to reuse same orders in a new version of `Exchange`.
Or if someone will create a clone of 0x system, orders will be indistinguishable.

**Solution 1** : keep order information in the separate storage contract that can be reused in a new version of `Exchange` contract.

**Solution 2** : order should be bounded to `Exchange` contract address.

## Medium

### Trust issues

It's recommended by the protocol to make unlimited allowance to the `AssetProxy` contracts.
`AssetProxy` allows it's owner to steal all the allowed tokens by adding a new authorized contract.
This situation is mitigated by the 2-weeks delay on every decision that `ProxyOwner` contract can make.
The problem is that 2-weeks delay logic has been put outside `AssetProxy` contracts which makes it harder to control contracts owner.
A user needs to constantly keep track on the `AssetProxy` owner and its updates, `secondsTimeLocked`(2-weeks delay can be changed) and all the authorized contracts in order to make sure that tokens are safe.
This complexity makes it hard to keep track of everything and creates a risk of missing some backdoor in 2-weeks term. 
Also, it takes time for every user to withdraw their allowance and it's possible to spam the network so not everyone will be able to do this in time.


**Solution 1** : create some limitations (like 2-weeks delay) on the `AssetProxy` contract side. 

**Solution 2 (only for takers)** : use forwarder contract for every trade. This can be safer and requires only one transaction.


### **Malicious token**

A token is not guaranteed to be valid and non-malicious. There is no `TokenRegistry` in v2 and no preprocess of tokens. 
A maker can post any address to the `makerToken` or `takerToken` field and almost no validation occurs on 0x protocol side. 

**Note 1** : *No dangerous reentrancy attack has been found yet.*

**Note 2** : malicious token can try to benefit from the inappropriate usage of arbitrage functions. For example, if someone will use `batchFillOrders` or `batchFillOrdersNoThrow` for arbitrage purposes. 

> TBD: Attack details will be shown later.


# Specific issues

## Medium

### **`cancelOrdersUpTo` issues**

* Order with `salt == 2^256` can not be cancelled using this function.
* If someone will cancel orders up to `salt == 2^256 - 1` value, it can not be undone.
* Exception and `uint` overflow on `cancelOrdersUpTo(2^256)` function call.

```uint256 newOrderEpoch = targetOrderEpoch + 1;``` - this statement can cause overflow and exception.

**Solution** : keep `salt == 0` canceled by default. Switch ```orderEpoch[order.makerAddress][order.senderAddress] > order.salt``` to ```orderEpoch[order.makerAddress][order.senderAddress] >= order.salt``` in 'if-canceled' checking.


### `registerAssetProxy` can register a proxy without checking its owner

`AssetProxyOwner` should only register valid proxies that ate owned by this `AssetProxyOwner` contract.

**Solution** : add ownership check



## **Minor**

### **Batch order canceling should be NoThrow**



### **Unbounded loop**

There is unbounded loop in `MixinAuthorizable.sol` in `removeAuthorizedAddress` function.

**Solution** : change data structure to more appropriate. For example, keep values in array and indexes in mapping.


### Using external for functions that are only called from outside

### Rounding error - TBC

### Front running - TBC

### Trading multiple erc721 is not possible ???

### Whitepaper and official website are outdated