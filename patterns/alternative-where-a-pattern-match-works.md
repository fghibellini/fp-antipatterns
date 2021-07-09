
Treating regular application logic (that can be handled with a pattern match) like a sequnce of parsers that could fail.

# Bad

```haskell
productStatus p = activeStatus <|> failedStatus
  where
    activeStatus = case p.status of
      "active" -> Just compute_active_status
      _ -> Nothing
    
    failedStatus = compute_failed_status
```

# Better ?

```haskell
productStatus p = case p.status of
  "active" -> compute_active_status
  _ -> compute_failed_status

```

# Why?

from experience a pattern match (i.e. a tree-like structure of code) is easier to read than having to resolve the complicated control flow of the alternative instance of `Maybe`.
