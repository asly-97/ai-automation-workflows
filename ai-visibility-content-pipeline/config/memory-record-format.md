# Memory record format

What the publish agent saves at the end of every run (via `save_turn`). The draft agent reads these back with `get_context` the next morning.

```
RUN: <date>
TARGET QUESTION: <question>
ANGLES DRAFTED:
  A) <angle>
  B) <angle>
  C) <angle>
PICKED: <A/B/C>
WHY THE OTHERS LOST: <one line each>
NOTES (verbatim): <my notes, or none>
LIVE URL: <url>
INDEXNOW: <response code>
```

The ranking agent saves one line per question per check:

```
RANK CHECK <date time>: <question> | <position or not ranked> | <top domains> | <change>
```

Plain text on purpose. It's easy for any agent to read, and easy for me to search when I want to know why a past article was written the way it was.
