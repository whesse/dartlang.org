---
layout: default
title: "Strings"
description: Recipes and prescriptions for using Dart.
has-permalinks: true
--- 

# {{ page.title }}

1. [Introduction](#introduction)
    1. [What is the Basic Multilingual Plane?](#what-is-the-basic-multilingual-plane)
    1. [What is a surrogate pair?](#what-is-a-surrogate-pair)
    1. [What is the difference between a code point and a code unit?](#what-is-the-difference-between-a-code-point-and-a-code-unit)
    1. [What exactly is a character?](#what-exactly-is-a-character)
1. [Concatenating Strings](#concatenating-strings)
1. [Interpolating expressions inside strings](#interpolating-expressions-inside-strings)
1. [Escaping special characters](#escaping-special-characters)
1. [Incrementally building a string efficiently using a StringBuffer](#incrementally-building-a-string-efficiently-using-a-stringbuffer)
1. [Converting between string characters and numerical codes](#converting-between-string-characters-and-numerical-codes)
1. [Determining if a string is empty](#determining-if-a-string-is-empty)
1. [Removing leading and trailing whitespace](#removing-leading-and-trailing-whitespace)
1. [Calculating the length of a string](#calculating-the-length-of-a-string)
1. [Subscripting a string](#subscripting-a-string)
1. [Processing a string one character at a time](#processing-a-string-one-character-at-a-time)
1. [Splitting a string into substrings](#splitting-a-string-into-substrings)
1. [Changing string case](#changing-string-case)
1. [Determining whether a string contains another string](#determining-whether-a-string-contains-another-string)
1. [Finding matches of a regExp pattern in a string](#finding-matches-of-a-regexp-pattern-in-a-string)
1. [Substituting strings based on regExp matches](#substituting-strings-based-on-regexp-matches)
{:.toc}

## Introduction

A Dart string represents a sequence of characters encoded in UTF-16. Decoding
UTF-16 yields Unicode code points. Borrowing terminology from Go, Dart uses
the term `rune` for an integer representing a Unicode code point. 

The string recipes included in this chapter assume that you have some
familiarity with Unicode and UTF-16. Here is a brief refresher:

### What is the Basic Multilingual Plane?

The Unicode code space is divided into seventeen planes of 65,536 points each. 
The first plane (code points U+0000 to U+FFFF) contains the most
frequently used characters and is called the Basic Multilingual Plane or BMP.

### What is a Surrogate Pair?

The term 'surrogate pair' refers to a means of encoding Unicode characters
outside the Basic Multilingual Plane.

In UTF-16, two-byte (16-bit) code sequences are used to store Unicode
characters. Since two bytes can only contain the 65,536 characters in the 0x0
to 0xFFFF range, a pair of code points are used to store values in the
0x10000 to 0x10FFFF range.

For example the Unicode character for musical Treble-clef (🎼 ), with
a value of '\u{1F3BC}', it too large to fit in 16 bits. 

{% prettify dart %}
var clef = '\u{1F3BC}'; // 🎼
{% endprettify %}

'\u{1F3BC}' is composed of a UTF-16 surrogate pair: [u\D83C, \uDFBC].

### What is the difference between a code point and a code unit?

Within the Basic Multilingual Plane, the code point for a character is
numerically the same as the code unit for that character.

{% prettify dart %}
'D'.runes.first; // 68
'D'.codeUnits.first; // 68 
{% endprettify %}

For non-BMP characters, each code point is represented by two code units.

{% prettify dart %}
var clef = '\u{1F3BC}'; // 🎼
clef.runes.length; // 1
clef.codeUnits.length; // 2
{% endprettify %}

### What exactly is a character?

A character is a string contained in the Universal Character Set. Each character
maps to a single rune value (code point); BMP characters map to 1 code
unit; non-BMP characters map to 2 code units. 

You can read more about the Universal Character Set at
http://en.wikipedia.org/wiki/Universal_Character_Set.

### Do I have to really deal with Unicode?

Yes, if you want to build robust international applications, you do.
Besides, the String library makes working with Unicode relatively painless,
so there's no great overhead in doing things right.

# Recipes

## Concatenating Strings

### Problem

You want to concatenate strings in Dart. You tried using `+`, but
that resulted in an error.

### Solution

Use adjacent string literals:

{% prettify dart %}
var fact = 'Dart'  'is' ' fun!'; // 'Dart is fun!'
{% endprettify %}
	
### Discussion

Adjacent literals also work over multiple lines:

{% prettify dart %}
var fact = 'Dart'
'is'
'fun!'; // 'Dart is fun!'
{% endprettify %}

They also work when using multiline strings:

{% prettify dart %}
var lunch = '''Peanut
butter'''
'''and
jelly'''; // 'Peanut\nbutter and\njelly'
{% endprettify %}
	
You can concatenate adjacent single line literals with multiline strings:

{% prettify dart %}
var funnyGuys = 'Dewey ' 'Cheatem'
''' and
Howe'''; // 'Dewey Cheatem and\n Howe'
{% endprettify %}


#### Alternatives to adjacent string literals

You can also use the `concat()` method on a string to concatenate it to another
string:

{% prettify dart %}
var film = filmToWatch();
film = film.concat('\n');  // 'The Big Lebowski\n' 
{% endprettify %}

Since `concat()` creates a new string every time it is invoked, a long chain of
`concat()`s can be expensive. Avoid those. Use a StringBuffer instead (see 
_Incrementally building a string efficiently using a StringBuffer_, below).

Use can `join()` to combine a sequence of strings:

{% prettify dart %}
var film = ['The', 'Big', 'Lebowski']).join(' '); // 'The Big Lebowski'
{% endprettify %}
	
You can also use string interpolation to concatenate strings (see 
_Interpolating expressions inside strings_, below).


## Interpolating expressions inside strings

### Problem

You want to create strings that contain Dart expressions and identifiers.

### Solution

You can put the value of an expression inside a string by using ${expression}.

{% prettify dart %}
var favFood = 'sushi';
var whatDoILove = 'I love ${favFood.toUpperCase()}'; // 'I love SUSHI'
{% endprettify %}

You can skip the {} if the expression is an identifier:

{% prettify dart %}
var whatDoILove = 'I love $favFood'; // 'I love sushi'
{% endprettify %}
      
### Discussion

An interpolated string, `string ${expression}` is equivalent to the
concatenation of the strings 'string ' and `expression.toString()`.
Consider this code:

{% prettify dart %}
var four = 4;
var seasons = 'The $four seasons'; // 'The 4 seasons'
{% endprettify %}
	
It is equivalent to the following:

{% prettify dart %}
var seasons = 'The '.concat(4.toString()).concat(' seasons'); // 'The 4 seasons'
{% endprettify %}

You should consider implementing a `toString()` method for user-defined
objects. Here's what happens if you don't:

{% prettify dart %}
class Point {
  num x, y;
  Point(this.x, this.y);
}

var point = new Point(3, 4);
print('Point: $point'); // "Point: Instance of 'Point'"
{% endprettify %}

Probably not what you wanted. Here is the same example with an explicit
`toString()`:

{% prettify dart %}
class Point {
  ...
  
  String toString() => 'x: $x, y: $y';
}

print('Point: $point'); // 'Point: x: 3, y: 4'
{% endprettify %}


## Escaping special characters

### Problem 

You want to put newlines, dollar signs, or other special characters in your strings.

### Solution

Prefix special characters with a `\`.

{% prettify dart %}
print(Wile\nCoyote'); 
// Wile
// Coyote
{% endprettify %}

### Discussion

Dart designates a few characters as special, and these can be escaped:

- \n for newline, equivalent to \x0A.
- \r for carriage return, equivalent to \x0D.
- \f for form feed, equivalent to \x0C.
- \b for backspace, equivalent to \x08.
- \t for tab, equivalent to \x09.
- \v for vertical tab, equivalent to \x0B.

If you prefer, you can use `\x` or `\u` notation to indicate the special
character:

{% prettify dart %}
print('Wile\x0ACoyote');       // same as print('Wile\nCoyote'); 
print('Wile\u000ACoyote');     // same as print('Wile\nCoyote'); 
{% endprettify %}

You can also use `\u{}` notation:

{% prettify dart %}
print('Wile\u{000A}Coyote'); // same as print('Wile\nCoyote'); 
{% endprettify %}

You can also escape the `$` used in string interpolation:

{% prettify dart %}
var superGenius = 'Wile Coyote';
print('$superGenius and Road Runner');  // 'Wile Coyote and Road Runner'
print('\$superGenius and Road Runner'); // '$superGenius and Road Runner'
{% endprettify %}


If you escape a non-special character, the `\` is ignored:
 
{% prettify dart %}
print('Wile \E Coyote'); // 'Wile E Coyote'
{% endprettify %}

	
## Incrementally building a string efficiently using a StringBuffer

### Problem

You want to collect string fragments and combine them in an efficient manner.

### Solution

Use a StringBuffer to programmatically generate a string. A StringBuffer 
collects the string fragments, but does not generate a new string until
`toString()` is called:

{% prettify dart %}
var sb = new StringBuffer();
sb.write('John, ');
sb.write('Paul, ');
sb.write('George, ');
sb.write('and Ringo');
var beatles = sb.toString(); // 'John, Paul, George, and Ringo'
{% endprettify %}
    
### Discussion

In addition to `write()`, the StringBuffer class provides methods to write a
list of strings (`writeAll()`), write a numerical character code
(`writeCharCode()`), write with an added newline ('writeln()`), and more. Here
is a simple example that show the use of these methods:

{% prettify dart %}
var sb = new StringBuffer();
sb.writeln('The Beatles:');
sb.writeAll(['John, ', 'Paul, ', 'George, and Ringo']);
sb.writeCharCode(33); // charCode for '!'.
var beatles = sb.toString(); // 'The Beatles:\nJohn, Paul, George, and Ringo!'

{% endprettify %}

Since a StringBuffer waits until the call to `toString()` to generate the
concatenated string,  it represents a more efficient way of combining strings
than `concat()`.  See the _Concatenating Strings_ recipe for a description of
`concat()`.

## Converting between string characters and numerical codes

### Problem 

You want to convert string characters into numerical codes and back.

### Solution

Use the `runes` getter to access a string's code points:

{% prettify dart %}
'Dart'.runes.toList(); // [68, 97, 114, 116]

var smileyFace = '\u263A'; // ☺
smileyFace.runes.toList(); // [9786]
{% endprettify %}
     
The number 9786 represents the code unit '\u263A'.

Use `string.codeUnits` to get a string's UTF-16 code units:
    
{% prettify dart %}
'Dart'.codeUnits.toList(); // [68, 97, 114, 116]
smileyFace.codeUnits.toList(); // [9786]
{% endprettify %}
 
### Discussion

Notice that using `runes` and `codeUnits` produces identical results
in the examples above. That happens because each character in 'Dart' and in
`smileyFace` fits within 16 bits, resulting in a code unit corresponding
neatly with a code point.

Consider an example where a character cannot be represented within 16-bits,
the Unicode character for a Treble clef ('\u{1F3BC}'). This character consists
of a surrogate pair: '\uD83C', '\uDFBC'. Getting the numerical value of this
character using `codeUnits` and `runes` produces the following result:

{% prettify dart %}
var clef = '\u{1F3BC}'; // 🎼 
clef.codeUnits.toList(); // [55356, 57276]
clef.runes.toList(); // [127932]
{% endprettify %}

The numbers 55356 and 57276 represent `clef`'s surrogate pair, '\uD83C' and
'\uDFBC', respectively. The number 127932 represents the code point '\u1F3BC'.

#### Using codeUnitAt() to access individual code units

To access the 16-Bit UTF-16 code unit at a particular index, use
`codeUnitAt()`:

{% prettify dart %}
'Dart'.codeUnitAt(0); // 68
smileyFace.codeUnitAt(0); // 9786
{% endprettify %}
    
Using `codeUnitAt()` with the multi-byte `clef` character leads to problems:

{% prettify dart %}
clef.codeUnitAt(0); // 55356
clef.codeUnitAt(1); // 57276
{% endprettify %}

In either call to `clef.codeUnitAt()`, the values returned represent strings
that are only one half of a UTF-16 surrogate pair.  These are not valid UTF-16
strings.


#### Converting numerical codes to strings

You can generate a new string from runes or code units using the factory 
`String.fromCharCodes(charCodes)`:

{% prettify dart %}
new String.fromCharCodes([68, 97, 114, 116]); // 'Dart'
	
new String.fromCharCodes([73, 32, 9825, 32, 76, 117, 99, 121]);
// 'I ♡ Lucy'

new String.fromCharCodes([55356, 57276]); // 🎼  
new String.fromCharCodes([127932]), // 🎼  
{% endprettify %}

You can use the `String.fromCharCode()` factory to convert a single rune or
code unit to a string:

{% prettify dart %}
new String.fromCharCode(68); // 'D'
new String.fromCharCode(9786); // ☺
new String.fromCharCode(127932); // 🎼  
{% endprettify %}

Creating a string with only one half of a surrogate pair is permitted, but not
recommended.

## Determining if a string is empty

### Problem

You want to know if a string is empty. You tried ` if(string) {...}`, but that
did not work.

### Solution

Use `string.isEmpty`:

{% prettify dart %}
    var emptyString = '';
  	emptyString.isEmpty; // true
{% endprettify %}
  	
A string with a space is not empty:
 
{% prettify dart %}
    var space = ' ';
    space.isEmpty; // false
{% endprettify %}
   
### Discussion 

Don't use `if (string)` to test the emptiness of a string. In Dart, all
objects except the boolean true evaluate to false. `if(string)` will always
be false.


## Removing leading and trailing whitespace

### Problem

You want to remove leading and trailing whitespace from a string.

### Solution

Use `string.trim()`:

{% prettify dart %}
var space = '\n\r\f\t\v';  // We'll use a variety of space characters.
var string = '$space X $space';
var newString = string.trim(); // 'X'
{% endprettify %}

The String class has no methods to remove only leading or only trailing
whitespace. But you can always use regExps.

Remove only leading whitespace:

{% prettify dart %}
var newString = string.replaceFirst(new RegExp(r'^\s+'), ''); //  'X $space'
{% endprettify %}
   
Remove only trailing whitespace:

{% prettify dart %}
var newString = string.replaceFirst(new RegExp(r'\s+$'), ''); // '$space X'
{% endprettify %}


## Calculating the length of a string

### Problem

You want to get the length of a string, but are not sure how to 
correctly calculate the length when working with Unicode.

### Solution

Use string.length to get the number of UTF-16 code units in a string:
	
{% prettify dart %}
'I love music'.length; // 12
'I love music'.runes.length; // 12
{% endprettify %}
	
### Discussion

For characters that fit into 16 bits, the code unit length is the same as the
rune length:

{% prettify dart %}
var hearts = '\u2661'; // ♡
hearts.length; // 1
hearts.runes.length; // 1
{% endprettify %}
  
If the string contains any characters outside the Basic Multilingual
Plane (BMP), the rune length will be less than the code unit length:
  
{% prettify dart %}
var clef = '\u{1F3BC}'; // 🎼  
clef.length; // 2
clef.runes.length; // 1

var music = 'I $hearts $clef'; // 'I ♡ 🎼 '
music.length; // 6
music.runes.length // 5
{% endprettify %}

Use `length` if you want to number of code units; use `runes.length` if you 
want the number of runes.


## Subscripting a string

### Problem

You want to be able to access a character in a string at a particular index.

### Solution

Subscript runes:

{% prettify dart %}
var teacup = '\u{1F375}'; // 🍵  
teacup.runes.toList()[0]; // 127861
{% endprettify %}
    
The number 127861 represents the code point for teacup, '\u{1F375}' (🍵 ). 

### Discussion

Subscripting a string directly can be problematic. This is because the default 
`[]` implementation subscripts along code units.  This means that
for non-BMP characters, subscripting yields invalid UTF-16 characters: 

{% prettify dart %}
'Dart'[0]; // 'D'

var hearts = '\u2661'; // ♡
hearts[0]; '\u2661' // ♡

teacup[0]; // 55356, Invalid string, half of a surrogate pair.
teacup.codeUnits.toList()[0]; // The same.
{% endprettify %}
    

## Processing a string one character at a time

### Problem

You want to do something with each individual character in a string.

### Solution

To access an individual character, map the string runes:

{% prettify dart %}
var charList = "Dart".runes.map((rune) => '*${new String.fromCharCode(rune)}*').toList();
      // ['*D*', '*a*', '*r*', '*t*']

var runeList = happy.runes.map((rune) => [rune, new String.fromCharCode(rune)]).toList(), 
          // [[73, 'I'], [32, ' '], [97, 'a'], [109, 'm'], [32, ' '], [9786, '☺']]
      
{% endprettify %}

If you are sure that the string is in the Basic Multilingual Plane (BMP), you
can use string.split(''):

{% prettify dart %}
'Dart'.split(''); // ['D', 'a', 'r', 't']
smileyFace.split('').length; // 1
{% endprettify %}
	
Since `split('')` splits at the UTF-16 code unit boundaries,
invoking it on a non-BMP character yields the string's surrogate pair:

{% prettify dart %}
var clef = '\u{1F3BC}'; // 🎼 , not in BMP.
clef.split('').length; // 2
{% endprettify %}
	
The surrogate pair members are not valid UTF-16 strings.


## Splitting a string into substrings

### Problem

You want to split a string into substrings.

### Solution

Use the `split()` method with a string or a regExp as an argument. 

{% prettify dart %}
var smileyFace = '\u263A';
var happy = 'I am $smileyFace';
happy.split(' '); // ['I', 'am', '☺']
{% endprettify %}
    
Here is an example of using `split()` with a regExp:

{% prettify dart %}
var nums = '2/7 3 4/5 3~/5';
var numsRegExp = new RegExp(r'(\s|/|~/)');
nums.split(numsRegExp); // ['2', '7', '3', '4', '5', '3', '5']
{% endprettify %}

In the code above, the string `nums` contains various numbers, some of which
are expressed as fractions or as int-divisions. A regExp is used to split the
string to extract just the numbers.
 
You can perform operations on the matched and unmatched portions of a string
when using `split()` with a regExp:

{% prettify dart %}
'Eats SHOOTS leaves'.splitMapJoin((new RegExp(r'SHOOTS')),
  onMatch: (m) => '*${m.group(0).toLowerCase()}*',
  onNonMatch: (n) => n.toUpperCase()); // 'EATS *shoots* LEAVES'
{% endprettify %}
      
The regExp matches the middle word ('SHOOTS'). A pair of callbacks are
registered to transform the matched and unmatched substrings before the
substrings are joined together again.


## Changing string case

### Problem

You want to change the case of strings.

### Solution

Use `string.toUpperCase()` and `string.toLowerCase()` to convert a string to 
lower-case or upper-case, respectively:

{% prettify dart %}
var theOneILove = 'I love Lucy';
theOneILove.toUpperCase(); // 'I LOVE LUCY!'
theOneILove.toLowerCase(); // 'i love lucy!'
{% endprettify %}
 
### Discussion
 
Case changes affect the characters of bi-cameral scripts like Greek and French:


{% prettify dart %}
var zeus = '\u0394\u03af\u03b1\u03c2'; // 'Δίας' (Zeus in modern Greek)
zeus.toUpperCase(); // 'ΔΊΑΣ'

var resume = '\u0052\u00e9\u0073\u0075\u006d\u00e9'; // 'Résumé'
resume.toLowerCase(); // 'résumé'
{% endprettify %}
    
They do not affect the characters of uni-cameral scripts like Devanagari (used for
writing many of the languages of India):

{% prettify dart %}
var chickenKebab = '\u091a\u093f\u0915\u0928 \u0915\u092c\u093e\u092c'; 
// 'चिकन कबाब'  (in Devanagari)
chickenKebab.toLowerCase();  // 'चिकन कबाब'
chickenKebab.toUpperCase();  // 'चिकन कबाब'
{% endprettify %}

If a character's case does not change when using `toUpperCase()` and
`toLowerCase()`, it is most likely because the character only has one
form.

## Determining whether a string contains another string

### Problem

You want to find out if a string is the substring of another string.

### Solution

Use `string.contains()`:

{% prettify dart %}
var fact = 'Dart strings are immutable';
string.contains('immutable'); // True.
{% endprettify %}

You can indicate a startIndex as a second argument:

{% prettify dart %}
string.contains('Dart', 2); // False
{% endprettify %}
    
### Discussion

The String library provides a couple of shortcuts for testing whether a string
is a substring of another:

{% prettify dart %}
string.startsWith('Dart'); // True.
string.endsWith('e'); // True.
{% endprettify %}
    
	
You can also use `string.indexOf()`, which returns -1 if the substring is
not found within a string, and its matching index, if it is:

{% prettify dart %}
string.indexOf('art') != -1; // True, `art` is found in `Dart`
{% endprettify %}

You can also use a regExp and `hasMatch()`:

{% prettify dart %}
new RegExp(r'ar[et]').hasMatch(string); //  True, 'art' and 'are' match.
{% endprettify %}
    

## Finding matches of a regExp pattern in a string

### Problem

You want to use regExp to match a pattern in a string, and 
want to be able to access the matches.

### Solution

Construct a regular expression using the RegExp class and find matches using
the `allMatches()` method:

{% prettify dart %}
var neverEatingThat = 'Not with a fox, not in a box';
var regExp = new RegExp(r'[fb]ox');
List matches = regExp.allMatches(neverEatingThat);
matches.map((match) => match.group(0)).toList(); // ['fox', 'box']
{% endprettify %}
    
### Discussion
     
You can query the object returned by `allMatches()` to find out the number of
matches:
 
{% prettify dart %}
matches.length; // 2
{% endprettify %}

To find the first match, use `firstMatch()`:
	
{% prettify dart %}
regExp.firstMatch(neverEatingThat).group(0); // 'fox'
{% endprettify %}
	
To directly access the matched string, use `stringMatch()`:

{% prettify dart %}
regExp.stringMatch(neverEatingThat); // 'fox'
regExp.stringMatch('I like bagels and lox'); // null
{% endprettify %}

## Substituting strings based on regExp matches

### Problem

You want to match substrings within a string and make substitutions based on
the matches.

### Solution

Construct a regular expression using the RegExp class and make replacements
using `replaceAll()` method:

{% prettify dart %}
'resume'.replaceAll(new RegExp(r'e'), '\u00E9'); // 'résumé'
{% endprettify %}

If you want to replace just the first match, use 'replaceFirst()`:

{% prettify dart %}
'0.0001'.replaceFirst(new RegExp(r'0+'), ''); // '.0001'
{% endprettify %}
	
The RegExp matches for one or more 0's and replaces them with an empty string.

You can use `replaceAllMatched()` and register a function to modify the
matches:

{% prettify dart %}
var heart = '\u2661'; // '♡'
var string = 'I like Ike but I $heart Lucy';
var regExp = new RegExp(r'[A-Z]\w+');
string.replaceAllMapped(regExp, (match) => match.group(0).toUpperCase()); 
// 'I like IKE but I ♡ LUCY'
{% endprettify %}
