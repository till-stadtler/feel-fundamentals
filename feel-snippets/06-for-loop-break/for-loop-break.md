# For Loop Breaks in FEEL

Recently, in my project, I came about an interesting use case. Given a sorted list of objects and some input parameters, provide the count of uninterrupted objects in the list, from the start, that match the provided input parameters.

As soon as an object does not match the provided parameters, stop the count.

In other programming languages, we can introduce a local variable, loop through the list and break out of the loop once we find an object that does not match the provided parameters.

In FEEL, this is not possible. You can't introduce a local variable and update its value later in the expression. You also can't break a for loop. In FEEL, a for loop always returns a list of some results you specify.

So, how did I approach this issue?

## Intermediate Results

First of all, the issue is quite complex, so I assume I am going to need a `multi-line context object` to get some intermediate results.

My first intermediate result is a list of boolean values that simply represent if an object at a specific index matches the provided parameters or not:

```
{
	"matchList":
	  for item
	  in sortedList
	  return item.field = providedParameter
}
```

My result might look like this: `[true, true, false, true]`.

From this intermediate result, I know that the answer is 2. The first two objects match the parameters. The third does not. The fourth does not matter.

Now, I would have liked to have a local variable and loop through the list. Well, if I had that, I wouldn't need this intermediate result in the first palce. But I don't have that. So, I came up with the following solution.

## Sublists and all()

It is clear that we need another for loop. This time, it should check if all booleans from start to the current loop position are true. If yes, save the loop position. If no, set something else, e.g., 0.

To achieve this, I create sublists of my `matchList` as I loop through it. The first sublist is just a list with the first boolean: `[true]`. My loop position is 1. So, my result so far would be: `[1]`. The next sublist is `[true, true]`. So far, the result would be: `[1, 2]`. Now, the next sublist is `[true, true, false]`. Not all booleans are true, so I just set 0: `[1, 2, 0]`. Similarly for the fourth sublist, representing the whole `matchList`.

In FEEL, this looks like:

```
{
	"matchList":
	  for item
	  in sortedList
	  return item.field = providedParameter,
	"continuedCount":
	  for i
	  in 1..count(matchList)
	  return
		  if all(sublist(matchList, 1, i))
		  then i
		  else 0
}
```

Using the range `1..count(matchList)` and get a list like `[1, 2, 3, 4]`, so each item represents the loop position.

With `sublist(matchList, 1, i)`, I specify the desired sublist, starting at `1` (the first item in FEEL) with the length `i`.

The function `all()` just checks if all booleans in the list are true.

With `if then else`, I either set the loop position in the resulting list, or I set 0.

So, my `continuedCount` at this point is `[1, 2, 0, 0]`. Now, I simply need to take the `max()` to get the answer to my initial question: 2!


## Remarks

In FEEL, it is not possible to introduce a local variable. Instead, we have the resulting list of the for loop providing the value of some calculation for each loop position.

In FEEL, we can't break out of a loop. Instead, we set some dummy results in the list to ignore later.

To summarize: I circumvented the limitations of FEEL by using the power of for loops in combination with sublists. Hurray!