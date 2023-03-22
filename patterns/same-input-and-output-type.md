# Same Input and Output Type

Generally speaking if I see a pipeline of data transformations like

```
a -> b, b -> c, d -> e, e -> f
```

I should be able to determine where the data is computed/generated/fetched from the type that introduces it.

## Example 1 - Image fetching service

```
data InputModel = InputModel { imageUrls :: [Url] }
data InternalModel = InternalModel { imageUrls :: [Url], images :: [Image] }
data OutputModel = OutputModel { images :: [Image] }

transformation1 :: InputModel -> InternalModel
transformation2 :: InternalModel -> InternalModel
transformation3 :: InternalModel -> OutputModel
```

in the observed example `transformation1` was setting `images` to an empty list. and only later transformations were actually fetching the images.

