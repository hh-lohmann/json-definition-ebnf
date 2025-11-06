# JSON definition as EBNF with characters of ISO 8859-15

The original definition of JSON on [json.org](https://json.org) is given in [McKeeman form](https://www.crockford.com/mckeeman.html), the IETF RFC's (4627 / 7159 / [8259](https://datatracker.ietf.org/doc/html/rfc8259)) use [ABNF](https://datatracker.ietf.org/doc/html/rfc5234), and its specifications in [ECMA 2009](https://ecma-international.org/wp-content/uploads/ECMA-262_5th_edition_december_2009.pdf) and as [ECMA-404](https://ecma-international.org/publications-and-standards/standards/ecma-404/) sport a "human readable" formulation.

For formal interests like defining a subset of JSON, eg. with restricted object keys, something like an EBNF representation would be a nice base.

Many attempts to express "the" JSON definition in EBNF stall on defining "character": EBNF has no way for character classes like McKeeman's `'0020' . '10FFFF' - '"' - '\' for "Unicode points from U+0020 to U+10FFFF except double quote and backslash". In EBNF one would to have explicitly list all the Unicode characters that match the McKeeman class definition - although the current Unicode 17 spec only defines [159,801](https://www.unicode.org/versions/Unicode17.0.0/#Summary) of the possible over a million Unicode points (up to U+10FFFF) a hard task.

The EBNF here is pragmatically restricted to "Western European" explicit characters as defined by ISO 8859-15 except control codes from U+0000 to U+001F and U+007F to U+009F, double quote, and backslash - only 188 possible characters, but making visible that pure JSON would allow object keys like `}«¶>{:` - something you would not like to work with and giving reason to implement a restricted subset of JSON for your purposes. Additionally to the explicit characters JSON allows `escape` sequences, reintroducing some control codes and, by `'u' hex hex hex hex' (McKeeeman form), effectively allowing for Unicode points from U+0000 to U+FFFF which does not only reintroduce the formerly excluded control codes but also restricts Unicode to its [Basic Multilingual Plane](https://en.wikipedia.org/wiki/Plane_(Unicode)#Basic_Multilingual_Plane) - being obviously that what was known as [Unicode 3.0](https://www.unicode.org/Public/3.0-Update/Blocks-3.txt) in 2001 when "JSON was first presented to the world", to use the words of ECMA-404. In most parts of the world you would tend to use a subset of this for e.g. sensible object keys, but if speakers of [Tai Yo](https://en.wikipedia.org/wiki/Tai_Yo_language) would become a commercial aspect, building a superset of JSON may become an interesting task.

Below naming and order of rules follow json.org, a rule `hex_alpha` acts as providing a character class for hex numbers.

  * **Note that you may encounter even good JSON validators / parsers that will reject pure strings like `"xyz"` and / or pure numbers like `999`, despite being corrext JSON according to the offical definition (see above)** - for pure strings the JSON requirement of surrounding double quotes often is a problem at the input for validators / parsers, i.e. you may have to single quote them as e.g. `'"xyz"'`, but sometimes, as with pure numbers, implementations are just wrong, of course mitigated by the fact that the use cases of using JSON for pure strings or pure numbers are more than rare. **The EBNF below handles pure strings and numbers correctly**.

```ebnf
json = element ;
value = object | array | string | number | 'true' | 'false' | 'null' ;
object = '{', ( ws | element ), '}' ;
members = member, { members } ;
member = ws, string, ':', element ;
array = '[', ( ws | elements ), ']' ;
elements = element, { elements } ;
element = ws, value, ws ;
string = '"', characters, '"' ;
characters = '' | ( character, characters ) ;
character =  ( ' ' | '!' | '#' | '$' | '%' | '&' | '"' | '(' | ')' | '*' | '+' | ',' | '-' | '.' | '/' | '0' | '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' | ':' | ';' | '<' | '=' | '>' | '?' | '@' | 'A' | 'B' | 'C' | 'D' | 'E' | 'F' | 'G' | 'H' | 'I' | 'J' | 'K' | 'L' | 'M' | 'N' | 'O' | 'P' | 'Q' | 'R' | 'S' | 'T' | 'U' | 'V' | 'W' | 'X' | 'Y' | 'Z' | '[' | ']' | '^' | '_' | '`' | 'a' | 'b' | 'c' | 'd' | 'e' | 'f' | 'g' | 'h' | 'i' | 'j' | 'k' | 'l' | 'm' | 'n' | 'o' | 'p' | 'q' | 'r' | 's' | 't' | 'u' | 'v' | 'w' | 'x' | 'y' | 'z' | '{' | '|' | '}' | '~' | '¡' | '¢' | '£' | '€' | '¥' | 'Š' | '§' | 'š' | '©' | 'ª' | '«' | '¬' | '®' | '¯' | '°' | '±' | '²' | '³' | 'Ž' | 'µ' | '¶' | '·' | 'ž' | '¹' | 'º' | '»' | 'Œ' | 'œ' | 'Ÿ' | '¿' | 'À' | 'Á' | 'Â' | 'Ã' | 'Ä' | 'Å' | 'Æ' | 'Ç' | 'È' | 'É' | 'Ê' | 'Ë' | 'Ì' | 'Í' | 'Î' | 'Ï' | 'Ð' | 'Ñ' | 'Ò' | 'Ó' | 'Ô' | 'Õ' | 'Ö' | '×' | 'Ø' | 'Ù' | 'Ú' | 'Û' | 'Ü' | 'Ý' | 'Þ' | 'ß' | 'à' | 'á' | 'â' | 'ã' | 'ä' | 'å' | 'æ' | 'ç' | 'è' | 'é' | 'ê' | 'ë' | 'ì' | 'í' | 'î' | 'ï' | 'ð' | 'ñ' | 'ò' | 'ó' | 'ô' | 'õ' | 'ö' | '÷' | 'ø' | 'ù' | 'ú' | 'û' | 'ü' | 'ý' | 'þ' | 'ÿ' ) | ( '\\', escape ) ;
escape = '"', '\\', '/', 'b', 'f', 'n', 'r', 't', ( 'u', hex, hex, hex, hex ) ;
hex = digit | hex_alpha ;
hex_alpha = 'A' | 'B' | 'C' | 'D' | 'E' | 'F' | 'a' | 'b' | 'c' | 'd' | 'e' | 'f' ;
number = integer | fraction | exponent ;
integer = digit | ( onenine, digits ) | ( '-', ( digit | ( onenine, digits ) ) ) ;
digits = digit, { digits } ;
digit = '0' | onenine ;
onenine = '1' | '2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' ;
fraction = '' | ( '.', digits ) ;
exponent = '' | ( ( 'E' | 'e' ), sign, digits ) ;
sign = '' | '+' | '-' ;
ws = '' | ( ' ' | '\n' | '\r' | '\t' ), ws ;
```
