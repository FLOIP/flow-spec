# Expressions

## Expressions

While a flow paradigm is useful in representing control logic in an easy to understand manner for non-technical users, there is a need to both refer to variables collected within other parts of a flow and transform them in basic ways.

Requirements:

* Given a dictionary \(the context\) and a string expression, evaluates to a dictionary value and an error which can be null
* Must allow easy variable substitutions within strings as this is the most common activity. ex: `Hi @contact.name`
* Must solve halting problem, we cannot let users build expressions which do not exit
* Must not allow looping, these should be represented in the flows themselves, we don't want programs

  in expressions, the logic should be in the flow. Same goes for declaration of variables etc

* Must allow for the vast majority of transformations users require, though we realize not all will be possible within

  an expression alone

### Excellent - Excel Inspired Expressions

Excellent is an expression language which consists of the functions provided by Excel with a few additions. This provides a number of advantages:

* No halting problem, a single function call \(possibly nested\)
* Many users are already familiar with the provided functions, as they have used them in Excel
* More complex expressions can be 'tried out' within Excel
* Excel's set of functions has evolved over time to capture most use cases while remaining lightweight
* Variables from the context are referenced using standard dot-notation

Function and variable names are not case-sensitive so UPPER is equivalent to upper:

* `contact.name` -&gt; `Marshawn Lynch`
* `FIRST_WORD(contact.name)` -&gt; `Marshawn`
* `first_word(CONTACT.NAME)` -&gt; `Marshawn`

Expressions can also include arithmetic with the add \(+\), subtract \(-\), multiply \(\*\), divide \(/\) and exponent \(^\) operators:

* `1 + (2 - 3) * 4 / 5 ^ 6` -&gt; `0.999744`

You can also join strings with the concatenate operator \(&\):

* `contact.first_name & " " & contact.last_name` -&gt; `Marshawn Lynch`

#### Types

Excellent will attempt to cast strings to the following types when used in functions:

* Strings
* Decimal values
* Datetimes: \(ISO 8601\) or `dd-mm-yyyy HH:MM(:SS)` or `mm-dd-yyyy HH:MM(:SS)` or `mm-dd` or `dd-mm` or `mm-dd-yy` or `dd-mm-yy` \(environment will help with ambiguous cases\)
* Boolean: True or False \(case insensitive\)

#### Logical comparisons

A logical comparison is an expression which evaluates to TRUE or FALSE. These may use the equals \(=\), not-equals \(&lt;&gt;\), greater-than \(&gt;\), greater-than-or-equal \(&gt;=\), less-then \(&lt;\) and less-than-or-equal \(&lt;=\) operators

* `contact.age > 18` -&gt; `TRUE`

Note that when comparing text values, the equals \(=\) and not-equals \(&lt;&gt;\) operators are case-insensitive.

### Templating

For templating, RapidPro uses the `@` character to denote either a single variable substitution or the beginning of an Excellent block. `@` was chosen as it is known how to type by a broad number of users regardless of keyboard. It does have the disadvantage of being used in email addresses and Twitter handles, but these are rarely ambiguous and escaping can be done easily via doubling of the character \(`@@`\).

Functions are called by using the block syntax: `10 plus 4 is @(SUM(10, 4))` Within a block, `@` is not required to refer to variable in the context: `Hello @(contact.name)` A template can contain more than one substitution or block: `Hello @contact.name, you were born in @(YEAR(contact.birthday))`

Examples below use the following context:

```javascript
{ 
   "contact": {
     "name": "Marshawn Lynch",
     "jersey": 24,
     "age": 30,
     "tel": "+12065551212",
     "birthday": "22-04-1986",
     "__value__": "Marshawn Lynch"
   },
   "channel": {
     "name": "Twilio 1423",
     "address": "1423"
   }
}
```

