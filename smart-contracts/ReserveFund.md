# Table of Contents

* [External Methods](#external-methods)
* [Readonly Methods](#readonly-methods)

# External Methods

* [redeem](#redeem)

<br>

## **redeem**
---

```
void redeem(Address _to, BigInteger _valueInLoop, String collateralSymbol)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _to | Address | The receiver of the redeemed value |
| _valueInLoop | Number | The amount of requested in loop |
| collateralSymbol | String | The collateral prioritized as redemption|

### **Description:**
Redeems a value from the reserve, only usable by loans to retire bad debt

### **Restrictions:**
### caller
  * Only allowed caller is the Loans contract.
### _valueInLoop
* reserve balance of collateralSymbol + sICX + baln has to be more valuable that _valueInLoop.
#### collateralSymbol:
* Has to be a supported collateral in Loans.

### **External calls:**
* **getCollateralTokens**: [getCollateralTokens](/smart-contracts/Loans.md#getcollateraltokens)()
* AssetRedemption for collateralSymbol:
  * **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(collateralSymbol)
  * **CollateralContract**: decimals()
  * **CollateralContract**: balanceOf(ReserveAddress)
  * **CollateralContract**: transfer(_to, _min(balance, amountNeeded))
* If there still remains value to redeem and collateralSymbol is not sICX. Repeats above for sICX
* If there still remains value to redeem use Baln for AssetRedemption

<br>

# Readonly Methods
