
## Maybe-encoded states

### Description

Typically you'll have two separate section of code separated by the enrichement step.
Before this point the value is never present and afterwards the value is always present. The types should reflect this.

**TODO** Elaborate on the case where the enrichment result is a Maybe (for beginners).

### Motivation

As the code grows fewer and fewer people will be able to keep track of which values are populated in specific sections of the code.

### Common occurences:

- state transitions, where each following state involves more data
- enrichments - e.g. read from DB and then enrich with data from some cache

### Example 1

Reading from DB and enriching data (e.g. names correspnding to ids)

#### Don't

```haskell
data Order
  = Order
  { _warehouseId :: UUID
  , _warehouseName :: Maybe Text
  }

queryDb :: UUID -> IO Order
resolveWarehouseName :: UUID -> IO Text

fetchOrder :: UUID -> IO Order
fetchOrder orderId = do
  dbOrder :: Order <- queryDb orderId
  whName <- resolveWarehouseName (_warehouseId dbOrder)
  let dbOrder' = dbOrder { _warehouseName = whName }
  pure dbOrder'
```

#### Do

```haskell
data DbOrder
  = Order
  { _warehouseId :: UUID
  }

data EnrichedOrder
  = EnrichedOrder
  { _warehouseId :: UUID
  , _warehouseName :: Text
  }

queryDb :: UUID -> IO DbOrder
resolveWarehouseName :: DbOrder -> IO EnrichedOrder

fetchOrder :: UUID -> IO EnrichedOrder
fetchOrder orderId = do
  dbOrder <- queryDb orderId
  resolveWarehouseName dbOrder
```

### Example 2

Order state transitions.

As the order moves through the processing steps it accumulates information about each step.

#### Don't

```haskell
data Order
  = Order
  // STATE 1 - OrderCreated
  { _id :: UUID
  // STATE 2 - OrderShipped
  , _trackingNr :: Maybe Text
  // STATE 3 - OrderDelivered
  , _deliveredOn :: Maybe UTCTime
  }
```

#### Do

Preferred:

```haskell
data CreatedOrder
  = CreatedOrder
  { _id :: UUID
  }

data ShippedOrder
  = ShippedOrder
  { _id :: UUID
  , _trackingNr :: Maybe Text
  }

data ShippedOrder
  = ShippedOrder
  { _id :: UUID
  , _trackingNr :: Maybe Text
  , _deliveredOn :: Maybe UTCTime
  }
```

#### Related variants

##### Hierarchical variants of previous solutions

Same as _Don't_ but nested:

```haskell
data Order
  = Order
  { _id :: UUID
  , _shipped :: Maybe ShippedOrder
  }

data ShippedOrder
  = ShippedOrder
  { _trackingNr :: Text
  , _deliveredOrder :: Maybe DeliveredOrder
  }

data DeliveredOrder
  = DeliveredOrder
  { _deliveredOn :: UTCTime
  }
```

Same as _Do_ but nested:

```haskell
data Order
  = Order
  { _id :: UUID
  }

data ShippedOrder
  = ShippedOrder
  { _trackingNr :: Text
  , _order :: Order
  }

data DeliveredOrder
  = DeliveredOrder
  { _deliveredOn :: UTCTime
  , _shippedOrder :: ShippedOrder
  }
```
