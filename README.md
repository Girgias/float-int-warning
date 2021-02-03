# PHP RFC: Warning for implicit float to int conversions
  * Version: 0.1
  * Date: 2021-02-03
  * Author: George Peter Banyard, <girgias@php.net>
  * Status: Draft
  * First Published at: http://wiki.php.net/rfc/float-int-warning

## Introduction 

PHP is a dynamically typed language and as such implicit type coercion naturally arises, most of these are harmless and rather convenient.
However, `float` to `int` conversion can lead to data loss if the fractional part is non zero.
This extends to the conversion of float strings when converted to `int`.

## Proposal
Emit an `E_WARNING` level diagnostic error for implicit conversion from `float` and float strings to `int` if the fractionnal part is non-zero.
And raise this warning to a `TypeError` in the next major version (PHP 9.0).

## Rationale [W.I.P.]

Cannot safely use PHP's type juggling for string to int, as it will convert an invalid float string, something usefull as everything is a string in HTTP

No way of knowing data loss

String offsets already emit warnigns

## Backward Incompatible Changes
The following operations will now emit an `E_WARNING` if a `float`s or float string with non-zero fractional part is used:

 - Bitwise OR operator `|`
 - Bitwise AND operator `&`
 - Bitwise XOR operator `^`
 - Shift right and left operator
 - Modulo operator
 - The combined assignment operators of the above operators
 - Assignement to a typed property of type `int`
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

### To Opcache 
Rules about accepting floats instead of int would need to be reviewed as emitting diagnostics pose an issue from my udnerstanding

## Open Issues 
Make sure there are no open issues when the vote starts!

## Unaffected PHP Functionality

 - Manually casting to integer will not raise a notice.
 - Integer to float implicit conversions are not affected.
 - Strict Type behaviour is unaffected.
 - The behaviour of passing float strings as an array key.
 - A bitwise NOT `~` operation on a float string is still performed with string semantics.

## Future Scope
 - Possibility to normalize float strings for array keys.

## Proposed Voting Choices
As per the voting RFC a yes/no vote with a 2/3 majority is needed for this proposal to be accepted.

## Patches and Tests 
W.I.P. patch: https://github.com/php/php-src/pull/6661

## Implementation 
After the project is implemented, this section should contain
  - the version(s) it was merged into
  - a link to the git commit(s)
  - a link to the PHP manual entry for the feature
  - a link to the language specification section (if any)

## References 
Links to external references, discussions or RFCs

## Rejected Features 
Keep this updated with features that were discussed on the mail lists.
