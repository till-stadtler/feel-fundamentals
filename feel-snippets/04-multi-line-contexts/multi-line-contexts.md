# Multi-Line Context Objects

You might have seen me using `multi-line contexts` in my previous posts. I love them! If your FEEL problem seems insurmountable, introduce a `multi-line context`!

Are you struggling with a complex expression with nested function calls? Use it!

Is your brain fried from trying to figure out a complex logical operation? Use it!

Do you want to reuse some results or even define a function? Use it!

Taking the last use case to the top: Calculate multiple intermediate results as part of a `multi-line context` in a script task. Then introduce output mappings for each end result you want to derive from them. The `multi-line context` automatically becomes a local variable, perfect for the intermediate results!

## Rewriting Complex Expressions

Let's start with some data from our zombie shelter. 

```jsonc
{
	"inhabitants": [
		{
			"firstName": "Beth",
			"lastName": "Brig",
			"infected": true,
			"health": 34
		}
		{
			"firstName": "John",
			"lastName": "Doe",
			"infected": false,
			"health": 76
		},
		// ...
	]
}
```

Now, we want to write out the full names of all inhabitants and sort them by last name.

```
// without proper formatting
string join(for inhabitant in sort(inhabitants, function(a, b) a.lastName < b.lastName)	return inhabitant.firstName + " " + inhabitant.lastName, "\n")
// with proper formatting
string join(
	for inhabitant
	in
	  sort(
			inhabitants,
			function(a, b) a.lastName < b.lastName
		)
	return inhabitant.firstName + " " + inhabitant.lastName,
	"\n"
)
```

Even with proper formatting, the expression is quite complex. Also, the outer function `string join()` is not that important, it's only about how to print the names, line-separated, comma-separated,... It's a bit sad that it's the first thing we see.

Let's introduce a `multi-line context`:

```
{
	"sortedInhabitants": 
	  sort(
			inhabitants,
			function(a, b) a.lastName < b.lastName
		),
	"concatNameList": 
	  for inhabitant 
		in sortedInhabitants 
		return inhabitant.firstName + " " + inhabitant.lastName,
	"joinedNameList":
	  string join(
			concatNameList,
			"\n"
		)
}.joinedNameList
```

Now, each steps is clearly labeled. And we also get a proper order of the steps. First, we sort, then we write out the full name, then we deal with the printing.

There is one complexity left in the expression above: how the full name is written out is mixed up with iterating over the `sortedInhabitants`. We can introduce a function to extract this logic.

```
{
	"printFullName": function(inhabitant) inhabitant.firstName + " " + inhabitant.lastName,
	"sortedInhabitants": 
	  sort(
			inhabitants,
			function(a, b) a.lastName < b.lastName
		),
	"concatNameList": 
	  for inhabitant 
		in sortedInhabitants 
		return printFullName(inhabitant),
	"joinedNameList":
	  string join(
			concatNameList,
			"\n"
		)
}.joinedNameList
```

This result is easy to understand and manage. Much better than what we started with! Of course, in the first version, without formatting, the expression was just one to two lines of code. But the expression being short does not equate to it being easy or quick to understand. 

With these `multi-line contexts`, reusability also improves. It is much easier to pick out and reuse the logic of an intermediate result in a `multi-line context` than to somehow extract it from a bundle of nested calls.


## Simplifying Logical Operations
In my post on [fun with formatting](../03-formatting-fun/formatting-fun.md), I already used `multi-line contexts` to rewrite logical operations. To revisit the main ideas:
* introduce a `multi-line context`
* split the logical operation at `or` disjunctions and calculate intermediate results
* combine the intermediate results in an end result variable
* output the end result of the `multi-line context`

In the following example, we use nested `multi-line context`s to simplify a complex logical operation. Here is an extended version of what we worked with previously.

```
not(performed in (null, false)) and deactivated in (null, false) or count(selectedList) > 0 and selectedList[1].action in ("actionA", "actionC") or not(userBlacklisted or userTimedOut or time(date and time(now(), "Europe/Berlin")).hour < 8)
```

We can rewrite this as:

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
	"userBlocked": 
	  userBlacklisted 
		or 
		userTimedOut 
		or 
		date and time(now(), "Europe/Berlin").hour < 8
  "result": actionActive or specialAction or not(userBlocked)
}.result
```

The new intermediate result `userBlocked` itself is a bit complex, well only the last part. So, let's introduce a nested `multi-line context` to simplify it:
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
	"userBlocked": 
	  userBlacklisted 
		or 
		userTimedOut 
		or 
		{
			"zonedNow": date and time(now(), "Europe/Berlin"),
			"currentHour": zonedNow.hour,
			"tooEarly": currentHour < 8
		}.tooEarly
  "result": actionActive or specialAction or not(userBlocked)
}.result
```

## Remarks
As you can see, you can introduce `multi-line context`s everywhere in your FEEL expressions. Regardless of the data type you need, you can always start a `multi-line context` and output exactly what you need.