# SRI Generic Query Specification

## Introduction

To allow filtering on list resources, [SRI][sri] describes the server should implement a set of URL parameters, to allow the client to restrict the list of items retrieved. Besides stating that a number of sensible data access paths should be supported, it does not specify any specifics. This specification describes a standard filtering method, that standardizes basic filtering. It allows the list resource to be restricted on any of the direct keys in the resource, and allows the client to apply operators to it's value. These basic filters should take this form :

  [key][operator]=[value]

For example, to restrict the list resource to only include items where publication date is before 1/1/2015 :

  GET /items?issuedBefore=2015-01-01T00:00:00Z

In this example *key* is `issued`, *operator* is `Before`, and *value* is `2015-01-01`.

The basic filters SHOULD BE applicable to any direct key of these types :

* `string`
* `timestamp` The *value* section should be a timestamp as specified in RFC3339
* `numeric`
* `array` The *value* section should contain a comma-separated list of one of the above 3 types

The supported `operator` values are :

* If no *operator* is specified (i.e. GET /items?publicationDate=2015-01-01T00:00:00+02:00). 
** For `strings` only resources that have a case insensitive match SHOULD BE selected.
** For `numeric` and `timestamp` only resources that have this exact value SHOULD BE selected.
** For `array` all resources MUST BE selected where the array in the resource is the same as the array that corresponds to `value`.

* `Greater`
** For strings, this does a case insensitive comparison, and selects resources that have a string [key] with sort order greater than [value]
For timestamps and numerics, it selects items that have [key] > [value]
GreaterOrEqual, After
For strings, this does a case insensitive comparison, and selects resources that have [key] with sort order greater than, or equal to [value].
For timestamps and numerics , it selects resources that have [key] >= [value]
Less
For strings, this does a case insensitive comparison, and selects resources that have [key] with sort order less than [value]
For dates and numerics, it selects resources that have [key] < [value]
LessOrEqual, Before
For strings, this does a case insensitive comparison, and selects resources that have [key] with sort order less than, or equal to [value]
For timestamps and numerics, it selects resources that have [key] <= [value]
In : Selects resources where [key] (string, numeric or timestamp) is any of [value]. Where [value] is a comma-separated list of items.

* Contains
For strings, selects resources that contains (case insensive) a substring [value]
For arrays, it returns resources where the resource has an array that contains the sub-array (possibly a single value) that is formed by [value] (being a comma-separated list of strings, numerics, or timestamps).
For DCMI-Period values, it returns those resources where [value] (an RFC3339 timestamp) is between start and end (inclusive).
RegEx
For strings, selects resource that match (case insensitive) a POSIX regular expression [value].

Any of the operators (including [none]) above can be extended with CaseSensitive as a prefix, this changes their behaviour to be case sensitive.

On top of the operator prefix CaseSensitive, all operator can be postfix with Not, to invert the operator’s logic.

Examples : 

GET /items?firstNameCaseSensitiveNotContains=ike
GET /items?firstName=mike
GET /items?firstNameNot=mike
GET /items?firstNameCaseSensitive=Mike
GET /items?publicationDateBefore=2015-01-10
GET /items?publicationDateContains=2015-04-01
GET /items?tagsContains=news
GET /transactions?amountLessEqual=0
GET /transactions?amountGreater=100000

Besides the general filters above it is advised that servers implemented a simple full-text search. They should do this by implementing a parameter q. The value of the parameter can have multiple keywords, separated by a plus (+) sign.

Example : http://api.vsko.be/persons?q=Mieke+Heck

Searching through this q parameter should by definition be a broad search. So at least case-insensitive matches on sub-strings should be supported. Typically this list resource will be used to provide auto-completion in a user interface, so speed is very important. 

Implementations of q should remain simple. The Search API is provided for very advanced full-text search operations.

Specifying a criterium (key + operator) more than once results in all operations being applied in additive (AND) fashion.


[sri]: https://github.com/dimitrydhondt/sri
