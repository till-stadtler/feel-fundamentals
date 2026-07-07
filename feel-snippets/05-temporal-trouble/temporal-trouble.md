# Temporal Trouble

Dealing with temporal data types in FEEL can be quite troublesome. All other data types have a JSON representation. When a FEEL expression accesses some number, list or context, it directly knows what to do with it. But a temporal variable is saved as a string.

Check out this `execution context`:
```
{
	"deadline": "2026-06-26T09:00:00Z"
}
```

If we "blindly" use a comparison like `now() > deadline`, we just get `null`, representing an error case. Why? Because we compared a `date and time`value with a `string`. Obviously, `deadline` is a string, it just looks like a proper date and time value. But how should the FEEL engine know?

Similarly, when you have two such variables looking like date and time values at hand and you compare them, suddenly, you are performing a string comparison, which might work, or it might not work.

So, how do we deal with this issue? We need to tell the engine what we want! So, if `deadline` is saved as a string in the execution context, we need to do the following for a proper temporal comparison:
```
now() > date and time(deadline)
```

Now, the engine is able to interpret `deadline` as a `date and time` value.