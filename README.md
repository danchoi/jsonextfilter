# jsonextfilter

Applies external unix filters to specified key paths of a JSON object stream.

## Example

example.json:

```json
{
  "title": "Terminator 2: Judgment Day",
  "year": 1991,
  "stars": [
    {"name": "Arnold Schwarzenegger"},
    {"name": "Linda Hamilton"}
  ],
  "ratings": {
    "imdb": 8.5
  },
  "description":"<p>Some <strong>HTML</strong></p>"

}
{
  "title": "Interstellar",
  "year": 2014,
  "stars": [
    {"name":"Matthew McConaughey"},
    {"name":"Anne Hathaway"}
  ],
  "ratings": {
    "imdb": 8.9
  },
  "description":"<p>Some <strong>more HTML</strong></p>"
}
```

We want to transform the "description" fields from HTML to plain text:

```bash
jsonextfilter 'elinks -dump'  'description' < example.json  | jq -M '.' 
```

Output:

```json
{
  "ratings": {
    "imdb": 8.5
  },
  "stars": [
    {
      "name": "Arnold Schwarzenegger"
    },
    {
      "name": "Linda Hamilton"
    }
  ],
  "year": 1991,
  "title": "Terminator 2: Judgment Day",
  "description": "   Some HTML\n"
}
{
  "ratings": {
    "imdb": 8.9
  },
  "stars": [
    {
      "name": "Matthew McConaughey"
    },
    {
      "name": "Anne Hathaway"
    }
  ],
  "year": 2014,
  "title": "Interstellar",
  "description": "   Some more HTML\n"
}
```

More than one keypath can be specified in the keypaths argument string. Separate keypaths with spaces. The external filter will be applied to all of them, e.g.

```bash
jsonextfilter 'elinks -dump'  'description review' < example.json  | jq -M '.' 
```

Currently only ONE external filter command can be given. If you want to apply a pipeline of commands, wrap it in a bash script.

The external filter will only be applied to STRING values. Number and boolean values are untouched.

## Preconditions

You can designate a pre-filter to determine whether the main filter should run
depending on the exit code of the pre-filter:

```bash
jsonextfilter -p 'grep -q more' 'elinks -dump' description  < example.json   | jq '.' -M
```

This causes `elinks -dump` to be applied only to "description" values that contain the string "more":

```json
{
  "ratings": {
    "imdb": 8.5
  },
  "stars": [
    {
      "name": "Arnold Schwarzenegger"
    },
    {
      "name": "Linda Hamilton"
    }
  ],
  "year": 1991,
  "title": "Terminator 2: Judgment Day",
  "description": "<p>Some <strong>HTML</strong></p>"
}
{
  "ratings": {
    "imdb": 8.9
  },
  "stars": [
    {
      "name": "Matthew McConaughey"
    },
    {
      "name": "Anne Hathaway"
    }
  ],
  "year": 2014,
  "title": "Interstellar",
  "description": "   Some more HTML\n"
}
```