| Template | Evaluated | Notes |
| :--- | :--- | :--- |
| Hi @contact.name | Hi Marshawn Lynch |  |
| Hi @contact | Hi Marshawn Lynch | if a dictionary is referred to which has a `__value__` element, that is used in substitution |
| Hi @channel | Hi { "name": "Twilio 1423", "address": "1423" } | without a `__value__` element, JSON representation is substituted |
| You can contact us at foo@bar.com | You can contact us at foo@bar.com | bar is not in context, so passed through |
| You can contact us at foo@contact.com | You can contact us at foo@contact.com | contact.com not in context, so passed through |
| You can contact us at foo@@contact.tel | You can contact us at foo@contact.tel | `@@` escapes to `@` |
| Next year you will be @\(contact.age+1\) | Next year you will be 31 | simple math possible within expression |
| You first name is @\(WORD\(contact.name, 1\)\) | Your first name is Marshawn | Specific to Excellent, 1 based indexing |
| You are now @\(YEAR\(NOW\(\)\) - YEAR\(contact.birthday\)\) | You are now 30 | Conversion to date, then subtraction |

## Function Reference

Function arguments in square brackets \(\[ ... \]\) are optional.

#### Date and time functions

**DATE\(year, month, day\)**

Defines a new date value

`This is a date @DATE(2012, 12, 25)`

**DATEVALUE\(text\)**

Converts date stored in text to an actual date, using your organization's date format setting

`You joined on @DATEVALUE(contact.joined_date)`

**DAY\(date\)**

Returns only the day of the month of a date \(1 to 31\)

`The current day is @DAY(contact.joined_date)`

**EDATE\(date, months\)**

Moves a date by the given number of months

`Next month's meeting will be on @EDATE(date.today, 1)`

**HOUR\(datetime\)**

Returns only the hour of a datetime \(0 to 23\)

`The current hour is @HOUR(NOW())`

**MINUTE\(datetime\)**

Returns only the minute of a datetime \(0 to 59\)

`The current minute is @MINUTE(NOW())`

**MONTH\(date\)**

Returns only the month of a date \(1 to 12\)

`The current month is @MONTH(NOW())`

**NOW\(\)**

Returns the current date and time

`It is currently @NOW()`

**SECOND\(datetime\)**

Returns only the second of a datetime \(0 to 59\)

`The current second is @SECOND(NOW())`

**TIME\(hours, minutes, seconds\)**

Defines a time value which can be used for time arithmetic

`2 hours and 30 minutes from now is @(date.now + TIME(2, 30, 0))`

**TIMEVALUE\(text\)**

Converts time stored in text to an actual time

`Your appointment is at @(date.today + TIME("2:30"))`

**TODAY\(\)**

Returns the current date

`Today's date is @TODAY()`

**WEEKDAY\(date\)**

Returns the day of the week of a date \(1 for Sunday to 7 for Saturday\)

`Today is day no. @WEEKDAY(TODAY()) in the week`

**YEAR\(date\)**

Returns only the year of a date

`The current year is =YEAR(NOW())`

#### Logical functions

**AND\(arg1, arg2, ...\)**

Returns TRUE if and only if all its arguments evaluate to TRUE

`@AND(contact.gender = "F", contact.age >= 18)`

**IF\(arg1, arg2, ...\)**

Returns one value if the condition evaluates to TRUE, and another value if it evaluates to FALSE

`Dear @IF(contact.gender = "M", "Sir", "Madam")`

**OR\(arg1, arg2, ...\)**

Returns TRUE if any argument is TRUE

`@OR(contact.state = "GA", contact.state = "WA", contact.state = "IN")`

#### Math functions

**ABS\(number\)**

Returns the absolute value of a number

`The absolute value of -1 is @ABS(-1)`

**MAX\(arg1, arg2, ...\)**

Returns the maximum value of all arguments

`Please complete at most @MAX(flow.questions, 10) questions`

**MIN\(arg1, arg2, ...﻿\)**

Returns the minimum value of all arguments

`Please complete at least @MIN(flow.questions, 10) questions`

**POWER\(number, power\)**

Returns the result of a number raised to a power - equivalent to the ^ operator

`2 to the power of 3 is @POWER(2, 3)`

**SUM\(arg1, arg2, ...\)**

Returns the sum of all arguments, equivalent to the + operator

`You have =SUM(contact.reports, contact.forms) reports and forms`

#### Text functions

**CHAR\(number\)**

Returns the character specified by a number

`As easy as @CHAR(65), @CHAR(66) , @CHAR(67)`

**CLEAN\(text\)**

Removes all non-printable characters from a text string

