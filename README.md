


### Table of content

* [What is a regular expression ?](#what-is-a-regular-expression-)

* [How a regular expression works ?](#how-a-regular-expression-works-)
  * [Two metacharacters are enough](#two-metacharacters-are-enough)
  * [Negating and quandtifying](#negating-and-quantifying)
  * [Backslash limitation](#backslash-limitation)
  * [The concept of rules](#the-concept-of-rules)
  * [Variables](#variables)
  * [Capturing group and backreference](#capturing-groups-and-backreferencing)
  
* [`ft_regex`](#ft_regex-1)
  * [libft](#the-latest-version-of-the-ft_regex-function-is-available-in-the-libft-library)
  * [The global flag](#the-global-flag)
  * [Adding rules](#adding-rules)
    * [RGX_ADD a rule](#rgx_add-a-rule)
    * [RGX_LOAD rules from a file](#rgx_load-rules-from-a-file)
  * [Global and local rules](#global-and-local-rules)
    * [RGX_TO a local list](#rgx_to-a-local-list)
  * [Flags](#flags)


# What is a regular expression ?
A regular expression (or regex) is a way to define a search pattern, wich is usually used for 'find' operations on documents and any type of text. 
There are many regex engines out there in almost all programming languages, and they all follow the same syntax, there could be some differences but in general they do not vary that much.

### ft_regex
The regex engine i am going to present you works differently, and has a different syntax. But still follows the same 'idea', meaning that each character in a regular expression is still either a metacharacter, which are in number of two, or a literal character. This will be explained later, we will first talk about how it differs from most regex engines and why it could be powerfull.

First of all, it is important to know that it has started as a school project, so it may not be perfect, and has definetely a tons of flaws.

Secondly, this regex engine does not compile or precompile or whatever, it reads the regex expression character by character without using dynamic allocation. It uses the concept of 'rules' which is where the main difference and power stands, we will get to it later.

So, before diving into the code, and learn how to use the `ft_regex` function, let's first have a look at the syntax.

# How a regular expression works ?

As told earlier, a regular expression is just a bunch of charcaters that are either a metacharacter or a literal character.
A metacharacter do not define itself, and controls how the matching is done where a literal character define itself.
For example the regex `abc.` *abc* are literal characters, which matches just *abc*, while *.* is a metacharacter, which matches any single character. So this regex might match different strings that starts with *abc* and ends with any single character. This regex engine would rather define the regex as `abc?`.

Well i wont give a hole tutorial on how regular expression works as it is defenitely not the goal of this. But you could check [this website](https://www.regular-expressions.info/tutorial.html) to learn more.

## Two metacharacters are enough

I talked about this regex having only two metacharacter and it should be enough to match any possible string, this is the first reason of it being limited to two, the second one is to keep it simple and generic.

They are:
- `*`, the wildcard might match multiple times, and is kind of a 'while' loop.
- `?`, the question mark will match only once, and is more of an 'if'.

Something else to know that i forgot to mention, is that there is no such as backslashing character, so to match the *?*, `\?` will not work. The backslash character is a literal character. I refused to choose the backslash character because things could get ugly really quickly as the C language already use the backslash inside quotes, you would need to backslash the backslash etc.

But how would you match these metacharacters you ask me ? Well i will go straight to the answer : `?[?]`.

Here we used the square brackets `[]` to have more control on what should match, we specifed inside the only characters that shall match, in this case the question mark. If you remember the question mark means match only once, so the wildcard (`*[?]`) awaits any number of question marks and will eventually match strings that only contains question marks.

So as i said earlier this regex follow the same 'idea', and in this context the square brackets works the same, so to specify a range for example, `*[A-Z0-9?*]` will match any uppercase characters, digits, question marks or wildcards, and we used the wildcard metacharacter which means that it wont stop after the first match and will continue if possible.

## Negating and quantifying

You can negate a match by following a metacharacter with an exlamation mark `?![...]` or `*![...]`, so `?![A-Za-z]` will match any single character that is not a letter (neither an uppercase letter nor a lowercase letter).

We use quantifiers to control how many times a match shall occur, it is possible with the `@` sign inside of the square brackets and with the `*` metacharacter `*[A-Z@=4]` matches exactly four times characters between uppercase A and uppercase Z. And `*![A-Z@=4]` matches exactly four times characters that are not uppercase letters. 

Now what if you want an '@' sign inside of a character set as there is no such as backslash ? To do this, you would need to understand that the square brackets always takes the last occurence of the '@' sign to delimit the character set field and the quantifier field. 
`?[@@@]` the character set here is made of two '@' sign, the last one is the delimitation of the quantifier

So with quantifying we could mimic the question mark behaviour with the wildcard for example `*[...@=1]`.
Never forget that quantifying only works with the wildcard metacharacter and wont have any effect with the question mark metacharacter, it wont make any sense otherwise.

There are three other operators that are used for quantifying matches, they are:
- `[...@<n]`, the match shall occur less than n times.
- `[...@>n]`, the match shall occur more than n times.
- `[...@?]`, the match is optional, and will result in a zero-length match if the match failed. This quantifier works with the question mark too.

It is possible to give an empty character set to both of the metacharacters, for the wildcard it means match any single character between one and unlimited times, while the wildcard alone means match between zero and unlimited times. For the question mark it doesn't change it's behaviour and will always match any single character once and only once.

## Backslash limitation

There is a question that has not been raised yet which concerns how the square brackets works and how to have any possible character in a character set, because as we know we can't backslash a closing square bracket and with the concept of 'rules' things could have get really messy, so we had to counter the problem by letting the user control what starts a character set and what closes it.
> An opening square bracket followed by *n* opening curly bracket shall be closed by *n* closing curly bracket followed by a closing square bracket.

We would get something of this shape `?[{{...}}]`, and here we reached the limitation of not using backslashes, because with this solution how would you have a character set that starts with an opening curly bracket ? You may think that this question does not make any sense as the order of a character set shall not change it's behaviour, well it is true for this context but not for the context of rules, that are going to be seen in the next part (finally). To counter this other problem, backslashing is coming back but stays only for the very first character, and it just cancels the meaning of the backslash and the opening curly bracket characters. So the solution of the problem is `?[{\{...}]` or `?[{{{{\\....}}}}]` or `?[{...]...}]`. The '@' sign should always appair before whatever closing has been used: `?[{{...@=3}}]`.

At this point we solve the backslashing problem.

## The concept of rules

We finally get to the main power of the `ft_regex` engine. The rules.

A rule is no more than the equivalent of a function in programming languages. It takes an input (the string), and gives an output (does it match and how many times). A rule is just a way to group other regexes, for example we could imagine a rule that checks if the match is an alphabetic character: `alpha:?[A-Za-z]`. So when we call the `alpha` rule it will, in fact, check if the character is between 'A' and 'Z' or 'a' and 'z'. And this rule already exists, and there are many others that abstracts a bunch of things.

To call a rule you would need to put the rule name right after the '@' sign: `?[@alnum]` is the equivalent of `?[a-zA-Z0-9]`. And all of the negating and quantifying stuff still works: `*![@digit>3]` will awaits more than three characters that are not digits. And all the other features that are available in other regex engines are available in this regex engine through rules.

It is mainly the case of the `or` rule, that should simply match one of the alternatives: `*[ok?[@digit]|?[@alpha]@or]` shall match one or more times; the literal characters *ok* followed by a digit; or an alphabetic character. And this could be a rule: `strange_rule:*[ok?[@digit]|?[@alpha]@or]`, you should have seen that what separates the regexes is the `|` character so in this context it loses its literal meaning, and if we wanted to match this character we would do the same as for the metacharacters `?[|]`. And in this example we understand that rules receives in reality two input, the string to match and an optional argument which is specified before the '@' sign.

The truth is that everything is a rule, and even when you try to match a character set: `?[abc]` you are in fact calling a rule named 'DEFAULT' so it is the same as typing `?[abc@DEFAULT]`. So at this point the `DEFAULT` rule cannot call other rules, as it is the lowest rule. Which is good, because it means that a rule doesn't need to call other rules in order to work, they can also call C functions ! We can conclude that rules are not limited to the power of other rules but of the C programming language !!

It is commonly known that regexes are limited, and `ft_regex` is no exception, but with the concept of rules we break all limitations.

Here is a non-exhaustive list of available rules.

| Name | Description | Method | Example | Explanation |
| --- | --- | --- | --- | --- |
| `@or` | One of the alternatives given in argument shall match | C function | `reg?[ex\|ular expression@or]?[s@?]` | Might match 'regex' 'regular expression' 'regexs' or 'regular expressions' |
| `@case` | Compare the strings ignoring case | C function | `?[hello@case] world` | Will match any possible form of the 'hello' string followed by exactly ' world' |
| `@^` | Beginning of string | C function | `?[@^]hello` | The string '*hello*' matches only if it is at the beginning of a string |
| `@$` | End of string | C function | `hello?[@$]` | The string '*hello*' matches only if it is at the end of a string |
| `@^n` | Beginning of string or line | C function | `?[@^n]hello` | The string '*hello*' matches only if it is at the beginning of a string or a line |
| `@$n` | End of string or line | C function | `hello?[@$n]` | The string '*hello*' matches only if it is at the end of a string or a line |
| `@^w` | Beginning of string, line or word | C function | `?[@^w]hello` | The string '*hello*' matches only if it is at the beginning of a string, line or word |
| `@$w` | End of string, line or word | C function | `hello?[@$w]` | The string '*hello*' matches only if it is at the end of a string, line or word |
| `@upper` | Any uppercase letter | Inline regex | `*![@upper=3]` | Matches three characters that are not uppercase letters |
| `@lower` | Any lowercase letter | Inline regex | `?[@upper]*[@lower]` | An uppercase letter followed by any number of lowercase letters |
| `@and` | All of the alternatives given in argument shall match  | C function | `*[?![@upper]&?![@lower]@and]` | Any number of characters that are neither an uppercase letter nor a lowercase letter, equivalent of `*![@alpha]` |
| `@word` | Any word character that is either an alphabetic character, a digit or an underscore. | Inline regex | `?[@^w]*[@word=3]?[@$w]` | Three word characters between word boundaries |
| `@space` | Any space character | Inline regex | `?![@space]` | Match any character that is not a space. |
| `@int` | A valid integer | Inline regex | `?[@int]` | An integer might start with zero or more space followed by; a minus or a plus sign or nothing; followed by one or more digits. |
| `@X` | The regex given in argument shall match | C function | `*[?[@int]:*;@X=3]` | Awaits three times an integer followed by ':' then zero or more  of any single character that ends with ';' |
| `@R` | Recurses the entire pattern | C function | `(*[?![()]\|?[@R]@or])` | Match nested parenthesis, this can also be done with a rule calling itself |
| `@E` | Perform arithmetic expressions or assignements on variables | C function | `?[n=e+6@E]` | Add 6 to the variable *e* and puts the result in the variable *n* |
| `@G` | Memorizes the part of the string that matched the regular expression given in argument | C function | `?[*[@word]@G]` | Memorize an entire word into a capturing group |
| `@B` | Backreferences to a previously memorized substring specified by an index read from the argument | C function | `?[*[@word]@G]:?[1@B]` | The memorized substring generated by the previous call of `@G` which has been indexed on 1, is rematched by the `@B` rule.
| `@debug` | Print informations about the current state of the regex without matching any characters | C function | `?[@debug]` | Print informations |

## Variables

Another interestring feature available with this engine is the possibility of using variables throughout the regular expression. Thanks to the `@E` rule you have access to 52 variables (26 lowercase letter and 26 uppercase letter). They can be instanciated to a literal value `?[n=3@E]`, or another variable `?[x=C@E]`, the first example puts the value *3* to the variable *n* and the second one sets the value of *x* to the value of *C*. You also have the possibility of using arithmetic expressions with variables, `?[n=3+P@E]` will add *3* to the current value of *P* and will put the result to the variable *n*.

These are the available arithmetic operators

| Operator | Descrption |
| --- | --- |
| `+` | Returns the addition of the two variables |
| `-` | Returns the subtraction the two variables |
| `-` | Returns the multiplication the two variables |
| `/` | Returns the division of the two variables |
| `=` | Returns 1 if the two variables are equal or 0 otherwise |
| `!` | Returns 1 if the two variables are different or 0 otherwise |
| `>` | Returns 1 if the first variable is greater than the second one or 0 otherwise |
| `<` | Returns 1 if the first variable is less than the second one or 0 otherwise |

It is important to mention that this rule is a zero-length match meaning that it wont consume any characters, it will just change the current state of variables. But it is possible to use it as a condition checker, by placing a *0* instead of a variable name as a first character.
`?[0=n=3@E]` will check if *n* is equal to *3*, the `0=` means that we want to return true if the resut of the expression is different than zero or false if it is equal to zero.

You also have the possibility to store the return of a regular expression inside of a variable by using a colon (':') or a semicolon (';') instead of an equals sign ('=') as a second character. `?[n:*[@space]@E]` will store in the variable *n* the number of space characters that were met or -1 if the regular expression failed, here we used the colon operator so the space characters that were met wont be consumed, to consume them you will need to match them right after the call of the `@E` rule (`?[n:*[@space]@E]*[@space]`), this is the difference with the semicolon operator which will consume the matched characters at the same time, but will fail if the subexpression has failed.

The point of variables is to use them for quantifying instead of using literal values. `*[@digit=n]` awaits *n* digits.

## Capturing groups and backreferencing

Another important and powerfull concept about regular expressions that this engine implements is the possibility of memorizing chuncks of strings.

To capture a group you need to use the `@G` rule, this rule takes in argument a regular expression which will store the matched part of the string. The captured groups can then be reused via a backreference with the `@B` rule that takes the index of which group to reuse, in arguemnt.

The regex `?[*[@alpha]@G]:?[0@B]` will first store the alphabetical characters in a group, then will match a colon, then will backreference to the first captured group. For example the string `hello:hello` will match but `hello:no` wont.

A more usefull and explicit example would be to define a regular expression that matches opening tags and closing tags for the HTML syntax: `<?[*[@word]@G]*>*</?[0@B]>`.

# `ft_regex`

#### The latest version of the `ft_regex` function is available in the [libft](https://github.com/mmerabet42/libft) library.

Now that we have seen how this regex engine works (and why it might be powerfull ?) let's dive into the code and let's see how we can use it.

Everything is done through one and only one function:
```C
#include "libft/includes/ft_regex.h"

int  ft_regex(int flags, const char *regex, const char *string, ...)
```

The way it behaves depends on the `flags` it receives, the `regex` is the search pattern which will be tested on the `string` subject.

By default, with no flags, the `string` argument shall entirely match the `regex` string.
```C
ft_regex(0, "?[@upper]*[@lower]", "Hello")
```

In this example the match is correct, and the function returns the number of matches which is 5.

But
```C
ft_regex(0, "?[@upper]", "Hello")
```

Wont match because the search pattern only ask for an uppercase letter, so the function returns -1.

The `RGX_END` flag will allow us to match string that at least starts with the given search pattern.
```C
ft_regex(RGX_END, "?[@upper]", "Hello")
```

Matches only once, because it at least starts with an uppercase letter, this call returns 1.

The `RGX_POS` makes `ft_regex` to return an extra information which is the position of the matched string, so it doesn't need to start with the search pattern, the extra information is stored in an int whose address is sent in parameter to the function.
```C
int pos;
int len = ft_regex(RGX_POS | RGX_END, "*[@digit>2]", "The number is 12 or 569 ?", &pos);
```

Will match the first three or more following digits in the string which is *569*, `pos` is equal to 20 as it is the position of the matched pattern, and `len` is the number of matched characters which is 3.

## The global flag

There are other flags, but it is not really usefull to mention them as they are used by the most important one: `RGX_GLOBAL`. This flag will store all the matching pattern in a linked list of `t_regex_match` structures.
```C
t_list *matches;
int num_of_matches = ft_regex(RGX_GLOBAL, "*[@word]", "Regular expressions are amazing !", &matches);
```

You can use the `ft_print_matches` function to hightlight all the matches in a human readable way
```C
void ft_print_matches(const char *string, t_list *matches)
```

  * ![ft_print_matches output](/screenshots/vowels.png)

Here, we've asked to match only words, but also to capture the vowels of these words, this is the reason why they are colored differently. The ![0 at the end](/screenshots/end_match.png) at the end indicates the id of the last called rule of the matched pattern.

The  `t_regex_match` structure is defined as follow:
```C
struct t_regex_match
{
  const char  *str; // A pointer to the start of the match
  int         pos; // The position of the matched pattern
  int         len; // The length of the matched pattern
  int         id; // An identifier to precise what matched
  t_list      *groups; // The captured groups
};
```

The fourth attribute is a way to distinguish what matched, and it is related to rules, we will see it in the next part.

After using the returned list you can free it properly with the following call
```C
ft_regex(RGX_FREE, NULL, NULL, &matches)
```

## Adding rules

As told earlier with the concept of rules, we can add rules by ourselves. This can be done with by various methods.

#### RGX_ADD a rule

The RGX_ADD flag adds one rule, by taking the `regex` parameter as the rule name and the `string` parameter as the regex definition.

```C
ft_regex(RGX_ADD, "three_char_word", "?[@^w]*[@word=3]?[@$w]", NULL)
```
This call will add a rule named '*three_char_word*' that defines words of excactly three characters long. The function returns the id that has been assigned to the rule which can be used to differenciate it from other rules.
The '*three_char_word*' rule can now be called in any regular expression:
```C
ft_regex(RGX_GLOBAL, "?[@three_char_word]", "Hello budy, how are you ?", &matches)
```
This call will match three times ('how' 'are' and 'you'), without the word boundaries (`?[@^w]` and `?[@$w]`) we would have also matched the first three characters of 'Hello' and 'budy'.

Now lets imagine a scenario were we would need to match three character words and five character words too, but still be able to distinguish these two possible match. For this case we would need to keep the '*three_char_word*' rule and add another one that matches five character words:
```C
ft_regex(RGX_ADD, "five_char_word", "?[@^w]*[@word=5]?[@$w]", NULL)
```
Then we will ask the engine to match '*three_char_word*' or '*five_char_word*'.
```C
ft_regex(RGX_GLOBAL, "?[?[@three_char_word]|?[@five_char_word]@or]", "Hello budy, how are you ?", &matches)
```
The linked list returned will, now, have a fourth match from the initial ones which is 'Hello', but this time the `t_regex_match.id` attribute will point to different rules depending on what matched.

You might have noticed the 'NULL' parameter at the end, well you have to know that a rule is either an inline regex, meaning that it is defined by another regex, or a function callback which means that if the rule is called it will in reality call a C function specified by the last parameter. But it cant be both. The callback function must follow this prototype:
```C
int (*callback)(t_regex_info *, t_regex_rule *)
```

The function receives two structures, `t_regex_info` gives informations about the regex in general, and `t_regex_rule` gives information about the rule:
```C
struct t_regex_info
{
  const char  *str_begin; // Pointer to the beginning of the string subject
  const char  *rgx_begin; // Pointer to the beginning of the regular expression
  const char  *str; // Current position in the string
  const char  *regex; // Current position in the regular expression
  int         len; // The number of matched characters
  int         *vars; // All the 52 variables
  int         flags; // Currently enabled flags
  void        *data; // The data received in parameter of ft_regex if the RGX_DATA flag is enabled
};
```

```C
struct t_regex_rule
{
  const char  *rule; // The rule name
  const char  *arg; // The argument received before the '@' sign. Keep in mind that it is not a duplicate of the argument
                    // but just a pointer to the beginning of the argument in the 'rgx_begin' string
  int         len_rule; // The length of the rule name
  int         len_arg; // The length of the received argument
};
```

There are two ways of consuming characters with this method, the first one is to simply return the number of matched characters and the second one is to increment directly the `t_regex_info.str` pointer, the difference is that with the last method the matched characters wont be added to the final return of the `ft_regex` function, (the boundary rules uses this last method `@$n` and `@$w`). If you want the match to fail the function must return -1.

#### RGX_LOAD rules from a file

You can also load rules from a file. For example the 'rules.rgx' file is formatted as
```
rule0  "regular expression ..."
rule1 "regular expression ..."
```

And can be loaded by calling `ft_regex` with the RGX_LOAD flag.
```C
ft_regex(RGX_LOAD, "rules.rgx", NULL)
```

The function returns the number of added rule. When adding rules with the RGX_LOAD flag, the RGX_READABLE flag is also enabled which means that it will ignore any space character in the regex definition.

## Global and local rules

It is also possible to add rules to a specific 'channel', by default the added rules are pushed into a global list which can be requested with the RGX_GET flag.
```C
t_list *list;
ft_regex(RGX_GET, NULL, NULL, &list)
```

The `list` variable now points to the global rules. The reason of that is that you can also set the global rules list with the RGX_SET flag.
```C
ft_regex(RGX_SET, NULL, NULL, &list)
```

After this call, all the added rules are pushed into the local list `list`. As the RGX_SET flag overwrite the current global list, it should be stored somewhere with the RGX_GET flag before overwriting it, so it can be set back.

#### RGX_TO a local list

Instead of playing with the RGX_GET and RGX_SET flag, for adding rules to a local list, you can use the RGX_TO flag, combined with the RGX_ADD or RGX_LOAD flag, to specify in which list to add the rules.
```C
t_list *list = NULL;
ft_regex(RGX_LOAD | RGX_TO, "rules.rgx", NULL, &list);
```

The 'rules.rgx' file will be loaded to the local list `list`. This is usefull to avoid conflicts and duplications.

## Flags

| Flag | Description | Prototype |
| --- | --- | --- |
| RGX_RGXN | The first n characters of the search pattern shall match (not yet implemented). | `int rgxn` |
| RGX_STRN | The first n characters of the string shall match (not yet implemented). | `int strn` |
| RGX_POS | Returns the starting position of the matched pattern. | `int *pos` |
| RGX_END | The string subject must at least start with the given search pattern, wiith the RGX_POS flag it must contain at least once the search pattern somewhere in the string. | |
| RGX_VAR | By default the `ft_regex` function creates and uses its own 52 variables, but it is possible to send them in paremeter so the `ft_regex` function will use them, so all modifications made are left unchanged. | `int vars[52]` |
| RGX_ID | Returns the id of the last called rule in an int pointer. With the RGX_ADD flag it lets you specify the id of the rule | `int *id` or `int id` |
| RGX_GLOBAL | Stores in a linked list the matching part of the string | `t_list **matches` |
| RGX_UGLOBAL | Stores in a linked list the non-matching part of the string, it 'splits' the string, to be distinguished from other matches, their id is equal to -1 | `t_list **matches` |
| RGX_GROUP | At the end of an `ft_regex` call, the capturing groups, if any, are destroyed by default. This flag will instead return the captured group in a linked list of `t_regex_group` structure. | `t_list **groups` |
| RGX_DATA | Sends an extra data to the regex functions | `void *data` |
| RGX_READABLE | Ignores all space characters in the regular expression and all other rules that allows regular expressions in argument. this flag is disabled when calling a rule that is an inline regex. For an inline regex rule to be readable, it needs to have the RGX_READABLE flag enabled with the RGX_ADD flag. | |
| RGX_ADD | Add a rule to the regex engine | `t_regex_funcptr *func` |
| RGX_ADD_MULTI | Add rules from an array of `t_regex_func` structures | `t_regex_func *funcs, size_t len` |
| RGX_LOAD | The engine will load the rules from a file formatted as `rule_name "regular expression"`, each rules are added with the RGX_READABLE flag automatically | |
| RGX_GET | Returns a linked list of all the added rules | `t_list **rules` |
| RGX_SET | Set the default global list | `t_list **rules` |
| RGX_TO | Must go with RGX_ADD or RGX_LOAD. It pushes the rule(s) into a specified local list. With RGX_CLEAN it free it properly. | `t_list **rules` |
| RGX_FREE | Free the linked list previously returned by a call of `ft_regex` with the `RGX_GLOBAL` or `RGX_UGLOBAL` flag | `t_list **matches` |
| RGX_FREEGRP | Free the linked list previously returned by a call of `ft_regex` with the `RGX_GROUP` flag | |
| RGX_CLEAN | Clean and free all the rules that were added manually by the `ft_regex` function with the `RGX_ADD` flag | |

And here is a table of all the possible combination with their order:

| Flags | Prototyping order |
| --- | --- |
| RGX_RGXN \| RGX_STRN \| RGX_POS \| RGX_END \| RGX_VAR \| RGX_ID \| RGX_DATA \| RGX_READABLE \| RGX_GROUP | `int rgxn, int strn, int *pos, int *id, t_list **groups, void *data, int vars[52]` |
| RGX_RGXN \| RGX_STRN \| RGX_GLOBAL \| RGX_UGLOBAL \| RGX_VAR \| RGX_DATA \| RGX_READABLE | `int rgxn, int strn, t_list **matches, void *data, int vars[52]` |
| RGX_ADD \| RGX_ID \| RGX_READABLE \| RGX_TO | `t_list **rules, t_regex_funcptr *func, int id` |
| RGX_ADD_MULTI | `t_list **rules, t_regex_funcs *funcs, size_t len` |
| RGX_LOAD \| RGX_TO | `const char *regex <, t_list **rules` |
| RGX_GET | `t_list **rules` |
| RGX_FREE | `t_list **matches` |
| RGX_FREEGRP | `t_list **groups` |
| RGX_CLEAN \| RGX_TO | `t_list **rules` |

![lol](/screenshots/picture.jpg)
