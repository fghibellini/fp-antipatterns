
# Business logic in parsing code

This is probably not fp-specific but I've seen it happen way too many times not to mention it.

## Don't

If you have a function that parses JSON, reads DB responses, unmarshals data from FFI calls. Do not mix business logic into its code.
The worst offenders are cases where multiple records in the response are grouped together so the output type is the same if you include the business logic or not, so the reader has no way of knowing that there's some magic going on inside.

## DO

Ideally you want to have different types for the data before and after the transformation

## Why?

people debugging issues will not look into the parsers. They will spend a lot of time trying to understand how something can happen given their expectation of the data coming from the DB.

## Examples

### Example 1

event was coming from message bus. events within a consumption batch for the same target resource were merged based on business defined logic (e.g. one field was simply taken from the most up to date event, other fields were merged on some kind of priority, others were merged into a single hashmap).

### Example 2

Errors codes stored within a structure were translated into differet kind of errors codes within the parser (both codes were encoded as Strings within the application).