`You entered @CLEAN(step.value)`

**CODE\(text\)**

Returns a numeric code for the first character in a text string

`The numeric code of A is @CODE("A")`

**CONCATENATE\(args\)**

Joins text strings into one text string

`Your name is @CONCATENATE(contact.first_name, " ", contact.last_name)`

**FIXED\(number, \[decimals\], \[no\_commas\]\)**

Formats the given number in decimal format using a period and commas

`You have @FIXED(contact.balance, 2) in your account`

**LEFT\(text, num\_chars\)**

Returns the first characters in a text string

`You entered PIN @LEFT(step.value, 4)`

**LEN\(text\)**

Returns the number of characters in a text string

`You entered @LEN(step.value) characters`

**LOWER\(text\)**

Converts a text string to lowercase

`Welcome @LOWER(contact)`

**PROPER\(text\)**

Capitalizes the first letter of every word in a text string

`Your name is @PROPER(contact)`

**REPT\(text, number\_times\)**

Repeats text a given number of times

`Stars! @REPT("*", 10)`

**RIGHT\(text, num\_chars\)**

Returns the last characters in a text string

`Your input ended with ...=RIGHT(step.value, 3)`

**SUBSTITUTE\(text, old\_text, new\_text, \[instance\_num\]\)**

Substitutes new\_text for old\_text in a text string. If instance\_num is given, then only that instance will be substituted

`@SUBSTITUTE(step.value, "can't", "can")`

**UNICHAR\(number\)**

Returns the unicode character specified by a number

`As easy as =UNICHAR(65), =UNICHAR(66) , =UNICHAR(67)`

**UNICODE\(text\)**

Returns a numeric code for the first character in a text string

`The numeric code of A is @UNICODE("A")`

**UPPER\(text\)**

Converts a text string to uppercase

`WELCOME =UPPER(contact)!!`

#### Excellent Specific Functions

These functions are not found in Excel but have been provided for the sake of convenience.

**FIRST\_WORD\(text\)**

Returns the first word in the given text - equivalent to WORD\(text, 1\)

`The first word you entered was @FIRST_WORD(step.value)`

**PERCENT\(number\)**

Formats a number as a percentage

`You've completed @PERCENT(contact.reports_done / 10) reports`

**READ\_DIGITS\(text\)**

Formats digits in text for reading in TTS

`Your number is @READ_DIGITS(contact.tel_e164)`

**REMOVE\_FIRST\_WORD\(text\)**

Removes the first word from the given text. The remaining text will be unchanged `You entered @REMOVE_FIRST_WORD(step.value)`

**WORD\(text, number, \[by\_spaces\]\)**

Extracts the nth word from the given text string. If stop is a negative number, then it is treated as count backwards from the end of the text. If by\_spaces is specified and is TRUE then the function splits the text into words only by spaces. Otherwise the text is split by punctuation characters as well

`@WORD("hello cow-boy", 2)` will return "cow"

`@WORD("hello cow-boy", 2, TRUE)` will return "cow-boy"

`@WORD("hello cow-boy", -1)` will return "boy"

**WORD\_COUNT\(text, \[by\_spaces\]﻿\)**

Returns the number of words in the given text string. If by\_spaces is specified and is TRUE then the function splits the text into words only by spaces. Otherwise the text is split by punctuation characters as well

`You entered @WORD_COUNT(step.value) words`

**WORD\_SLICE\(text, start, \[stop\], \[by\_spaces\]﻿\)**

Extracts a substring of the words beginning at start, and up to but not-including stop. If stop is omitted then the substring will be all words from start until the end of the text. If stop is a negative number, then it is treated as count backwards from the end of the text. If by\_spaces is specified and is TRUE then the function splits the text into words only by spaces. Otherwise the text is split by punctuation characters as well

`@WORD_SLICE("RapidPro expressions are fun", 2, 4)` will return 2nd and 3rd words "expressions are"

`@WORD_SLICE("RapidPro expressions are fun", 2)` will return "expressions are fun"

`@WORD_SLICE("RapidPro expressions are fun", 1, -2)` will return "RapidPro expressions"

`@WORD_SLICE("RapidPro expressions are fun", -1)` will return "fun"

