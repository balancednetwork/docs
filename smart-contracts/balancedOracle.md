# Table of Contents

* [External Methods](/smart-contracts/balancedOracle.md#External-Methods)
* [Readonly Methods](/smart-contracts/balancedOracle.md#Readonly-Methods)

# External Methods

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

# Readonly Methods
