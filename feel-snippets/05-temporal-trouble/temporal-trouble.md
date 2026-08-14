# Temporal Trouble

Dealing with temporal data types in FEEL can be quite troublesome. All other data types have a JSON representation. When a FEEL expression accesses some number, list or context, it directly knows what to do with it. But a temporal variable is saved as a string.

Check out this `execution context`:
```
{
	"deadline": "2026-06-26T09:00:00Z"
}
```

If we "blindly" use a comparison like `now() > deadline`, we just get `null`, representing an error case. Why? Because we compared a `date and time` value with a `string`. Obviously, `deadline` is a string, it just looks like a proper date and time value. But how should the FEEL engine know?

Similarly, when you have two such variables looking like date and time values at hand and you compare them, suddenly, you are performing a string comparison, which might work, or it might not work.

So, how do we deal with this issue? We need to tell the engine what we want! So, if `deadline` is saved as a string in the execution context, we need to do the following for a proper temporal comparison:
```
now() > date and time(deadline)
```

Now, the engine is able to interpret `deadline` as a `date and time` value.

## Duration Trouble

FEEL has two different durations:

1. days and time duration
2. years and months duration

In most use cases, you will work with the `days and time duration`. It is important to always keep them separate!

So, instead of writing `duration("P1Y3MT2H30M")`, you should use `duration("P1Y3M")` and `duration("PT2H30M")` separately. For example when adding the duration to a date and time value:

```
now() + duration("P1Y3M") + duration("PT2H30M")
```

It is important to understand that the `days and time duration` is the default choice of the FEEL engine. For example, when subtracting two date and time values or two date values from one another, you get a `days and time duration`:

```
// expression
now() - date and time("2026-08-13T10:00:00+02:00")
// result
"P1DT2M21S"
```

Reminder: I wrote the result as a string value. But as long as this value is kept in the FEEL engine, e.g., as part of a multi-step calculation, the engine knows it's a `days and time duration`. One you save it, it becomes a string value!

If you want to know how many `years and months` lie between two date and time values, you need to use a conversion function. This conversion function does not exist for `days and time duration`s, as they are the default.

```
// expression
years and months duration(now, date and time("2023-02-13T10:00:00+02:00"))
// result
"-P3Y6M"
```

This result only has the properties "years" and "months". You can't get any other information.

If you want to know the full days that lie between these two date and time values, subtract them, then get the days property from the `days and time duration`:

```
// expression
(now() - date and time("2023-02-13T10:00:00+02:00")).days
// result
1278
```

To summarize: 

* if you are working with dates, e.g., always the 15th day of a month in a calendar year, you can use the `years and months duration`, easily adding or subtracting a month, regardless of its specific length (28 vs. 30 vs. 31 days).
* if you are interested in the number of days, hours, minutes,... between two points in time, you have to use the `days and time` duration. This duration easily covers longer periods of time, but it always uses days as the maximum unit to describe them.
* keep the two durations separate, they don't mix well. If you are ever inclined to mix them, check your use case again, you are probably doing something wrong

## Properties

Check the Camunda FEEL documentation to see which properties there are for the various temporal data types. 

The properties are mostly self-explanatory. If you have a temporal value that describes a point in time, like `date`, `time` or `date and time`, you can get (in singular) the year, month, day, hour, minute, and second. Of course, a date won't have an hour or minute property, and a time won't have a year property.

If you have a duration, you can get (in plural) the years, months, days, hours, minutes and seconds. A `years and months duration` only has the years and months properties. And a `days and time duration` only has the days, hours, minutes and seconds properties.

## Math

When working with durations, you can also use numbers to multiply or divide them. Or even divide the durations to get a resulting number.

You might also need `abs()` to get the absolute value, e.g., of a property having a negative value.


# Timezones

When working with date and time variables, using time zones is important. If you want to get the current hour, you can't simply write `now().hour`. This will always return the hour at UTC. So, to correctly get the current hour at your location, you need to use your time zone id. For me that is:

```
date and time(now(), "Europe/Berlin").hour
```

This is a conversion function with two arguments. The first is a date and time value, which should describe a defined instant, so it also need information about its time zone. The second is the time zone id of your location.

Only after the conversion, the hour reflects the correct time at my location.

If you try to compare two date and time values, make sure that both either contain no information about the time zone or both do contain information about the time zone. Of course, if they contain no information, the date and time values should still make sense. But you can't compare one date and time value with time zone information with a date and time value without it!