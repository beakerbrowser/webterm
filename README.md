# Webterm

Specs and discussions for the Webterm environment.

## Contribute

Webterm is still in development. Open issues and PRs to discuss the design of the shell grammar and environment.

Active proposals:

 - [String quoting](https://github.com/beakerbrowser/webterm/issues/1)
 - [JSON quoting](https://github.com/beakerbrowser/webterm/issues/2)
 - [Page commands](https://github.com/beakerbrowser/webterm/issues/3)
 - [Sub-invocations](https://github.com/beakerbrowser/webterm/issues/4)

## Current Grammar

The *current* shell grammar follows the following BNF:

```
shell-input  ::= command, ( " ", inputs )* ;
inputs       ::= short-switch | long-switch | argument ;
short-switch ::= "-", ( alpha | digit ), " "?, switch-value? ;
long-switch  ::= "--", switch-key, " ", switch-value? ;
switch-key   ::= ( alpha | digit ), ( alpha | digit | "-" | "_" )* ;
switch-value ::= ( alpha | digit ), ( char )* ;
argument     ::= ( alpha | digit ), ( char )* ;
alpha        ::= ? A-Z a-z ? ;
digit        ::= ? 0-9 ? ;
char         ::= ? all visible characters ? ;
```

## Current Execution Model

Every `shell-input` translates to a call of a single javascript function, called a "shell command function."

Every command function is expected to match the following signature:

```
function cmdname (opts:Object, ...args:any): Promise<any>
```

The `opts` argument is an object which contains all switch inputs (`short-switch` and `long-switch`). The remaining `args` are an ordered list of the `argument` inputs.

If the function's resolved value possesses a `toHTML()` function, that function will be called during the result rendering and its value will be used in the shell output.
