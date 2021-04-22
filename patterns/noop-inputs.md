
## Noop Inputs

### Gist

The caller should handle failure (it's part of piping the operations together).
Functions should accept the smallest set of possible values possible to fulfill their purpose.

### Common occurences

- Any parsing-like step followed by a processing step
- any operation where an intermediary step yields a Maybe-like type

### Exceptions

1. If the noop input represents an actual observable state of the system. In this case it is convenient for our function to reflect the transition function and so we want it to handle the noop-states by itself too.
   e.g. entities that got into error states (effectively their records act as tombstones) that we want to ignore in our processing.

### Don't

```haskell
handler = do
  req :: E Req <- parseRequest
  processReq req

processReq :: E Req -> _
processReq req = runExcept $ do
  r <- except req
  doSomething r
```

### Do

```haskell
handler = do
  req :: E Req <- parseRequest
  case req of
    Right r -> processReq r
    Left e -> ...

processReq :: Req -> _
processReq req = do
  doSomething r
```

