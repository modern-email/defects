## Formats

### EML/IMF

General advice: ...

| Title                | Description                                                      | Reference                                        | Observed in | Reported | Status  | Comment                            |
|----------------------|------------------------------------------------------------------|--------------------------------------------------|-------------|----------|---------|------------------------------------|
|                      |                                                                  |                                                  |             |          |         |                                    |

## Protocols

### IMAP

Note: "[...] do not attempt to deduce command syntax from the command section alone; instead refer to the Formal Syntax section."

#### Defects

| Title                | Description                                                      | Reference                                        | Observed in | Reported | Status  | Comment                            |
|----------------------|------------------------------------------------------------------|--------------------------------------------------|-------------|----------|---------|------------------------------------|
| Zero UID             | UID MUST be >= 1 but was `0`                                     | `uniqueid = nz-number` RFC 3501                  | Outlook     | No       | Unknown |                                    |
| Negative line length | `body-fld-lines` should be >= 0 and < 4_294_967_296 but was `-1` | `body-fld-lines  = number` RFC 3501              | Dovecot     | No       | Unknown |                                    |
| Missing `text`       | `HIGHESTMODSEQ` status should have a `text` but didn't           | https://www.rfc-editor.org/rfc/rfc7162 section 7 | Gmail       | Yes      | Open    | Examples in RFC are wrong. Errata? |
| CHANGEDSINCE 0       | CHANGEDSINCE 0 was sent for an account on a mail server that was update to support CONDSTORE (did not have the CONDSTORE extension before). CHANGEDSINCE should have been omitted from the command. | chgsince-fetch-mod rfc/7162:2479 mod-sequence-value rfc/7162:2551 | iPhone Mail (16.5.1) | Yes      | Open    | |

#### Ambiguities

* `code` and `text` didn't play well ...

```
Greeting { kind: Ok, code: None, text: "[FOO] ..." }

... will result in ...

* OK [FOO] ...

... and be interpreted like ...

Greeting { kind: Ok, code: Foo, text: "..." }

And ...

Greeting { kind: Ok, code: None, text: "[...]" }

... can't be expressed.
```

* Unclear command continuation response(s).
  * When is base64 allowed in command continuation response?

```
`+ Rm9vbw==` could be interpreted as "Fooo" (base64) or just "Rm9vbw==" (text).
```




Created with https://www.tablesgenerator.com/markdown_tables
