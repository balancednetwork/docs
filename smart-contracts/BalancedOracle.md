# Table of Contents

* [External Methods](external-methods)
* [Readonly Methods](readonly-methods)

# External Methods

* [getPriceInLoop](#getPriceInLoop)
* [Setters](#setters)

## **getPriceInLoop**
```
BigInteger getPriceInLoop(String symbol)
```

### **Description:**
Calculates and returns that latest price in loop for a symbol.

sICX: uses staking contracts to fetch the price in loop

Dex Priced assets: Uses the price from the previous call and the difference in blocks between them to calculate a new EMA. So the current price has only future impact.

$decayWeight = (1-decay)^{blockDiff}$

$priceChange = previousPrice - currentEMA$

$newEMA = previousPrice - previousPrice * decayWeight$

Other symbols: try to fetch rate directly from BandOracle

### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| symbol | String | The symbol of token which price you want in loop |

### **Restrictions:**
#### symbol:
* Must be a symbol supported by that has either a peg, is dexPriced, is sICX, or natively supported by the band oracle.

### **External calls:**
* if symbol is "sICX"
  * Staking:getTodayRate()
* if symbol is dexPriced
  * Dex: getPoolBase(poolId) -> baseAddress
  * baseAddress contract: decimals()
  * Dex: getQuotePriceInBase(poolId)
  * BandOracle: get_reference_data("USD", "ICX");
* if symbol is neither "sICX or dexPriced
  * BandOracle: get_reference_data(symbol, "ICX");

### **Result:**
  * if symbol is dexPriced
    * previousPrices[symbol] = currentPrice
    * lastUpdateBlock[symbol] = current block
    * movingAverages[symbol] = new EMA
  * return priceInLoop of symbol

<br>

## **Setters**
---
All setters should only be available to be called from the governance contract

### **addDexPricedAsset**

```
void addDexPricedAsset(String symbol, BigInteger dexBnusdPoolId);

```

<br>

### **setPeg**

```
void setPeg(String symbol, String peg);

```

<br>

### **setDexPriceEMADecay**

```
void setDexPriceEMADecay(BigInteger decay);

```

<br>

# Readonly Methods

* [getLastPriceInLoop](#getpriceinloop)
* [getAssetBnusdPoolId](#getassetbnusdpoolid)
* [getPeg](#getpeg)
* [getDexPriceEMADecay](#getdexpriceemadecay)

## **getLastPriceInLoop**
---
```
BigInteger getLastPriceInLoop(String symbol)
```

### **Description:**
Calculates and returns that latest price in loop for a symbol.

sICX: uses staking contracts to fetch the price in loop

Dex Priced assets: Uses the price from the latest getPriceInLoop call and the difference in blocks between them to calculate a new EMA.

$decayWeight = (1-decay)^{blockDiff}$

$priceChange = previousPrice - currentEMA$

$newEMA = previousPrice - previousPrice * decayWeight$

Other symbols: try to fetch rate directly from BandOracle

### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
| symbol | String | The symbol of token which price you want in loop |

### **Restrictions:**
#### symbol:
* Must be a symbol supported by that has either a peg, is dexPriced, is sICX, or natively supported by the band oracle.

### **External calls:**
* if symbol is "sICX"
  * Staking:getTodayRate()
* if symbol is neither "sICX or dexPriced
  * BandOracle: get_reference_data(symbol, "ICX");

<br>

### **getAssetBnusdPoolId**
---
```
void getAssetBnusdPoolId((String symbol)
```
If symbol is dex priced this will return what dex pool id it uses to calculate the price.

<br>

### **getPeg**
---
```
void getPeg(String symbol)
```


#### **Description:**
If symbol is oracle priced this will return what symbol it uses to fetch the price from band. example bnUSD->USD


<br>

### **getDexPriceEMADecay**
---
```
void getDexPriceEMADecay()
```

<br>
