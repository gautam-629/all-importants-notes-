
## Flags

|Flag|Description|Example|
|---|---|---|
|`g`|Global (find all matches)|`/cat/g`|
|`i`|Case-insensitive|`/cat/i`|
|`m`|Multiline (^ and $ match line breaks)|`/^cat/m`|
|`s`|Dot matches newlines|`/cat.dog/s`|
|`u`|Unicode|`/\u{1F600}/u`|
|`y`|Sticky (match from lastIndex)|`/cat/y`|

## Character Classes

|Pattern|Matches|Example|
|---|---|---|
|`.`|Any character except newline|`/c.t/` → "cat", "c9t"|
|`\d`|Any digit [0-9]|`/\d/` → "5"|
|`\D`|Any non-digit|`/\D/` → "a"|
|`\w`|Word character [a-zA-Z0-9_]|`/\w/` → "a", "5", "_"|
|`\W`|Non-word character|`/\W/` → "@", " "|
|`\s`|Whitespace (space, tab, newline)|`/\s/` → " "|
|`\S`|Non-whitespace|`/\S/` → "a"|
|`\t`|Tab|`/\t/`|
|`\n`|Newline|`/\n/`|
|`\r`|Carriage return|`/\r/`|
|`\0`|Null character|`/\0/`|
|`[abc]`|Any of a, b, or c|`/[aeiou]/` → "a", "e"|
|`[^abc]`|NOT a, b, or c|`/[^0-9]/` → "a"|
|`[a-z]`|Range from a to z|`/[a-z]/` → "m"|
|`[A-Z]`|Range from A to Z|`/[A-Z]/` → "M"|
|`[0-9]`|Range from 0 to 9|`/[0-9]/` → "5"|
|`[a-zA-Z]`|Any letter|`/[a-zA-Z]/` → "A", "z"|

## Quantifiers

|Pattern|Meaning|Example|
|---|---|---|
|`*`|0 or more|`/bo*/` → "b", "bo", "booo"|
|`+`|1 or more|`/bo+/` → "bo", "booo"|
|`?`|0 or 1 (optional)|`/colou?r/` → "color", "colour"|
|`{n}`|Exactly n times|`/\d{3}/` → "123"|
|`{n,}`|n or more times|`/\d{3,}/` → "123", "12345"|
|`{n,m}`|Between n and m times|`/\d{2,4}/` → "12", "123", "1234"|
|`*?`|Lazy 0 or more|`/<.+?>/` → "<div>" (not "<div>...</div>")|
|`+?`|Lazy 1 or more|`/\d+?/` → "1" (not "123")|
|`??`|Lazy 0 or 1|`/\d??/`|
|`{n,}?`|Lazy n or more|`/\d{2,}?/`|
|`{n,m}?`|Lazy n to m|`/\d{2,4}?/`|

## Anchors

|Pattern|Matches|Example|
|---|---|---|
|`^`|Start of string/line|`/^Hello/` → "Hello world"|
|`$`|End of string/line|`/world$/` → "Hello world"|
|`\b`|Word boundary|`/\bcat\b/` → "cat" (not "category")|
|`\B`|Non-word boundary|`/\Bcat\B/` → "concatenate"|

## Groups and Capturing

|Pattern|Description|Example|
|---|---|---|
|`(abc)`|Capturing group|`/(cat) and (dog)/`|
|`(?:abc)`|Non-capturing group|`/(?:cat) and (dog)/`|
|`\1`|Backreference to group 1|`/(\w+) \1/` → "hello hello"|
|`\2`|Backreference to group 2|`/(\w+) (\w+) \2/`|
|`(?<name>abc)`|Named capturing group|`/(?<year>\d{4})/`|
|`\k<name>`|Named backreference|`/(?<word>\w+) \k<word>/`|

## Alternation

|Pattern|Matches|Example|
|---|---|---|
|`a\|b`|a OR b|`/cat\|dog/` → "cat" or "dog"|
|`(a\|b)c`|ac OR bc|`/(cat\|dog)s/` → "cats" or "dogs"|

## Lookahead and Lookbehind

