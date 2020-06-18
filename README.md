# Webterm

Specs and discussions for the Webterm environment.

## Contribute

Webterm is still in development. Open issues and PRs to discuss the design of the shell grammar and environment.

**Active proposals:**

 - [#1 String quoting](https://github.com/beakerbrowser/webterm/issues/1)
 - [#2 JSON quoting](https://github.com/beakerbrowser/webterm/issues/2)
 - [#3 Page commands](https://github.com/beakerbrowser/webterm/issues/3)
 - [#4 Sub-invocations](https://github.com/beakerbrowser/webterm/issues/4)

## Current Grammar

The *current* shell grammar follows the following BNF:

```
// Webterm Input Grammar (PEG.js)
// ==============================

Expression
  = Term+
  
Term
  = term:Switch _? { return {type: 'param', key: term.key, value: term.value} }
  / term:String _? { return {type: 'token', value: term} }

Switch "switch"
  = '-'+ key:String _+ value:NonSwitch { return {key, value} }
  / '-'+ key:String { return {key, value: undefined} }

NonSwitch "non-switch"
  = !'-' value:String { return value }

String "string"
  = head:Char rest:CharOrQuote* { return head + rest.join('') }
  / '"' value:CharOr_OrSingleQuote* '"' { return value.join('') }
  / '\'' value:CharOr_OrDoubleQuote* '\'' { return value.join('') }

CharOr_OrDoubleQuote "character, whitespace, or double quote"
  = CharOr_
  / '"'

CharOr_OrSingleQuote "character, whitespace, or single quote"
  = CharOr_
  / '\''

CharOr_ "character or whitespace"
  = Char
  / _

CharOrQuote "character, single quote, or double quote"
  = Char
  / '\''
  / '"'

Char "character"
  = '\\ ' { return ' ' } // escaped space
  / [^ "'\f\n\r\t\v\u00a0\u1680\u2000-\u200a\u2028\u2029\u202f\u205f\u3000\ufeff]i // regular characters

_ "whitespace"
  = [ \t]+
```

## Current Execution Model

Every `shell-input` translates to a call of a single javascript function, called a "shell command function."

Every command function is expected to match the following signature:

```
function cmdname (opts:Object, ...args:any): Promise<any>
```

The `opts` argument is an object which contains all switch inputs (`short-switch` and `long-switch`). The remaining `args` are an ordered list of the `argument` inputs.

If the function's resolved value possesses a `toHTML()` function, that function will be called during the result rendering and its value will be used in the shell output.
