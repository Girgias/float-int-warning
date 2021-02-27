# PHP RFC: Deprecate implicit non-integer-compatible float to int conversions
  * Version: 0.2
  * Date: 2021-02-03
  * Author: George Peter Banyard, <girgias@php.net>
  * Status: Draft
  * First Published at: http://wiki.php.net/rfc/implicit-float-int-deprecate
  * GitHub mirror: https://github.com/Girgias/float-int-warning

## Introduction 

PHP is a dynamically typed language and as such implicit type coercion naturally arises,
most of these are harmless and rather convenient.
However, `float` to `int` conversions can lead to data loss such as when the `float` is outside the platform integer range, or it has a fractional part.
This extends to the conversion of float strings when converted to `int`.

## Terminology

A float is said to be integer-compatible if posses the following characteristics:

 - Is a number (i.e. not NaN or Infinity)
 - Is in range of a PHP integer (platform dependent)
 - Has no fractional part

## Proposal
Emit an `E_DEPRECATED` deprecation diagnostic for implicit conversion from `float` and float strings to `int` if the floating point number is not integer-compatible.

If the conversion happens from a `float` the diagnostic message is:
> Implicit conversion to int from non-compatible float

If the conversion happens from a `float`-string the diagnostic message is:
> Implicit conversion to int from non-compatible float-string

Raise these warnings to `TypeError` in the next major version (PHP 9.0).

## Rationale

Following the changes in behaviour and definition of numeric strings in PHP 8 [1][2] it should be a reasonable expectation to safely be able to use PHP's type juggling from string to integer as such data can come from a variety of places (HTTP request, database query, text file, etc. ).
However, because `float`s, and by extension float strings, get silently converted to `int` there is no way to know if the data provided is erroneous and/or provokes data loss.

The lack of possibility of knowing if data loss arises necessitates the use of the `strict_type` mode, which is an issue in itself when using a function which returns a `float` but given the input arguments
an `int` compatible return is to be expected, this mostly affects mathematical functions, the most notable example being the ``round()`` function when passing a non-positive precision

The use of a `float` or float string as a string offset already emits a warning as it needs
to perform a conversion, this proposal would generalize this aspect to other areas of PHP.

Finally, attempting to pass a `float` or float string which exceeds the range representable by an `int` already causes a `TypeError` to be thrown.

## Implementation notes

A new C function `is_long_compatible()` is introduced which performs a round trip (i.e. `(double)(zend_long)`) check to establish if a float is integer-compatible.

A new `zend_dval_to_lval_safe()` C function is introduced which performs a check to `is_long_compatible()` that if it fails will emit the deprecation diagnostic.

The `zend_get_long_func()` is modified to accept an additional argument named `is_lax`, which purpose is to toggle between using `zend_dval_to_lval()` or `zend_dval_to_lval_safe()`, as this function is used by the `(int)` cast.

## Backward Incompatible Changes
The following operations will now emit an `E_DEPRECATED` if a non-integer-compatible `float`s or float string is used:

 - Bitwise OR operator `|`
 - Bitwise AND operator `&`
 - Bitwise XOR operator `^`
 - Shift right and left operators
 - Modulo operator
 - The combined assignment operators of the above operators
 - Assignment to a typed property of type `int` in coercive typing mode
 - Argument for a parameter of type `int` for both internal and userland functions in coercive typing mode
 - Returning such a value for userland functions declared with a return type of ``int`` in coercive typing mode

The following operations will now emit an `E_DEPRECATED` if a non-integer-compatible `float` is used:

  - Bitwise NOT operator `~`
  - As an array key

## Proposed PHP Version

Next minor version: PHP 8.1.

## RFC Impact 
### To Existing Extensions 

Test output might need to be adjusted as the `zval_get_long()` (and TBD `convert_to_long()`) will call the new `zend_dval_to_lval_safe()` function instead of `zend_dval_to_lval()`

Extensions which call directly `zend_get_long_func()` will need to be adjusted to either use `zval_get_long()` or pass explicitly the new `lax` flag.

### To OPcache

Rules about accepting floats instead of int would need to be reviewed as emitting diagnostics pose an issue from my understanding

## Unaffected PHP Functionality

 - Manually casting to integer will not raise a notice.
 - Integer to float implicit conversions are not affected.
 - Strict Type behaviour is unaffected.
 - The behaviour of passing float strings as an array key.
 - The behaviour of passing a `float` or float string as a string offset.
 - A bitwise NOT `~` operation on a float string is still performed with string semantics.

## Future Scope
 - Possibility to normalize float strings for array keys.
 - Possibility to allow compatible float and float strings for string offsets.
 - Normalize behaviour when casting to int from out-of-range float.

## Proposed Voting Choices
As per the voting RFC a yes/no vote with a 2/3 majority is needed for this proposal to be accepted.

## Patches and Tests 
Patch: https://github.com/php/php-src/pull/6661

## Implementation 
After the project is implemented, this section should contain
  - the version(s) it was merged into
  - a link to the git commit(s)
  - a link to the PHP manual entry for the feature
  - a link to the language specification section (if any)

## References

[1]<https://wiki.php.net/rfc/saner-numeric-strings>  
[2]<https://wiki.php.net/rfc/string_to_number_comparison>  

## Rejected Features 
Keep this updated with features that were discussed on the mail lists.