|Pattern|Description|Example|
|---|---|---|
|`(?=abc)`|Positive lookahead|`/\d(?= dollars)/` → "5" in "5 dollars"|
|`(?!abc)`|Negative lookahead|`/\d(?! dollars)/` → "5" in "5 euros"|
|`(?<=abc)`|Positive lookbehind|`/(?<=\$)\d+/` → "5" in "$5"|
|`(?<!abc)`|Negative lookbehind|`/(?<!\$)\d+/` → "5" in "5" (not "$5")|

## Special Characters (Must Escape with )

|Character|Description|
|---|---|
|`\.`|Literal period|
|`\^`|Literal caret|
|`\$`|Literal dollar sign|
|`\*`|Literal asterisk|
|`\+`|Literal plus|
|`\?`|Literal question mark|
|`\{` `\}`|Literal braces|
|`\[` `\]`|Literal brackets|
|`\\`|Literal backslash|
|`\|`|Literal pipe|
|`\(` `\)`|Literal parentheses|

## Assertions

|Pattern|Description|
|---|---|
|`^`|Start of string|
|`$`|End of string|
|`\b`|Word boundary|
|`\B`|Not word boundary|

## Unicode

|Pattern|Description|Example|
|---|---|---|
|`\u{XXXX}`|Unicode character (u flag)|`/\u{1F600}/u` → 😀|
|`\uXXXX`|Unicode character|`/\u00A9/` → ©|
|`\xXX`|Hex character|`/\x41/` → "A"|

## JavaScript Methods

|Method|Description|Example|
|---|---|---|
|`.test()`|Returns true/false|`/cat/.test("I have a cat")` → true|
|`.exec()`|Returns match array or null|`/cat/.exec("I have a cat")` → ["cat"]|
|`.match()`|Returns matches array or null|`"cat dog".match(/\w+/g)` → ["cat", "dog"]|
|`.matchAll()`|Returns iterator of all matches|`"cat dog".matchAll(/\w+/g)`|
|`.search()`|Returns index of first match|`"cat".search(/at/)` → 1|
|`.replace()`|Replace matches|`"cat".replace(/cat/, "dog")` → "dog"|
|`.replaceAll()`|Replace all matches|`"cat cat".replaceAll(/cat/g, "dog")`|
|`.split()`|Split by pattern|`"a,b,c".split(/,/)` → ["a", "b", "c"]|

## Common Patterns

|Pattern|Description|
|---|---|
|`/^\d+$/`|Only digits|
|`/^[a-zA-Z]+$/`|Only letters|
|`/^\w+$/`|Only word characters|
|`/^[\w\s]+$/`|Letters, digits, spaces|
|`/^[\w.-]+@[\w.-]+\.\w+$/`|Email (basic)|
|`/^\d{3}-\d{3}-\d{4}$/`|Phone (US format)|
|`/^https?:\/\//`|URL starting|
|`/^#[A-Fa-f0-9]{6}$/`|Hex color|
|`/^\d{4}-\d{2}-\d{2}$/`|Date (YYYY-MM-DD)|
|`/\s+/g`|Multiple spaces|
|`/<[^>]+>/g`|HTML tags|
|`/\/\*[\s\S]*?\*\//g`|Multi-line comments|

## Quick Examples

```javascript
// Match email
/^[\w.-]+@[\w.-]+\.\w{2,}$/.test("user@example.com")

// Extract numbers
"Price: $25.99".match(/\d+\.?\d*/g)  // ["25.99"]

// Replace spaces
"hello  world".replace(/\s+/g, " ")  // "hello world"

// Validate password (8+ chars, letter + number)
/^(?=.*[A-Za-z])(?=.*\d).{8,}$/.test("pass1234")

// Split by multiple delimiters
"a,b;c:d".split(/[,;:]/)  // ["a", "b", "c", "d"]

// Find all hashtags
"#hello #world".match(/#\w+/g)  // ["#hello", "#world"]

// Remove HTML tags
"<p>Text</p>".replace(/<[^>]+>/g, "")  // "Text"

// Check duplicate words
/\b(\w+)\s+\1\b/.test("hello hello")  // true
```