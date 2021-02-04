# PHP RFC: Warning for implicit float to int conversions
  * Version: 0.1
  * Date: 2021-02-03
  * Author: George Peter Banyard, <girgias@php.net>
  * Status: Draft
  * First Published at: http://wiki.php.net/rfc/float-int-warning
  * GitHub mirror: https://github.com/Girgias/float-int-warning

## Introduction 

PHP is a dynamically typed language and as such implicit type coercion naturally arises,
most of these are harmless and rather convenient.
However, `float` to `int` conversion can lead to data loss if the fractional part is non zero.
This extends to the conversion of float strings when converted to `int`.

## Proposal
Emit an `E_WARNING` level diagnostic error for implicit conversion from `float` and float strings
to `int` if the fractionnal part is non-zero.

The warning is dependent on the origin type which leads the conversion, i.e. separate warnings for
`float` to `int` and float string to `int`.

And raise these warnings to `TypeError` in the next major version (PHP 9.0).

## Rationale

Following the changes in behaviour and definition of numeric strings in PHP 8 [1][2] it should be a
reasonable expectation to safely be able to use PHP's type juggling from string to int as such data
can come from a variety from places (HTTP request, database query, text file, etc.).
However, because `float`s, and by extension float strings, get silently converted to `int` there is
no way to know if the data provided is erroneous and/or provokes data loss.

The lack of possibility of knowing if data loss arised necessitates the use of the `strict_type` mode,
which is an issue in itself when using a function which returns a `float` but given the input arguments
an `int` compatible return is to be expected, this mostly affects mathematical functions, the most notable
example being the ``round()`` function when passing a non-positive precision

Finally, the use of a `float` or float string as a string offset already emits a warning as it needs
to perform a conversion, this proposal would generalize this aspect to other areas of PHP.

## Implementation notes

A new `zend_dval_to_lval_safe()` C function is introduced which performs an additional `modf()` check against 0.

The C function `zend_dval_to_lval_cap()` is modified and has this additional `modf()` check introduced as it is only
used twice, and both times for converting float strings to int.

## Backward Incompatible Changes
The following operations will now emit an `E_WARNING` if a `float`s or float string with non-zero fractional part is used:

 - Bitwise OR operator `|`
 - Bitwise AND operator `&`
 - Bitwise XOR operator `^`
 - Shift right and left operators
 - Modulo operator
 - The combined assignment operators of the above operators
 - Assignement to a typed property of type `int` in coercive typing mode
 - Argument for a parameter of type `int` for both internal and userland functions in coercive typing mode
 - Returning such a value for userland functions declared with a return type of ``int`` in coercive typing mode

The followig operations will now emit an `E_WARNING` if a `float` with non-zero fractional part is used:

  - Bitwise NOT operator `~`
  - Used as an array key

## Proposed PHP Version(s) 
Next minor version: PHP 8.1.

## RFC Impact 
### To Existing Extensions 
None

### To OPcache 
Rules about accepting floats instead of int would need to be reviewed as emitting diagnostics pose an issue from my understanding

## Open Issues 
### Which severity should the diagnostic be?
The use of `E_WARNING` might be unwarrented and too high for many code bases which hard error on `E_WARNING`.
Maybe use `E_NOTICE` or ressurect `E_STRICT` instead?

As the proposal is to also change this into a `TypeError` in the next major version,
possibly `E_DEPRECATED` is more appropriate?

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
[1] https://wiki.php.net/rfc/saner-numeric-strings
[2] https://wiki.php.net/rfc/string_to_number_comparison

## Rejected Features 
Keep this updated with features that were discussed on the mail lists.
