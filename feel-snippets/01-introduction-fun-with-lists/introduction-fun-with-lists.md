# Introduction – Fun With Lists
<div>
<img src="./1.png" alt="FEEP Introduction Slide 1" width="300">
<img src="./2.png" alt="FEEP Introduction Slide 2" width="300">
<img src="./3.png" alt="FEEP Introduction Slide 3" width="300">
</div>
</br>
<div>
<img src="./4.png" alt="FEEP Introduction Slide 4" width="300">
<img src="./5.png" alt="FEEP Introduction Slide 5" width="300">
<img src="./6.png" alt="FEEP Introduction Slide 6" width="300">
</div>

## Explanations
Let's take a closer look at this problem!

A FEEL expression is always evaluated as part of an `execution context`, e.g., the available variables in a process instance. In this case, we assume that a `list` of `context objects` called `jungle` is available.

In FEEL the following structure is called `context object`:
```json
{
  "type": "snake",
  "mood": "depressed"
}
```

Our `execution context` looks like this:
```jsonc
{
  "jungle": [
    {
      "type": "snake",
      "mood": "depressed"
      //...
    },
    {
      "type": "parrot",
      "mood": "grumpy"
      //...
    }
    //...
  ]
  //...
}
```

Each `context object` can have additional fields. The `jungle` will contain more animals, of course. And the `execution context` can contain more variables.

### The Issue
Now, if we go back to the problematic FEEL expression:
``` 
some parrot
in jungle
satisfies parrot.mood = "grumpy"
```
This structure is a combination of iterating over a list and checking if any  expression evaluated for each item in the list results in true.

There is no pressing need to write this expression in three separate line. It does make sense for now, because we will work on each of the three elements on the right-hand side.

As we can see from the `execution context`, not each animal in the `jungle` is a parrot. You could even argue that `jungle` is not a suitable name if it just contains information about animals. There are many ways to make this expression semantically and syntactically correct. At the same time, it has to be correct in terms of the logic it is supposed to capture: There is one or more parrot in the jungle with the mood `grumpy`.

### The Solution
Let's look at some solutions!

By not using `parrot` as the `iterator variable`, but instead using `animal`, we can find the following solution:
```             
some animal
in jungle
satisfies 
  animal.type = "parrot" 
  and 
  animal.mood = "grumpy"
```
An additional check for the `type` is necessary. We use logical conjunction by using the keyword `and`.

While we lose the focus on the parrot as the `iterator variable`, the expression that is applied to each item in the list is concise and easy to understand.

---

We can keep the parrot as the `iterator variable` by ensuring that the list we iterate over only contains parrots.
```               
some parrot
in jungle[type = "parrot"]
satisfies parrot.mood = "grumpy"
```
We have now filtered the `jungle` to get a list of `context objects` that fulfils the expression `type = "parrot"`.

While there are other options, the two solutions above are the most suitable. One focusing on all animals, the other on parrots.

### The Alternatives
We can also make use of different FEEL functions and find:
```    
// any() plus for loop instead of some in satisfies           
any(
  for animal 
  in jungle 
  return 
    animal.type = "parrot" 
    and 
    animal.mood = "grumpy"
)
// or
any(
  for parrot 
  in jungle[type = "parrot"] 
  return parrot.mood = "grumpy"
)
```
Or:
```
count(jungle[type = "parrot" and mood = "grumpy"]) > 0
```
We can take a closer look at these alternatives in a future post.

## Try It Yourself
You can jump directly into the new Scala FEEL Playground with these links:

* [Some animal with unfiltered list](https://feel-playground.camunda.com/playground/index.html?expression-type=expression&expression=c29tZSBhbmltYWwKaW4ganVuZ2xlCnNhdGlzZmllcwogIGFuaW1hbC50eXBlID0gInBhcnJvdCIgCiAgYW5kIAogIGFuaW1hbC5tb29kID0gImdydW1weSI%3D&context=ewogICJqdW5nbGUiOiBbCiAgICB7CiAgICAgICJ0eXBlIjogInNuYWtlIiwKICAgICAgIm1vb2QiOiAiZGVwcmVzc2VkIgogICAgfSwKICAgIHsKICAgICAgInR5cGUiOiAicGFycm90IiwKICAgICAgIm1vb2QiOiAiZ3J1bXB5IgogICAgfQogIF0KfQ%3D%3D)
* [Some parrot with filtered list](https://feel-playground.camunda.com/playground/index.html?expression-type=expression&expression=c29tZSBwYXJyb3QKaW4ganVuZ2xlW3R5cGUgPSAicGFycm90Il0Kc2F0aXNmaWVzIHBhcnJvdC5tb29kID0gImdydW1weSI%3D&context=ewogICJqdW5nbGUiOiBbCiAgICB7CiAgICAgICJ0eXBlIjogInNuYWtlIiwKICAgICAgIm1vb2QiOiAiZGVwcmVzc2VkIgogICAgfSwKICAgIHsKICAgICAgInR5cGUiOiAicGFycm90IiwKICAgICAgIm1vb2QiOiAiZ3J1bXB5IgogICAgfQogIF0KfQ%3D%3D)
* [Any animal](https://feel-playground.camunda.com/playground/index.html?expression-type=expression&expression=YW55KAogIGZvciBhbmltYWwgCiAgaW4ganVuZ2xlIAogIHJldHVybiAKICAgIGFuaW1hbC50eXBlID0gInBhcnJvdCIgCiAgICBhbmQgCiAgICBhbmltYWwubW9vZCA9ICJncnVtcHkiCik%3D&context=ewogICJqdW5nbGUiOiBbCiAgICB7CiAgICAgICJ0eXBlIjogInNuYWtlIiwKICAgICAgIm1vb2QiOiAiZGVwcmVzc2VkIgogICAgfSwKICAgIHsKICAgICAgInR5cGUiOiAicGFycm90IiwKICAgICAgIm1vb2QiOiAiZ3J1bXB5IgogICAgfQogIF0KfQ%3D%3D)
* [Any parrot](https://feel-playground.camunda.com/playground/index.html?expression-type=expression&expression=YW55KAogIGZvciBwYXJyb3QgCiAgaW4ganVuZ2xlW3R5cGUgPSAicGFycm90Il0gCiAgcmV0dXJuIHBhcnJvdC5tb29kID0gImdydW1weSIKKQ%3D%3D&context=ewogICJqdW5nbGUiOiBbCiAgICB7CiAgICAgICJ0eXBlIjogInNuYWtlIiwKICAgICAgIm1vb2QiOiAiZGVwcmVzc2VkIgogICAgfSwKICAgIHsKICAgICAgInR5cGUiOiAicGFycm90IiwKICAgICAgIm1vb2QiOiAiZ3J1bXB5IgogICAgfQogIF0KfQ%3D%3D)
* [Count filtered list](https://feel-playground.camunda.com/playground/index.html?expression-type=expression&expression=Y291bnQoanVuZ2xlW3R5cGUgPSAicGFycm90IiBhbmQgbW9vZCA9ICJncnVtcHkiXSkgPiAw&context=ewogICJqdW5nbGUiOiBbCiAgICB7CiAgICAgICJ0eXBlIjogInNuYWtlIiwKICAgICAgIm1vb2QiOiAiZGVwcmVzc2VkIgogICAgfSwKICAgIHsKICAgICAgInR5cGUiOiAicGFycm90IiwKICAgICAgIm1vb2QiOiAiZ3J1bXB5IgogICAgfQogIF0KfQ%3D%3D)