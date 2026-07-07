# Formatting Fun
As with all programming languages, it is useful to stick to a strict formatting style. This will benefit you and everyone who tries to quickly understand your complex script task.

In this part, we will take a look at a few common FEEL use cases:
* `if then else` (also nested) + `for in return` + `some in satisfies` + `every in satisfies`
* concatenating strings
* dealing with logic operations `and` and `or`

## Three-Keyword-Syntaxes
In FEEL, there are various three-keyword syntaxes, like `if then else`, `for in return`, `some in satisfies`, and `every in satisfies`.

I suggest to use a new line for each keyword, regardless of circumstances. In most cases, you should use a new line for the first keyword as well, so all three are properly aligned.

Even if the expression just takes a single line, the reader cannot easily grasp it in it entirety. They never know how long the expressions between the keywords will be. Maybe there are line breaks, or there will be after you change something. You can make it easier for you and the reader by formatting the syntax consistently from the start.

Let's look at some examples:

```
{
  "key": if isNew then new.key else distinct values(old.key)[1]
}
```

Here, `new` is a `context object` containing a field `key`. `old` is a list of such `context objects`. The last expression is quite complex, so why not break it down into a `multi step context object`?

```
// just the formatting
{
  "key": 
    if isNew
    then new.key
    else distinct values(old.key)[1]
}
// break down of last expression
{
  "key": 
    if isNew
    then new.key
    else 
      {
        "projectedKeys": old.key,
        "distinctKeys": distinct values(projectedKeys),
        "returnFirst": distinctKeys[1]
      }.returnFirst
}
```

Setting the braces `{` at the end of the line or the next does not matter that much! Just be consistent!

We will cover `multi line context objects` in a future post!

The same can be done for the other syntaxes. Here, we also use a new line after `return` to align the string concatenation.

```
for animal
in jungle
return 
  "type: " + animal.type + ",\n" + 
  "mood: " + animal.mood
```

You already know some examples for `some in satisfies` from our previous posts!

## Concatenating Strings
You just saw an easy example of formatting when concatenating strings. Here are some general tips:
* add line breaks (`\n`) as a separate summand
* use a line break after concatenating a line break
* use a new code line before a multi-argument function
* properly format multi-arguments functions in multiple lines
* keep the + at the end of the line
* when using `if then else` in combination with string concatenation, always use parantheses around the entire syntax
* don't follow `DRY` (don't repeat yourself), just duplicate some text instead of building a complex expression

Here is an example:

```
"Requests processed:" + "\n" +
(
  if count(requests[failed in (null, false)]) in (null, 0)
  then ""
  else 
    "Successful: " + 
    string join(
      requests[failed in (null, false)].id,
      ", "
    ) + "\n"
) +
(
  if count(requests[not(failed in (null, false))]) in (null, 0)
  then ""
  else 
    "Failed: " + 
    string join(
      requests[not(failed in (null, false))].id,
      ", "
    )
)
```

## Logical Operations

My first suggestion is: use a `multi-line context` and break up a complex logical operation at its `or` disjunctions.

```
not(performed in (null, false)) and deactivated in (null, false) or count(selectedList) > 0 and selectedList[1].action in ("actionA", "actionC")
```

This becomes:

```
{
  "actionActive": not(performed in (null, false)) and deactivated in (null, false),
  "specialAction": count(selectedList) > 0 and selectedList[1].action in ("actionA", "actionC"),
  "result": actionActive or specialAction
}.result
```

Now, both steps can be named which greatly helps with the understanding and maintenance. Still, we can now improve the formatting on the `and` conjunctions.

```
{
  "actionActive": 
    not(performed in (null, false)) 
    and 
    deactivated in (null, false),
  "specialAction": 
    count(selectedList) > 0 
    and 
    selectedList[1].action in ("actionA", "actionC"),
  "result": actionActive or specialAction
}.result
```

I suggest putting the logical operator on its own new line. While breaking apart the entire operations at the disjunctions, resulting in mostly `and`s in the logical steps, you might still have `or`s as part of smaller steps you don't want to break up, or as part of a larger negation.

While larger negations could be rewritten with de Morgan's law, doing so might cost you. You might have a similar logical operations, one negated (hide condition in a form) and one not negated (FEEL component in a form). Keeping the larger negation shows the similarities between the operations. Rewriting it will make the similarities vanish.

Let's start with the same operation, but negate it:

```
not(not(performed in (null, false)) and deactivated in (null, false) or count(selectedList) > 0 and selectedList[1].action in ("actionA", "actionC"))
```

Interestingly, we can still use the same approach of breaking a logical operation at the disjunctions, but it's not easy to see this directly. First, let's introduce some useful formatting:
* treat `not()` as a multi-argument function
* set the logical operators on their own new lines
* add new lines above and below `or`s to highlight the logical conjunctions (optional)

```
not(
  not(performed in (null, false)) 
  and 
  deactivated in (null, false) 

  or 

  count(selectedList) > 0 
  and 
  selectedList[1].action in ("actionA", "actionC")
)
```

Now, we can introduce our `multi-line context` to calculate the conjunction blocks individually:
```
{
  "actionActive": 
    not(performed in (null, false)) 
    and 
    deactivated in (null, false),
  "specialAction": 
    count(selectedList) > 0 
    and 
    selectedList[1].action in ("actionA", "actionC"),
  "result": not(actionActive or specialAction)
}.result
```