### Hash library

Hash is beta library that will serialize deeply nested collection in a blob. The blob can then
be stored in the DB or a file. That blob can then be unserialized back into a collection.

It also provides an alternate way of defining and populating new collection. This could be used to parse a 
configuration file.

My need for nested serialization is the result of a prototype circuit that never
moved to a table implementation. Collections in the prototype emulated the tables.

Since "collections to blob" does not handle nested collection, the former solution was to convert the collection 
to an objectTools object.  While it works, it is a little slow and used XML to initialize the collections. 
The XML was bumping up to the version 4.5 text limit. While it my be my objecttools to collection parse. This
method is almost 4 times faster (32k in 2 seconds versus 8 seconds)

I tried to use JSON but it also has the 32K text limit and I didn't feel like parsing a blob.

This approach uses a YAML like structure. The current version will serialize most simple data in the key value pairs.
Basically numbers, boolean, dates, pointer and text. It also will handle arrays of the simple types.

What is not implemented: time, blob, picture, undefined, subtable and 2d arrays

The key/value pairs also use the Ruby concept of a "symbol", which allows you to define words without quotes :a_word is 
converted to "a_word". It can be used on any string (key or value) that does not contain spaces and
a few special characters. I hate to use quotes which was one of the reasons for implementing symbols.

In my application the YAML like format is basically private, but a test routine at the end of the library
starts with a text version to create a collection.

In the configuration file or toCollection mode, you can just pass text to the unserialize or toCollection method and get the collection.

The rules are simple

* uses familiar 4D syntax for assignment with some short cuts to avoid text quotes and delimiter when not needed.
* lines with only a key will create a new node
* lines with the assignment format {:}key := {value} will add an element to the current node
* nesting is defined by prepended dashes "-" to define the nesting level (parent child relations) for example:

```
foo
-:when := date("11/11/11")
-:active := true
-why
--:because := t[I want to; I can; I am bored]
// This is a comment. The above is an array form.  text, integer, real, boolean, pointer and date arrays supported
--:note := "can also use word version of text array t[one two three], i.e. no semicolon delimiters"
bar
-:when := !12/12/12!
-:active := false
-:cost := 3745.23
```

With herdoc text, you can even use it to replace new collection with initial values

```
$collection := hash.toCollection("""
    foo
    -:when := date("11/11/11")
    -:active := true
    -why
    --:because := t[I want to; I can; I am bored]
    // This is a comment. The above is an array form.  text, integer, real, boolean, pointer and date arrays supported
    --:note := "can also use word version of text array t[one two three], i.e. no semicolon delimiters"
    bar
    -:when := !12/12/12!
    -   active := false
    // symbol : is optional for keys, so is white space
    -:cost := 3745.23
    -:records := i[787 5556 6657 1 33]
""")
```
