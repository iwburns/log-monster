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
  "trace": [{
    'file': '',
    'line': 1
    'method': '',
  }]
}
```

The `group` is just a of key to use when grouping different logs together.  Back-ends may use this however they want to.  Some might use it as a file name, some might use it as a grouping field in a database, etc.

The `occurred` is when the log occurred on the machine that is making the current request.  This is different from when that request was received by `log-monster`.  This should be in the format shown above.

The `text` is just the text that will be logged using the current back-end.

The `trace` is an array of objects that show where something came from.

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
## Internals

The `log-monster` package should be made up of three things:

- `log-monster` itself, which should be a light-weight wrapper around a logging mechanism
- `log-monster-types` which should be a Typescript package specifying the API of all backend logging packages
- `log-monster-backend-*` which should be a library implementing the `log-monster-types` API.  There could be many of these, each with their own logging strategy.

### `log-monster`

`log-monster` should be as light-weight as possible and should have a fairly small API

| verb   | resource        | desc                           |
|--------|-----------------|--------------------------------|
| `POST` | `/trace`        | insert a trace-level log       |
| `POST` | `/debug`        | insert a debug-level log       |
| `POST` | `/info`         | insert a info-level log        |
| `POST` | `/warn`         | insert a warn-level log        |
| `POST` | `/error`        | insert a error-level log       |
| `POST` | `/search/logs/` | select logs by group and level |

### `log-monster-types`

`log-monster-types` should be small and focused as well.  Here we should define the types of functions and objects that each backend should know how to deal with.

Rough sketch:

```typescript
export enum Level {
  Trace,
  Debug,
  Info,
  Warn,
  Error,
}

export interface Entry {
  level: Level,
  group: String,
  occurred: Date,
  text: String,
}

export function createEntry(level: Level, group: String, occurred: Date, text: String) => Entry {
  return {
    level,
    group,
    occurred,
    text,
  };
}

export abstract class LoggingStrategy {
  constructor() {}
  
  abstract log(entry: Entry): boolean; // not sure if I like the boolean, or if this should return a Result<T, E> -ish thing
  abstract find_logs(level: Level, group: String): Array<Entry>
}
```

### `log-monster-backend-*`

Each `log-monster-backend-*` package should export a class extending the `LoggingStrategy` class above.  Each backend is free to implement its strategy however it sees fit, optimizing for different use-cases along the way.
