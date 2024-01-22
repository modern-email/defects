# Formats

## EML/IMF

...

# Protocols

## IMAP

Note: "[...] do not attempt to deduce command syntax from the command section alone; instead refer to the Formal Syntax section."

### Observed defects

<details>
<summary>Negative line length breaks parsing</summary>
An IMAP server tells the number of lines in a message using the `body-fld-lines` ABNF rule:

```abnf
body-fld-lines = number
number         = 1*DIGIT
                   ; Unsigned 32-bit integer
                   ; (0 <= n < 4,294,967,296)
```

According to the standard, negative values are not allowed. Still, Dovecot was observed sending `-1` breaking parsing.

* Observed in: Dovecot (unknown version)
* Reported: No
* Status: Unknown
* Comment: None
* Proposed solution(s):
	* Accept `-1`, emit warning, and rectify to `0` (implemented in [imap-codec])
</details>

<details>
<summary>Missing `text`</summary>
Status responses in IMAP MUST have a non-empty `text` (and preceeding space).

```abnf
resp-text = ["[" resp-text-code "]" SP] text
text      = 1*TEXT-CHAR
TEXT-CHAR = <any CHAR except CR and LF>
```

It was observed that Gmail has neither the `SP` nor the `text` when using `HIGHESTMODSEQ`.

* Observed in: Gmail (Sep. 27, 2023)
* Reported: Yes
* Status: [Open](https://issuetracker.google.com/issues/289877509)
* Comment: The examples in RFC 7162 are wrong, see [errata](https://www.rfc-editor.org/errata/eid5055).
* Proposed solution(s):
	* Wait for Gmail to fix it (or comment on issue)
	* Replace with a sentinel value and log warning (implemented in [imap-codec])
	* Discard, but make sure not to reproduce the defect
</details>

<details>
<summary>CHANGEDSINCE 0</summary>

CHANGEDSINCE 0 was sent for an account on a mail server that was update to support CONDSTORE (did not have the CONDSTORE extension before). CHANGEDSINCE should have been omitted from the command.

```abnf
mod-sequence-value = 1*DIGIT
                      ;; Positive unsigned 63-bit integer
                      ;; (mod-sequence)
                      ;; (1 <= n <= 9,223,372,036,854,775,807).
```

* Observed in: iPhone Mail (16.5.1)
* Reported: Yes
* Status: Open
* Comment: None
* Proposed solution(s): None
</details>

<details>
<summary>Not all variants of empty ID list are accepted</summary>

IMAP's ID command allows `()`, and `nil` to encode an empty parameter list:

```abnf
id-params-list = "(" [field-value *(SP field-value)] ")" / nil
field-value    = string SP nstring
```

See https://github.com/modern-email/defects/issues/12#issuecomment-1841669856.

Some servers don't recognize both variants.

* Observed in: Nemesis (GMX, Web.de, ...), and Exchange (See https://github.com/modern-email/defects/issues/12#issuecomment-1845491532)
* Reported: No
* Status: Unknown
* Comment: None
* Proposed solution(s):
	* Don't send an empty ID command. Note: Proxies may require extra attention!
</details>

#### Ambiguities

<details>
<summary>`code` and `text` are ambiguous</summary>

```rust
Greeting { kind: Ok, code: None, text: "[FOO] ..." }
```

... will result in ...

```imap
* OK [FOO] ...
```

... and be interpreted like ...

```rust
Greeting { kind: Ok, code: Foo, text: "..." }
```

And ...

```rust
Greeting { kind: Ok, code: None, text: "[...]" }
```

... can't be expressed.
</details>

<details>
<summary>Unclear command continuation response(s)</summary>
Unclear when base64 is allowed in command continuation responses.

`+ Rm9vbw==` could be interpreted as `Fooo` (base64) or just `Rm9vbw==` (text).
</details>

[imap-codec]: https://github.com/duesee/imap-codec
[mox]: https://github.com/mjl-/mox
