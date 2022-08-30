# Table of Contents

* [External Methods](#external-methods)
* [Readonly Methods](#readonly-methods)

# External Methods

* [tokenFallback](#tokenfallback)
* [whitelistTokens](#whitelisttokens)
* [updateLimit](#updatelimit)
* [setFeeIn](#setfeein)
* [setFeeOut](#setfeeout)

<br>

## **tokenFallback**
---

```
void tokenFallback(Address _from, BigInteger _value, byte[] _data)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| _from | Address | The address which sent and will receive funds |
| _value | Number | The amount of stablecoin deposited |
| _data | byte[] | When caller is bnusd this specifies target token |

### **Description:**
Handles collateral deposits of IRC2 tokens, with possibility to take loan in same transaction.

### **Restrictions:**
#### _value:
* Larger than 0
* If caller is bnUSD
  * _value + current balance must be less than limit
  * stabilityFund balance of targetAsset needs to be at least equal to _value
#### _data:
* If caller is bnUSD
  * _data must be convertible to a address
  * address must a whitelisted token
#### Caller:
* Must be a whitelisted asset

### **External calls:**
* If caller is bnUSD
  * **bnUSD Contract** : burn(_value-fee)
  * **bnUSD Contract** : transfer(feeHandler, fee)
  * **target token Contract** : transfer(_from, _value-fee in target amount)
* If caller is a whitelistedToken
  * **bnUSD Contract** : mint(_value in bnUSD)
  * **bnUSD Contract** : transfer(feeHandler, fee)
  * **bnUSD Contract** : transfer(_from, _value in bnUSD - fee)

<br>

## **whitelistTokens**
---

```
void whitelistTokens(Address _address, BigInteger _limit);
```

### **Description:**
Governance only function that whitelist _address with a bnusd mint limit of _limit

<br>

## **updateLimit**
---

```
void updateLimit(Address _address, BigInteger _limit);
```

### **Description:**
Governance only function that updates _address with a new bnusd mint limit of _limit

<br>

## **setFeeIn**
---

```
void setFeeIn(BigInteger _feeIn)
```

### **Description:**
Set the fee taken when depositing a stable asset to mint bnUSD

<br>

## **setFeeOut**
---

```
void setFeeOut(BigInteger _feeOut);
```

### **Description:**
Set the fee taken when depositing bnUSD for another stable token

<br>

# Readonly Methods

* [getFeeIn](#getfeein)
* [getFeeOut](#getfeeout)
* [getLimit](#getlimit)
* [getAcceptedTokens](#getacceptedtokens)


## **getFeeIn**
---

```
BigInteger getFeeIn()
```

<br>

## **getFeeOut**
---

```
BigInteger getFeeOut()
```

<br>

## **getLimit**
---

```
BigInteger getLimit(Address _address);
```

<br>

## **getAcceptedTokens**
---

```
List<Address> getAcceptedTokens();
```

<br>