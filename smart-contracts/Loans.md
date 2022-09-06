# Table of Contents

* [External Methods](#external-methods)
* [Readonly Methods](#readonly-methods)

# External Methods

* [tokenFallback](#tokenfallback)
* [borrow](#borrow)
* [depositAndBorrow](#depositandborrow)
* [returnAsset](#returnasset)
* [withdrawCollateral](#withdrawcollateral)
* [withdrawAndUnstake](#withdrawandunstake)
* [liquidate](#liquidate)
* [retireBadDebt](#retirebaddebt)
* [retireBadDebtForCollateral](#retirebaddebtforcollateral)
* [Setters](#setters)

<br>

## **tokenFallback**
---

```
void tokenFallback(Address _from, BigInteger _value, byte[] _data)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _from | Address | The address of the collateral holder |
| _value | Number | The amount of collateral deposited |
| _data | byte[] | Json string containing data specifying action |

### **Description:**
Handles collateral deposits of IRC2 tokens, with possibility to take loan in same transaction.

### **Restrictions:**
#### _from:
#### _assetToBorrow:
* Must be larger than 0
#### _data:
* Must be convertible to json with keys "_asset" and "_amount".
#### Caller:
* Must be an supported collateral
* Must be an active collateral

### **External calls:**
* **Collateral contract**: symbol()
* if _asset and _amount is specified
  * [borrow](#borrow) is applied

### **Result:**
* user collateral for _from is increased by value
* if _asset and _amount is specified
  * results is the result of calling [borrow](#borrow) for _from

<br>

## **borrow**
---
```
void borrow(String _collateralToBorrowAgainst, String _assetToBorrow, BigInteger _amountToBorrow)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _collateralToBorrowAgainst | String | The symbol of the collateral on which to collateralize the loan|
| _assetToBorrow | String | The asset to mint |
| _amountToBorrow | Number | The amount of the asset to be minted |

### **Description:**
Issues a loan for caller against the specified collateral, resulting in a debt of specified amount plus a additional fee.

### **Restrictions:**
#### _collateralToBorrowAgainst:
* Must be a supported collateral type.
#### _assetToBorrow:
* Must be a supported asset type.
* Must be active.
#### _amountToBorrow:
* must be larger than 0
* Total debt must be above 10 usd in value.
* current user debt + _amountToBorrow + fee is below locking ratio for specified collateral.
* _amountToBorrow + fee does not bring total debt + bad debt for specified collateral above debt ceiling.

### **External calls:**
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_assetToBorrow)
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_collateralToBorrowAgainst)
* **Rewards** : updateRewardsData("Loans", previousAssetTotalDebt, caller, previousUserDebt)
* **_assetToBorrow contract**: mintTo(caller, _amountToBorrow)
* **_assetToBorrow contract**: mintTo(*feehandler*, fee)
* **CollateralContract** : decimals()

### **Result:**
* total contract wide debt is increased by _amountToBorrow + fee
* total contract wide debt for specific collateral is increased by _amountToBorrow + fee
* user total debt is increased by _amountToBorrow + fee
* user debt for specific collateral is increased by _amountToBorrow + fee

<br>

## **depositAndBorrow**
---

```
@Payable
depositAndBorrow(@Optional String _asset, @Optional BigInteger _amount, @Optional Address _from,  @Optional BigInteger _value)
```

### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _asset | String | The asset to mint, if left empty only deposits collateral |
| _amount | String | The  mount to mint, if left empty only deposits collateral |
| _from | Address | Ignored |
| _value | Number | Ignored |


### **Description:**
If ICX is received, stakes and deposits it as sICX collateral.
If _asset and _amount is specified issues a loan on sICX

### **Restrictions:**
* If _asset and _amount is specified
  * [borrow](#borrow) restrictions is applied
### **External calls:**
* If ICX was deposited
  * **Staking** : stakeICX(LoansAddress)
* If _asset and _amount is specified
  * [borrow](#borrow) is applied

### **Result:**
* If ICX was deposited
  * User sICX collateral is increased by sICX amount given by staking deposited ICX
* if _asset and _amount is specified
  * results is the result of calling [borrow](#borrow) for _from

<br>

## **returnAsset**
---

```
void returnAsset(String _symbol, BigInteger _value, @Optional String _collateralSymbol)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _symbol | String | The symbol of the asset to return |
| _value | Number | The amount to return |
| _collateralSymbol | String | The collateral backing the asset to be returned, Default = "sICX"|

### **Description:**
Burns the returned _value of asset from callers wallet

### **Restrictions:**
#### _symbol:
* Must be a supported asset type.
* Must be active
#### _value:
* Must be more than 0.
* User needs a asset balance of at least _value.
* User need a debt for specified collateral of at least _value
#### _amountToBorrow:
* Must be a supported collateral type

### **External calls:**
**AssetContract** : burnFrom(caller, _value)
* **Rewards** : updateRewardsData("Loans", previousAssetTotalDebt, caller, previousUserDebt)

### **Result:**
* total contract wide debt is decreased by _value
* total contract wide debt for specific collateral is decreased by _value
* user total debt is decreased by _value
* user debt for specific collateral is decreased by _value

<br>

## **withdrawCollateral**
---

```
void withdrawCollateral(BigInteger _value, @Optional String _collateralSymbol);
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _value | Number | The amount to withdraw |
| _collateralSymbol | String | The collateral to withdraw, default "sICX" |

### **Description:**
Withdraws collateral from callers position

### **Restrictions:**
#### caller:
* Has a position on balanced
#### _value:
* Must be more than 0.
* User needs a collateral balance on loans at least of _value.
* Users new collateral/loan ratio must not go below locking ratio for specified collateral
#### _collateralSymbol:
* Must be a supported collateral type

### **External calls:**
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)("bnUSD")
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_collateralSymbol)
* **CollateralContract** : decimals()
* **CollateralContract** : transfer(caller, _value)

### **Result:**
* User collateral position for specified collateral is decreased by _value

<br>

## **withdrawAndUnstake**
---

```
void withdrawAndUnstake(BigInteger _value);
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _value | String | amount of sICX to withdraw and unstake |

### **Description:**
Withdraws sICX collateral and unstakes it for user

### **Restrictions:**
Restrictions are the same as [withdrawCollateral](#withdrawcollateral)(_value, "sICX")

### **External calls:**
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)("bnUSD")
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)("sICX")
* **sICX** : decimals()
* **sICX** : transfer(stakingContract, _value, {"method": "unstake", "user": caller})

### **Result:**
* User collateral position for specified collateral is decreased by _value

<br>

## **liquidate**
---

```
liquidate(Address _owner, @Optional String _collateralSymbol)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _owner | Address | The address to be liquidated |
| _collateralSymbol | String | Which collateral to liquidate for, default sICX |

### **Description:**
Liquidates a user for _collateralSymbol if collateral/debt ratio is below liquidation ratio

### **Restrictions:**
#### _owner:
* _owner has a position in Loans

#### _collateralSymbol:
* Must be a supported collateral type

### **External calls:**
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)("bnUSD")
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_collateralSymbol)
* **CollateralContract** : decimals()
* If a users collateral/debt ratio is below liquidation ratio
  * **CollateralContract** : transfer(caller, liquidationReward)
  * **Rewards** : updateRewardsData("Loans", previousTotalDebt, caller, previousUserDebt)

### **Result:**
* If a users collateral/debt ratio is below liquidation ratio
  * total contract wide debt is decreased by user previous debt
  * total contract wide debt for specific collateral is decreased by user previous debt
  * user total debt is decreased by user previous debt
  * user debt for specific collateral is decreased by user previous debt
  * bad debt for _collateralSymbol is increased by user previous debt
  * liquidation pool for _collateralSymbol is increased by (user previous collateral - liquidation reward)

<br>

## **retireBadDebt**
---

```
retireBadDebt(String _symbol, BigInteger _value)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _symbol | String | The asset to be retired |
| _value | Number | The amount to be retired |

### **Description:**
Retires bad debt for a asset across all collateral types.
Caller receives the different collaterals to compensate, ff needed reserve will be used to compensate.

### **Restrictions:**
#### _symbol:
* _symbol is a supported asset.
* _symbol is a active asset.
* Bad debt for _symbol has to be more than 0.
#### _value:
* Caller asset balance must be at least _value.
* _value must be larger than 0
* _value has to be at least the amount of bad debt retired.

### **External calls:**
* for each collateral with bad debt of _symbol, until _value is spent
  * **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_symbol)
  * **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(collateralSymbol)
  * **CollateralContracts**: decimals()'
  * **AssetContract**: burnFrom(caller, amountRedeemed)
  * **CollateralContract**: transfer(caller, amount needed to compensate or whats left in liquidation pool)
  * If remaining bad debt is 0
    * **CollateralContract** : transfer(caller, rest of Liquidation pool)
  * If collateral in liquidationPool is not enough to cover badDebt or remaining value
    * **Reserve**: [redeem](/smart-contracts/ReserveFund.md#redeem)(caller, remainingValue - liquidationPoolValue, collateralSymbol);

### **Result:**
* for each collateral with bad debt of _symbol, until _value is spent
  * if value is more than bad debt for collateral
    * collateral bad debt is removed/null
    * collateral liquidation pool is null
  * if value is less than bad debt for collateral
    * collateral bad debt is decreased by remaining value
    * collateral liquidation pool is decreased by amount needed to compensate remaining value

<br>

## **retireBadDebtForCollateral**
---

```
retireBadDebtForCollateral(String _symbol, BigInteger _value, String _collateralSymbol)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _symbol | String | The asset to be retired |
| _value | Number | The amount to be retired |
| _collateralSymbol | String | The collateral for which to retire bad debt|

### **Description:**
Retires bad debt for a specific collateral.
Caller receives different the specified collateral from liquidation pool. If needed reserve will be used to compensate.

### **Restrictions:**
#### _symbol:
* _symbol is a supported asset.
* _symbol is a active asset.
#### _value:
* Caller asset balance must be at least _value.
* _value must be larger than 0
* _value has to be at least the amount of bad debt retired.
#### _collateralSymbol:
* _collateralSymbol is a supported collateral.
* _collateralSymbol has bad debt.
### **External calls:**
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_symbol)
* **BalancedOracle**: [getPriceInLoop](/smart-contracts/BalancedOracle.md#getpriceinloop)(_collateralSymbol)
* **CollateralContracts**: decimals()'
* **AssetContract**: burnFrom(caller, amountRedeemed)
* **CollateralContract**: transfer(caller, amount needed to compensate or whats left in liquidation pool)
* If remaining bad debt is 0
  * **CollateralContract** : transfer(caller, rest of Liquidation pool)
* If collateral in liquidationPool is not enough to cover badDebt or remaining value
  * **Reserve**: [redeem](/smart-contracts/ReserveFund.md#redeem)(caller, value or bad debt - liquidationPoolValue, _collateralSymbol);

### **Result:**
* If value is more than bad debt for collateral
  * collateral bad debt is removed/null
  * collateral liquidation pool is null
* If value is less than bad debt for collateral
  * collateral bad debt is decreased by _value
  * collateral liquidation pool is decreased by amount needed to compensate remaining value


## **Setters**
---

All setters should only be available to be called from the governance contract
### **delegate**

```
void delegate(PrepDelegations[] prepDelegations)
```

#### **Description:**
Delegates loans sICX holding to specified p-reps

<br>

### **addAsset**

```
void addAsset(Address _token_address, boolean _active, boolean _collateral)
```

#### **Description:**
Adds a asset to Loans, uses symbol method of _token_address to save address.
_collateral specifies if the assets is a collateral or not.

<br>

### **setLockingRatio**

```
void setLockingRatio(String _symbol, BigInteger _ratio)
```

<br>

### **setLiquidationRatio**

```
void setLiquidationRatio(String _symbol, BigInteger _ratio)
```

<br>

### **setOriginationFee**

```
void setOriginationFee(BigInteger _fee)
```

<br>

### **setRedemptionFee**

```
void setRedemptionFee(BigInteger _fee)
```

<br>

### **setRetirementBonus**

```
void setRetirementBonus(BigInteger _points)
```

<br>

### **setLiquidationReward**

```
void setLiquidationReward(BigInteger _points)
```

<br>

### **setNewLoanMinimum**

```
void setNewLoanMinimum(BigInteger _minimum)
```

<br>

### **setDebtCeiling**

```
void setDebtCeiling(String symbol, BigInteger limit)
```

<br>



<br>

# Readonly Methods
* [getPositionAddress](#getpositionaddress)
* [getAssetTokens](#getassettokens)
* [getCollateralTokens](#getcollateraltokens)
* [getTotalCollateral](#gettotalcollateral)
* [getAccountPositions](#getaccountpositions)
* [getAvailableAssets](#getavailableassets)
* [assetCount](#assetcount)
* [borrowerCount](#borrowercount)
* [hasDebt](#hasdebt)
* [getBalanceAndSupply](#getbalanceandsupply)
* [getBnusdValue](#getbnusdvalue)
* [getLockingRatio](#getlockingratio)
* [getLiquidationRatio](#getliquidationratio)
* [getDebtCeiling](#getdebtceiling)
* [getTotalDebt](#gettotaldebt)
* [getTotalCollateralDebt](#gettotalcollateraldebt)
* [getParameters](#getparameters)

## **getPositionAddress**
---

```
Address getPositionAddress(int _index)
```

### **Description:**
Returns the address for the position with _index. Used to iterate through all users of loans.
Indexes start from 1 and increased by 1 for every new position

<br>

## **getAssetTokens**
---

```
Map<String, String> getAssetTokens()
```

<br>

## **getCollateralTokens**
---

```
Map<String, String> getCollateralTokens()
```

<br>

## **getTotalCollateral**
---

```
BigInteger getTotalCollateral()
```

### **Description:**
Returns Total collateral in Loop

<br>

## **getAccountPositions**
---

```
Map<String, Object> getAccountPositions(Address _owner)
```

### **Description:**
Returns position data for a address

<br>

## **getAvailableAssets**
---

```
Map<String, Map<String, Object>> getAvailableAssets()
```

<br>

## **assetCount**
---

```
int assetCount()
```

<br>

## **borrowerCount**
---

```
int borrowerCount()
```

### **Description:**
Returns total amount of position with debt

<br>

## **hasDebt**
---

```
boolean hasDebt(Address _owner)
```

<br>

## **getBalanceAndSupply**
---

```
Map<String, BigInteger> getBalanceAndSupply(String _name, Address _owner)
```

### **Description:**
Used by rewards, required for a dataSource.

Returns total bnUSD debt and users total bnUSD debt

<br>

## **getBnusdValue**
---

```
BigInteger getBnusdValue(String _name)
```


### **Description:**
Used by rewards to calculate APY

Returns total debt
<br>

## **getLockingRatio**
---

```
BigInteger getLockingRatio(String _symbol)
```

<br>

## **getLiquidationRatio**
---

```
BigInteger getLiquidationRatio(String _symbol)
```

<br>

## **getDebtCeiling**
---

```
BigInteger getDebtCeiling(String symbol)
```

<br>

## **getTotalDebt**
---

```
BigInteger getTotalDebt(String assetSymbol)
```

<br>

## **getTotalCollateralDebt**
---

```
BigInteger getTotalCollateralDebt(String collateral, String assetSymbol)
```

<br>

## **getParameters**
---

```
Map<String, Object> getParameters()
```