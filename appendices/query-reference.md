---
description: Axon Server Query Language
---

# E. Axon Server Query Language

## Principles

The Axon Server query language processes a stream of events. Processing steps include filters and projections, defined in a pipeline. The query engine executes each step in the pipeline and forwards the result to the next step. The result of the last step is returned. The idea is based on the UNIX pipe commands.

The input of a query is a stream of events with the following fields:

* token - a unique sequence number for an event
* aggregateIdentifier - the unique identifier for the aggregate
* aggregateSequenceNumber - sequence number of the event for the aggregate
* aggregateType - the type of the aggregate
* payloadType - the type of the payload of the event
* payloadRevision - version number of the payload type
* payloadData - content of the event, the format of this depends on the serializer used to store the data
* timestamp - time when the event was created (milliseconds since 1970/01/01).

### Filters

Filters are expressions that evaluate to either true or false. Basic filter operations do comparisons between fields and other fields or fixed values. The following are samples of valid filters:

```
token > 1000000
aggregateIdentifier = "1234"
payloadType = aggregateType
```

Basic operators:

* \=
* \>
* \&lt;
* != or <>
* \>=
* \&lt;=
* in

All comparison operators expect either the same type, or performs a String comparison.

Filter expressions can be combined using the logical operators:

* and
* or
* not

You can use parenthesis in the expression to change the evaulation orders.

In expressions you can use basic arithmetic operators:

*
  *
*
  *
* \*
* /
* %

All expect for the '+' only work on numeric values. '+' on String does a concatenation.

Apart from these operators there are 2 matching functions:

*   contains: if both parameters are string values it is true when the first contains the second. If the first parameter is

    a list it returns true if the list contains the second value.
* match: compares the value of the first paramater to a regular expression (regexp format same as in Java).

Function names may be used in the traditional ways, but for binary functions also in infix mode. So the following two samples are both valid:

```
contains(payloadData, "Smith")
payloadData contains "Smith"
```

### Projections

Projection functions change the shape of the data. The following projection functions are available:

* select - map each element in the stream to a new element, for instance with less fields or with calculated fields.
* groupby - map elements with the same value for the group by fields to a new element.
*   count - counts the number of elements in the stream, when used with a parameter counts the number of non-null/true fields for

    the parameter value.
* min - minimum value for the paramater value
* max - maximum value for the paramater value
* avg - average value for the paramater value

The count, min, max and avg functions can also be used inside a group by.

### Examples

```
select(payloadType, aggregateType, aggregateSequenceNumber, hour(timestamp) as time)
```

Returns only the payloadType, aggregateType, aggregateSequenceNumber fields and the hour of the timestamp for each event.

```
groupby(payloadType, count())
```

Counts the number of events per payloadType.

```
groupby([payloadType, aggregateType], count(), min(aggregateSequenceNumber))
```

Counts the number of events and finds the minimum aggregateSequenceNumber per combination of payloadType, aggregateType.

```
count(aggregateSequenceNumber > 100)
```

Counts the number of events with aggregateSequenceNumber > 100.

### Other functions

*   xpath(data, expression \[,resultType]) - performs an xpath function on the first parameter value. Data must contain XML

    (so depends on the serializer used for events). The resultType may be specified to indicate if you want to have an XML

    node, an XML nodelist, a string or a number returned.
* jsonpath(data, expression) - performs a jsonpath function on the first parameter value. Data must contain JSON
* formatDate(data, format \[,timezone]) - converts a timestamp value to a readable date
* concat(listData, delimiter) - concatenates all elements in the listData to a single string, with delimiter between the elements.
* left( data, n) - returns the first n characters from data. If data is shorter than n it returns the whole string,
* right( data, n) - returns the last n characters from data. If data is shorter than n it returns the whole string,
* length( data) - returns the length of the string
* lower( data) - converts string to lowercase
* upper( data) - converts string to lowercase
*   substring( data, first \[, last]) - returns substring from _first_ to end of string or _last_ (exclusive). If string is shorter

    than first it returns an empty string.
* hour(timestamp)
* minute(timestamp)
* day(timestamp)
* week(timestamp)
* month(timestamp)
* year(timestamp)

### Examples

```
select( xpath(payloadData, "//customerId") as customerId)
```

Gets the first customerId in the payloadData.

```
xpath(payloadData, "count(//customerId)", "NUMBER") > 10
```

Returns events with more than 10 customerId elements in the payload.

```
select(jsonpath(payloadData, "$.book[*].title") as titles)
```

Gets the titles for all books.

### Pipeline

Expressions can be put together in a pipeline

```
aggregateType contains "abcde" | groupby(payloadType, count())
```

or even more steps:

```
aggregateType contains "abcde" | groupby(payloadType, count() as count) | count > 10
```

### Time constraints

When an event store contains many millions of events it is usually not required to search through all the events. You can add time constraints to the pipeline to only search recent events.

* last X minutes
* last X hours
* last X days
* last X weeks
* last X months
* last X years

```
last 2 minutes
```

Returns all events from the last 2 minutes.

```
aggregateSequenceNumber = 0 | last hour
```

All events with aggregateSequenceNumber 0 for the last hour.

```
last minute | groupby(payloadType, count())
groupby(payloadType, count()) | last minute
```

These 2 are the same. The time constraint may be anywhere in the pipeline, always applies to the timestamp of the event.
