# Design

`log-monster` should by very fast at consuming log data.  Our design should try to minimize misuse while providing as much flexibility as possible.  To that end, `log-monster` should consist of a front-end REST API and a back-end logging strategy.  The back-end can be implemented as a library that implements the functionality of the REST API in a certain way.  We can then add support for swapping out back-ends as needed.

## Logging levels

- `trace` for extremely verbose tracing of control flow logic or similar
- `debug` for information that might be useful for debugging purposes
- `info` for information that is good to keep track of
- `warn` for warnings about potentially dangerous situations
- `error` for errors that have occurred

## Endpoints

Each logging level should have its own endpoint so that misuse is obvious (a 404 response). Each endpoint should be identical in all other respects.

Each endpoint should accept data in the following format:

```json
{
  "group": "A string used to group logs together.",
  "occurred": "2020-01-01T01:00+00:00",
  "text": "The text you wish to be logged.",
}
```

The `group` is just a of key to use when grouping different logs together.  Back-ends may use this however they want to.  Some might use it as a file name, some might use it as a grouping field in a database, etc.

The `occurred` is when the log occurred on the machine that is making the current request.  This is different from when that request was received by `log-monster`.  This should be in the format shown above.

The `text` is just the text that will be logged using the current back-end.  

Example request:
```
POST /trace HTTP/1.1
...
Content-Type: application/json
...

{
  "group": "A string used to group logs together.",
  "occurred": "2020-01-01T01:00+00:00",
  "text": "The text you wish to be logged.",
}
```
