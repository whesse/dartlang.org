---
layout: default
title: "Dart Cookbook"
description: "Recipes and prescriptions for using Dart."
has-permalinks: true
---

# Dart Cookbook

## Contents

1. [Strings](#strings)
    1. [Concatenating Strings](#concatenating-strings)
    1. [Interpolating expressions inside strings](#interpolating-expressions-inside-strings)
    1. [Handling special characters within strings](#handling-special-characters-within-strings)
    1. [Incrementally building a string using a StringBuffer](#incrementally-building-a-string-using-a-stringbuffer)
    1. [Determining whether a string is empty](#determining-whether-a-string-is-empty)
    1. [Removing leading and trailing whitespace](#removing-leading-and-trailing-whitespace)
    1. [Changing string case](#changing-string-case)
    1. [Handling extended characters that are composed of multiple code units](#handling-extended-characters-that-are-composed-of-multiple-code-units)
    1. [Converting between characters and numerical codes](#converting-between-characters-and-numerical-codes)
    1. [Calculating the length of a string](#calculating-the-length-of-a-string)
    1. [Processing a string one character at a time](#processing-a-string-one-character-at-a-time)
    1. [Splitting a string into substrings](#splitting-a-string-into-substrings)
    1. [Determining whether a string contains another string](#determining-whether-a-string-contains-another-string)
    1. [Finding matches of a RegExp pattern in a string](#finding-matches-of-a-regexp-pattern-in-a-string)
    1. [Substituting strings based on RegExp matches](#substituting-strings-based-on-regexp-matches)
{:.toc}



## Strings

### Concatenating Strings

#### Problem

You want to concatenate strings in Dart. You tried using `+`, but that
resulted in an error.

#### Solution

Use adjacent string literals:

<pre class="programlisting">
var fact = 'Dart'  'is' ' fun!'; // 'Dart is fun!'
</pre>

#### Discussion

Adjacent literals work over multiple lines:

<pre class="programlisting">
var fact = 'Dart'
'is'
'fun!'; // 'Dart is fun!'
</pre>

They also work when using multiline strings:

<pre class="programlisting">
var lunch = '''Peanut
butter'''
'''and
jelly'''; // 'Peanut\nbutter and\njelly'
</pre>

You can concatenate adjacent single line literals with multiline
strings:

<pre class="programlisting">
var funnyGuys = 'Dewey ' 'Cheatem'
''' and
Howe'''; // 'Dewey Cheatem and\n Howe'
</pre>

##### Alternatives to adjacent string literals

You can use the `concat()` method on a string to concatenate it to
another string:

<pre class="programlisting">
var film = filmToWatch();
film = film.concat('\n');  // 'The Big Lebowski\n' 
</pre>

Because `concat()` creates a new string every time it is invoked, a long
chain of `concat()` s can be expensive. Avoid those. Use a StringBuffer
instead (see _Incrementally building a string efficiently using a
StringBuffer_, below).

Use `join()` to combine a sequence of strings:

<pre class="programlisting">
var film = ['The', 'Big', 'Lebowski']).join(' '); // 'The Big Lebowski'
</pre>

You can also use string interpolation to concatenate strings (see
_Interpolating expressions inside strings_, below).

### Interpolating expressions inside strings

#### Problem

You want to embed Dart code inside strings.

#### Solution

You can put the value of an expression inside a string by using
$\{expression}.

<pre class="programlisting">
var favFood = 'sushi';
var whatDoILove = 'I love ${favFood.toUpperCase()}'; // 'I love SUSHI'
</pre>

You can skip the \{} if the expression is an identifier:

<pre class="programlisting">
var whatDoILove = 'I love $favFood'; // 'I love sushi'
</pre>

#### Discussion

An interpolated string, `'string ${expression}` is equivalent to the
concatenation of the strings `string` and `expression.toString()`.
Consider this code:

<pre class="programlisting">
var four = 4;
var seasons = 'The $four seasons'; // 'The 4 seasons'
</pre>

This is functionally equivalent to the following:

<pre class="programlisting">
var seasons = 'The '.concat(4.toString()).concat(' seasons');
// 'The 4 seasons'
</pre>

You should consider implementing a `toString()` method for user-defined
objects. Here's what happens if you don't:

<pre class="programlisting">
class Point {
  num x, y;
  Point(this.x, this.y);
}

var point = new Point(3, 4);
print('Point: $point'); // "Point: Instance of 'Point'"
</pre>

Probably not what you wanted. Here is the same example with an explicit
`toString()`:

<pre class="programlisting">
class Point {
  ...
    
  String toString() => 'x: $x, y: $y';
}

print('Point: $point'); // 'Point: x: 3, y: 4'
</pre>

### Handling special characters within strings

#### Problem

You want to put newlines, dollar signs, or other special characters in strings.

#### Solution

Prefix special characters with a `\`.

<pre class="programlisting">
  print(Wile\nCoyote'); 
  // Wile
  // Coyote
</pre>

#### Discussion

Dart designates a few characters as special, and these can be escaped:

* \n for newline, equivalent to \x0A.
* \r for carriage return, equivalent to \x0D.
* \f for form feed, equivalent to \x0C.
* \b for backspace, equivalent to \x08.
* \t for tab, equivalent to \x09.
* \v for vertical tab, equivalent to \x0B.

If you prefer, you can use `\x` or `\u` notation to indicate the special
character:

<pre class="programlisting">
print('Wile\x0ACoyote');   // Same as print('Wile\nCoyote')
print('Wile\u000ACoyote'); // Same as print('Wile\nCoyote') 
</pre>

You can also use `\u{}` notation:

<pre class="programlisting">
print('Wile\u{000A}Coyote'); // same as print('Wile\nCoyote')
</pre>

You can also escape the `$` used in string interpolation:

<pre class="programlisting">
var superGenius = 'Wile Coyote';
print('$superGenius and Road Runner');  // 'Wile Coyote and Road Runner'
print('\$superGenius and Road Runner'); // '$superGenius and Road Runner'
</pre>

If you escape a non-special character, the `\` is ignored:

<pre class="programlisting">
print('Wile \E Coyote'); // 'Wile E Coyote'
</pre>


### Incrementally building a string using a StringBuffer

#### Problem

You want to collect string fragments and combine them in an efficient
manner.

#### Solution

Use a StringBuffer to programmatically generate a string. Consider this code
below for assembling a series of urls from fragments:

<pre class="programlisting">
var data = [{'scheme': 'https', 'domain': 'news.ycombinator.com'}, 
            {'domain': 'www.google.com'}, 
            {'domain': 'reddit.com', 'path': 'search', 'params': 'q=dart'}
           ];

String assembleUrlsUsingStringBuffer(entries) {
  StringBuffer sb = new StringBuffer();
  for (final item in entries) {
    sb.write(item['scheme'] != null ? item['scheme']  : 'http');
    sb.write("://");
    sb.write(item['domain']);
    sb.write('/');
    sb.write(item['path'] != null ? item['path']  : '');
    if (item['params'] != null) {
      sb.write('?');
      sb.write(item['params']);
    }
    sb.write('\n');
  }
  return sb.toString();
}

// https://news.ycombinator.com/
// http://www.google.com/
// http://reddit.com/search?q=dart
</pre>

A StringBuffer collects string fragments, but does not generate a new string
until `toString()` is called. 

#### Discussion

Using a StringBuffer is vastly more efficient than concatenating fragments
at each step: Consider this rewrite of the above code:

<pre class="programlisting">
String assembleUrlsUsingConcat(entries) {
  var urls = '';
  for (final item in entries) {
    urls = urls.concat(item['scheme'] != null ? item['scheme']  : 'http');
    urls = urls.concat("://");
    urls = urls.concat(item['domain']);
    urls = urls.concat('/');
    urls = urls.concat(item['path'] != null ? item['path']  : '');
    if (item['params'] != null) {
      urls = urls.concat('?');
      urls = urls.concat(item['params']);
    }
    urls = urls.concat('\n');
  }
  return urls;
}
</pre>

This approach produces the exact same result, but incurs the cost of
joining strings multiple times. 

See the _Concatenating Strings_ recipe for a description of `concat()`.

##### Other StringBuffer methods

In addition to `write()`, the StringBuffer class provides methods to
write a list of strings (`writeAll()`), write a numerical character code
(`writeCharCode()`), write with an added newline (`writeln()`), and
more. The example below shows how to use these methods:

<pre class="programlisting">
var sb = new StringBuffer();
sb.writeln('The Beatles:');
sb.writeAll(['John, ', 'Paul, ', 'George, and Ringo']);
sb.writeCharCode(33); // charCode for '!'.
var beatles = sb.toString(); // 'The Beatles:\nJohn, Paul, George, and Ringo!' 
</pre>


### Determining whether a string is empty

#### Problem

You want to know whether a string is empty. You tried `if (string) {...}`, but
that did not work.

#### Solution

Use `string.isEmpty`:

<pre class="programlisting">
var emptyString = '';
print(emptyString.isEmpty); // true
</pre>

You can also just use `==`:

<pre class="programlisting">
if (string == '') {...} // True if string is empty.
</pre>

A string with a space is not empty:

<pre class="programlisting">
var space = ' ';
print(space.isEmpty); // false
</pre>

#### Discussion

Don't use `if (string)` to test the emptiness of a string. In Dart, all objects
except the boolean true evaluate to false, so `if(string)` is always false. You
will see a warning in the editor if you use an 'if' statement with a non-boolean
in checked mode.


### Removing leading and trailing whitespace

#### Problem

You want to remove spaces, tabs, and other whitespace from the beginning and
end of strings.

#### Solution

Use `string.trim()`:

<pre class="programlisting">
var space = '\n\r\f\t\v';       // A variety of space characters.
var string = '$space X $space';
var newString = string.trim();  // 'X'
</pre>

The String class has no methods to remove only leading or only trailing
whitespace. You can always use a RegExp.

Remove only leading whitespace:

<pre class="programlisting">
var newString = string.replaceFirst(new RegExp(r'^\s+'), ''); // 'X \n\r\f\t\v'
</pre>

Remove only trailing whitespace:

<pre class="programlisting">
var newString = string.replaceFirst(new RegExp(r'\s+$'), ''); // '\n\r\f\t\v X'
</pre>


### Changing string case

#### Problem

You want to change the case of strings.

#### Solution

Use String's `toUpperCase()` and `toLowerCase()` methods: 

<pre class="programlisting">
var theOneILove = 'I love Lucy';
theOneILove.toUpperCase();                           // 'I LOVE LUCY!'
theOneILove.toLowerCase();                           // 'i love lucy!'

// Zeus in modern Greek.
var zeus = '\u0394\u03af\u03b1\u03c2';               // 'Δίας'
zeus.toUpperCase();                                  // 'ΔΊΑΣ'

var resume = '\u0052\u00e9\u0073\u0075\u006d\u00e9'; // 'Résumé'
resume.toLowerCase();                                // 'résumé'
</pre>

The `toUpperCase()` and `toLowerCase()` methods don't affect the characters of
scripts such as Devanagri that don't have distinct letter cases.

<pre class="programlisting">
var chickenKebab = '\u091a\u093f\u0915\u0928 \u0915\u092c\u093e\u092c'; 
// 'चिकन कबाब'  (in Devanagari)
chickenKebab.toLowerCase();  // 'चिकन कबाब'
chickenKebab.toUpperCase();  // 'चिकन कबाब'
</pre>

If a character's case does not change when using `toUpperCase()` and
`toLowerCase()`, it is most likely because the character only has one
form.


### Handling extended characters that are composed of multiple code units

#### Problem

You want to use emoticons and other special symbols that don't fit into 16
bits. How can you create such strings and use them correctly in your code? 

#### Solution

You can create an extended character using `'\u{}'` syntax:

<pre class="programlisting">
var clef = '\u{1F3BC}'; // 🎼 
</pre>

#### Discussion

Most UTF-16 strings are stored as two-byte (16 bit) code sequences.
Since two bytes can only contain the 65,536 characters in the 0x0 to 0xFFFF
range, a pair of strings is used to store values in the 0x10000 to 0x10FFFF
range. These strings only have semantic meaning as a pair. Individually, they
are invalid UTF-16 strings. The term 'surrogate pair' is often used to
describe these strings. 

The clef glyph `'\u{1F3BC}'` is composed of the `'\uD83C'` and `'\uDFBC'`
surrogate pair.

You can get an extended string's surrogate pair through its `codeUnits`
property:

<pre class="programlisting">
clef.codeUnits.map((codeUnit) => codeUnit.toRadixString(16)); 
// ['d83c', 'dfbc']
</pre>

Accessing a surrogate pair member leads to errors, and you should avoid
properties and methods that expose it:

<pre class="programlisting">
print('\ud83c'); // Error: '\ud83c' is not a valid string.
print('\udfbc'); // Error: '\udfbc' is not a valid string either.
clef.split()[1]; // Error: accessing half of a surrogate pair.
print(clef[i];)  // Again, error: accessing half of a surrogate pair.
</pre>

When dealing with strings containing extended characters, you should use the
`runes` getter.

To get the string's length, use `string.runes.length`. Don't use
`string.length`:

<pre class="programlisting">
print(clef.runes.length);     // 1
print(clef.length);           // 2
print(clef.codeUnits.length); // 2
</pre>

To get an individual character or its numeric equivalent, index the rune list:

<pre class="programlisting">
print(clef.runes.toList()[0]); // 127932 ('\u{1F3BC}')
</pre>

To get the string's characters as a list, map the string runes:

<pre class="programlisting">
var clef = '\u{1F3BC}'; // 🎼 
var title = '$clef list:'
print(subject.runes.map((rune) => new String.fromCharCode(rune)).toList());
// ['🎼', ' ', 'l', 'i', 's', 't', ':']
</pre>


### Converting between characters and numerical codes

#### Problem

You want to convert string characters into numerical codes and vice versa.
You want to do this because sometimes you need to compare characters in a string
to numerical values coming from another source. Or, maybe you want to split a
string and then operate on each character.

#### Solution

Use the `runes` getter to get a string's code points:

<pre class="programlisting">
'Dart'.runes.toList();            // [68, 97, 114, 116]

var smileyFace = '\u263A';        // ☺
print(smileyFace.runes.toList()); // [9786], (equivalent to ['\u263A']).

var clef = '\u{1F3BC}';           // 🎼 
print(clef.runes.toList());       // [127932], (equivalent to ['\u{1F3BC}']).
</pre>

Use `string.codeUnits` to get a string's UTF-16 code units:

<pre class="programlisting">
'Dart'.codeUnits.toList();     // [68, 97, 114, 116]
smileyFace.codeUnits.toList(); // [9786]
clef.codeUnits.toList();       // [55356, 57276]
</pre>

##### Using codeUnitAt() to get individual code units

To get the code unit at a particular index, use `codeUnitAt()`:

<pre class="programlisting">
'Dart'.codeUnitAt(0);     // 68
smileyFace.codeUnitAt(0); // 9786 (the decimal value of '\u263A')
clef.codeUnitAt(0);       // 55356 (does not represent a legal string) 
</pre>

#### Converting numerical codes to strings

You can generate a new string from numerical codes using the factory
`String.fromCharCodes(charCodes)`. You can pass either runes or code units and
`String.fromCharCodes(charCodes)` can tell the difference and do the right
thing automatically:

<pre class="programlisting">
print(new String.fromCharCodes([68, 97, 114, 116]));                  // 'Dart'

print(new String.fromCharCodes([73, 32, 9825, 32, 76, 117, 99, 121]));
// 'I ♡ Lucy'

// Passing code units representing the surrogate pair.
print(new String.fromCharCodes([55356, 57276]));                      // 🎼  

// Passing runes.
print(new String.fromCharCodes([127932]));                            // 🎼  
</pre>

You can use the `String.fromCharCode()` factory to convert a single rune
or code unit to a string:

<pre class="programlisting">
new String.fromCharCode(68);     // 'D'
new String.fromCharCode(9786);   // ☺
new String.fromCharCode(127932); // 🎼  
</pre>

Creating a string with only one half of a surrogate pair is permitted,
but not recommended.


### Calculating the length of a string

#### Problem

You want to get the length of a string, but are not sure how to calculate the
length correctly when working with variable length Unicode characters.

#### Solution

Use `string.runes.length` to get the number of characters in a string.

<pre class="programlisting">
print('I love music'.runes.length); // 12
</pre>

You can safely use `string.runes.length` to get the length of strings that
contain extended characters:

<pre class="programlisting">
var clef = '\u{1F3BC}';        // 🎼  
var subject = '$clef list:';   // 
var music = 'I $hearts $clef'; // 'I ♡ 🎼 '

clef.runes.length;             // 1
music.runes.length             // 5
</pre>

#### Discussion

You can directly use a string's `length` property (minus `runes`). This returns
the string's code unit length. Using `string.length` produces the same length
as `string.runes.length` for most unicode characters.

For extended characters, the code unit length is one more than the rune
length:

<pre class="programlisting">
clef.length;                   // 2

var music = 'I $hearts $clef'; // 'I ♡ 🎼 '
music.length;                  // 6
</pre>

Unless you specifically need the code unit length of a string, use
`string.runes.length`.

##### Working with combined characters

It is tempting to brush aside the complexity involved in dealing with runes and
code units and base the length of the string on the number of characters it
appears to have. Anyone can tell that 'Dart' has four characters, and 'Amelié'
has six, right? Almost. The length of 'Dart' is indeed four, but the length of
'Amelié' depends on how that string was constructed:

<pre class="programlisting">
var name = 'Ameli\u00E9';               // 'Amelié'
var anotherName = 'Ameli\u0065\u0301';  // 'Amelié'
print(name.length);                     // 6
print(anotherName.length);              // 7
</pre>

Both `name` and `anotherName` return strings that look the same, but where
the 'é' is constructed using a different number of runes. This makes it
impossible to know the length of these strings by just looking at them.


### Processing a string one character at a time

#### Problem

You want to do something with each character in a string.

#### Solution

Map the results of calling `string.split('')`:

<pre class="programlisting">
var lang= 'Dart';

// ['*D*', '*a*', '*r*', '*t*']
print(lang.split('').map((char) => '*${char}*').toList());

var smileyFace = '\u263A';
var happy = 'I am $smileyFace';
print(happy.split('')); // ['I', ' ', 'a', 'm', ' ', '☺']
</pre>

Or, loop over the characters of a string:

<pre class="programlisting">
var list = [];
for(var i = 0; i < lang.length; i++) {
  list.add('*${lang[i]}*'); 
}

print(list); // ['*D*', '*a*', '*r*', '*t*']
</pre>

Or, map the string runes:

<pre class="programlisting">
// ['*D*', '*a*', '*r*', '*t*']
var charList = "Dart".runes.map((rune) {
  return '*${new String.fromCharCode(rune)}*').toList();
});

// [[73, 'I'], [32, ' '], [97, 'a'], [109, 'm'], [32, ' '], [9786, '☺']]
var runeList = happy.runes.map((rune) {
  return [rune, new String.fromCharCode(rune)]).toList();
});
</pre>

When working with extended characters, you should always map the string runes.
Don't use `split('')` and avoid indexing an extended string. See the _Handling
extended characters that are composed of multiple code units_ recipe for
special considerations when working with extended strings.


### Splitting a string into substrings

#### Problem

You want to split a string into substrings using a delimiter or a pattern.

#### Solution

Use the `split()` method with a string or a RegExp as an argument.

<pre class="programlisting">
var smileyFace = '\u263A';
var happy = 'I am $smileyFace';
happy.split(' '); // ['I', 'am', '☺']
</pre>

Here is an example of using `split()` with a RegExp:

<pre class="programlisting">
var nums = '2/7 3 4/5 3~/5';
var numsRegExp = new RegExp(r'(\s|/|~/)');
nums.split(numsRegExp); // ['2', '7', '3', '4', '5', '3', '5']
</pre>

In the code above, the string `nums` contains various numbers, some of which
are expressed as fractions or as int-divisions. A RegExp splits the string to
extract just the numbers.

You can perform operations on the matched and unmatched portions of a string
when using `split()` with a RegExp:

<pre class="programlisting">
var phrase = 'Eats SHOOTS leaves';

var newPhrase = phrase.splitMapJoin((new RegExp(r'SHOOTS')),
  onMatch:    (m) => '*${m.group(0).toLowerCase()}*',
  onNonMatch: (n) => n.toUpperCase());

print(newPhrase); // 'EATS *shoots* LEAVES'
  
</pre>

The RegExp matches the middle word ('SHOOTS'). A pair of callbacks are
registered to transform the matched and unmatched substrings before the
substrings are joined together again.


### Determining whether a string contains another string

#### Problem

You want to find out whether a string is the substring of another string.

#### Solution

Use `string.contains()`:

<pre class="programlisting">
var fact = 'Dart strings are immutable';
print(fact.contains('immutable')); // True.
</pre>

You can use a second argument to specify where in the string to start looking:

<pre class="programlisting">
print(fact.contains('Dart', 2)); // False
</pre>

#### Discussion

The String class provides a couple of shortcuts for testing whether a
string is a substring of another:

<pre class="programlisting">
print(string.startsWith('Dart')); // True.
print(string.endsWith('e'));      // True.
</pre>

You can also use `string.indexOf()`, which returns -1 if the substring
is not found within a string, and otherwise returns the matching index:

<pre class="programlisting">
var found = string.indexOf('art') != -1; // True, `art` is found in `Dart`.
</pre>

You can also use a RegExp and `hasMatch()`:

<pre class="programlisting">
var found = new RegExp(r'ar[et]').hasMatch(string);
//  True, 'art' and 'are' match.
</pre>

### Finding matches of a RegExp pattern in a string

#### Problem

You want to use RegExp to match a pattern in a string, and want to be
able to access the matches.

#### Solution

Construct a regular expression using the RegExp class, and find matches
using the `allMatches()` method:

<pre class="programlisting">
var neverEatingThat = 'Not with a fox, not in a box';
var regExp = new RegExp(r'[fb]ox');
List matches = regExp.allMatches(neverEatingThat);
print(matches.map((match) => match.group(0)).toList()); // ['fox', 'box']
</pre>

#### Discussion

You can query the object returned by `allMatches()` to find out the
number of matches:

<pre class="programlisting">
var howManyMatches = matches.length; // 2
</pre>

To find the first match, use `firstMatch()`:

<pre class="programlisting">
var firstMatch = RegExp.firstMatch(neverEatingThat).group(0); // 'fox'
</pre>

To directly get the matched string, use `stringMatch()`:

<pre class="programlisting">
print(regExp.stringMatch(neverEatingThat));         // 'fox'
print(regExp.stringMatch('I like bagels and lox')); // null
</pre>

### Substituting strings based on RegExp matches

#### Problem

You want to match substrings within a string and make substitutions
based on the matches.

#### Solution

Construct a regular expression using the RegExp class and make
replacements using `replaceAll()` method:

<pre class="programlisting">
var resume = 'resume'.replaceAll(new RegExp(r'e'), '\u00E9'); // 'résumé'
</pre>

If you want to replace just the first match, use `replaceFirst()`:

<pre class="programlisting">
// Replace the first match of one or more zeros with an empty string.
var smallNum = '0.0001'.replaceFirst(new RegExp(r'0+'), ''); // '.0001'
</pre>

You can use `replaceAllMatched()` to register a function that modifies the
matches:

<pre class="programlisting">
var heart = '\u2661'; // '♡'
var string = 'I like Ike but I $heart Lucy';
var regExp = new RegExp(r'[A-Z]\w+');
var newString = string.replaceAllMapped(regExp, (match) {
  return match.group(0).toUpperCase()
}); 
print(newString); // 'I like IKE but I ♡ LUCY'
</pre>



