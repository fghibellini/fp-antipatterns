
Draft

## Exposing the ingredients

Don't expose separate parts of your API unless you expect the user to actually configure them. It increases the API surface.
Expose an API that has as few possible ways of calling it as possible.

The user has to learn about all the techniques/libraries that are used (e.g. what is an `SProxy`) instead of simply taking advantage of the existing code.

### Don't

```haskell
module X  (_invalidBody, _duplicateInSubmission, errorsAs) where

_invalidBody :: SProxy "invalidBody"

_duplicateInSubmission :: SProxy "duplicatesInSubmission"

errorsAs ∷
  ∀ variant r1 r2 label a m.
  Functor m =>
  Cons label r2 r1 variant =>
  IsSymbol label =>
  SProxy label ->
  ExceptT r2 m a ->
  ExceptT (Variant variant) m a
errorsAs = withExceptT <<< inj

```

```haskell
someOperation = do
  errorsAs _invalidBody $ except $ someFn
```

### Do

```purescript
module X  (mapToInvalidBody, mapToDuplicateInSubmission) where

mapToInvalidBody :: forall r. Either e a -> Either { invalidBody: e | r } a
mapToInvalidBody = errorsAs _invalidBody <<< except
  where
    _invalidBody :: SProxy "invalidBody"

mapToDuplicateInSubmission :: forall r. Either e a -> Either { dupDuplicateInSubmission: e | r } a
mapToDuplicateInSubmission = errorsAs _dupDuplicateInSubmission <<< except
  where
    _duplicateInSubmission :: SProxy "duplicatesInSubmission"

errorsAs ∷
  ∀ variant r1 r2 label a m.
  Functor m =>
  Cons label r2 r1 variant =>
  IsSymbol label =>
  SProxy label ->
  ExceptT r2 m a ->
  ExceptT (Variant variant) m a
errorsAs = withExceptT <<< inj

```

```purescript
someOperation = do
   mapToInvalidBody someFn
```

