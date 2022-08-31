# Table of Contents

* [External Methods](#external-methods)
* [Readonly Methods](#readonly-methods)

# External Methods

* [tokenFallback](#tokenfallback)
* [updateBalTokenDistPercentage](#updatebaltokendistpercentage)
* [addNewDataSource](#addnewdatasource)
* [distribute](#distribute)
* [claimRewards](#claimrewards)
* [updateRewardsData](#updaterewardsdata)
* [updateBatchRewardsData](#updatebatchrewardsdata)
* [addDataProvider](#adddataprovider)
* [setBatchSize](#setbatchsize)

<br>

## **tokenFallback**
---

```
void tokenFallback(Address _from, BigInteger _value, byte[] _data)
```

### **Description:**
TokenFallback which accepts only BALN tokens

<br>

## **updateBalTokenDistPercentage**
---

```
void updateBalTokenDistPercentage(DistributionPercentage[] _recipient_list)
```
### **Parameters**:

| Name | Type | Description |
|----------|-------------|------|
| _recipient_list | DistributionPercentage[] | List of name, percentage pairs |

### **Description:**
Set the baln distribution percentages of each recipient/dataSource

### **Restrictions:**
#### Only callable by governance
#### _recipient_list:
* Has to sum up to 100%.
* Has the be the same length as total amount of recipients/dataSources.
* Each name in input has to exists in rewards

### **Result:**
* Each datasource is assigned a new distPercentage.
* Each name, percentage pair is snapshot for the current day.
<br>

## **addNewDataSource**
---

```
boolean addNewDataSource(String _name, Address _address)
```
### **Parameters**:

| Name | Type | Description |
|----------|-------------|------|
|_name | String | The name used to access the dataSource |
|_address | Address | The address used to fetch total and a users current supply |

### **Description:**
Adds a new dataSource which is able to be assigned a distribution percentage.

### **Restrictions:**
#### Only callable by governance.
#### _name:
* _name does not already exist as a dataSource.
#### _address:
* _address has to be a contract address.

### **Result:**
* Creates a new dataSource with _name and Contract address _address
<br>

## **distribute**
---

```
void distribute()
```

### **Description:**
Mints Baln rewards for one day, while also distributes to platform recipients. For example Reserve

### **External calls:**
If baln has not been minted for current day + 1
 * Baln: transfer(each platform recipient, emission*recipientDist)

### **Result:**
* If baln has not been minted for current day + 1
  * set emission share for each dataSource
  * increase platform day by 1
  * returns false
* If platform Day is higher than current day
  * return true
<br>

## **claimRewards**
---

```
void claimRewards()
```
### **Description:**
For each dataSource updates the callers accrued rewards and transfers them to the user.

### **External calls:**
* calls [distribute](#distribute)
* For each dataSource name
  * **DataSource contract**: getBalanceAndSupply(source name, caller)
* **Baln**: transfer(caller, accruedRewards)

### **Result:**
* For each datasource name:
  * Result of [updateRewardsData](#updaterewardsdata)(name, current supply, caller, current balance)
* Sets user accrued rewards to 0
<br>

## **updateRewardsData**
---

```
void updateRewardsData(String _name, BigInteger _totalSupply, Address _user, BigInteger _balance)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
|_name | String | The name used to access the dataSource |
|_totalSupply | BigInteger | The previous totalSupply |
|_user | Address | The name used to access the dataSource |
|_balance | BigInteger | The previous user balance |

### **Description:**
Updates total weight for specified dataSource and calculates user accrued rewards.
Uses previous supplies since the rewards accrued are only calculated from previous update up to time of the call

### **Restrictions:**
#### Caller
* Update has to be done by a approved DataProvider

### **External calls:**
* calls [distribute](#distribute)

### **Result:**
* Updates total weight for dataSource
* Updates last update timestamp for dataSource
* Updates user weight for dataSource
<br>

## **updateBatchRewardsData**
---

```
void updateBatchRewardsData(String _name, BigInteger _totalSupply, RewardsDataEntry[] _data)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
|_name | String | The name used to access the dataSource |
|_totalSupply | BigInteger | The previous totalSupply |
|_data | RewardsDataEntry[] | List a address, previous balance pairs|

### **Description:**
calls [updateRewardsData](#updaterewardsdata) for each address, balance pair in _data

<br>

## **addDataProvider**
---

```
void addDataProvider(Address _source)
```
### **Parameters**:
| Name | Type | Description |
|----------|-------------|------|
|_source | Address |  |

### **Description:**
Adds _source as a dataProvider
### **Restrictions:**
#### Only callable by governance

### **Result:**
* _source can now call [updateRewardsData](#updaterewardsdata)

<br>

# Readonly Methods

* [getEmission](#getemission)
* [getBalnHoldings](#getbalnholdings)
* [getBalnHolding](#getbalnholding)
* [distStatus](#diststatus)
* [getDataSourceNames](#getdatasourcenames)
* [getRecipients](#getrecipients)
* [getRecipientsSplit](#getrecipientssplit)
* [getDataSources](#getdatasources)
* [getDataSourcesAt](#getdatasourcesat)
* [getSourceData](#getsourcedata)
* [recipientAt](#recipientat)
* [getAPY](#getapy)
* [getDataProviders](#getdataproviders)

## **getEmission**
---

```
BigInteger getEmission(@Optional BigInteger _day)
```

### **Description:**
Returns how much baln will be distributed at the given day.

Negative values return currentDay - _day + 1

<br>

## **getBalnHoldings**
---

```
Map<String, BigInteger> getBalnHoldings(Address[] _holders)
```

### **Description:**
Return accrued rewards for the _holders.

Does not return continuous rewards.

<br>

## **getBalnHolding**
---

```
BigInteger getBalnHolding(Address _holder)
```

### **Description:**

Calculates user rewards up until time of call and return total accrued BALN.
<br>

## **distStatus**
---

```
Map<String, Object> distStatus()
```

### **Description:**
Return platform days for rewards and dataSources
<br>

## **getDataSourceNames**
---

```
List<String> getDataSourceNames()
```

<br>

## **getRecipients**
---

```
List<String> getRecipients()
```

### **Description:**
Returns all sources that can be allocated rewards
<br>

## **getRecipientsSplit**
---

```
Map<String, BigInteger> getRecipientsSplit()
```

### **Description:**
Returns the distribution percentages of each recipient
<br>

## **getDataSources**
---

```
Map<String, Map<String, Object>> getDataSources()
```

<br>

## **getDataSourcesAt**
---

```
Map<String, Map<String, Object>> getDataSourcesAt(BigInteger _day)
```

<br>

## **getSourceData**
---

```
Map<String, Object> getSourceData(String _name)
```

<br>

## **recipientAt**
---

```
Map<String, BigInteger> recipientAt(BigInteger _day)
```

### **Description:**
Distribution percentages at a given day.
<br>

## **getAPY**
---

```
BigInteger getAPY(String _name)
```
<br>

## **getDataProviders**
---

```
List<Address> getDataProviders()
```

<br>
