# The Jade Token Stream (WIP)

This document describes the types of tokens emitted by jade-lexer.

## Base Objects

This section describes notations that will be used throughout the document. These are not actual token types, but define the properties of a token.

```js
interface Token {
  type: string;
  line: number; // line number of the start position of the token
}
```

A token.

```js
interface ValueToken <: Token {
  val: string; // the value of the specific token
}
```

A token with a corresponding value.

```js
interface SimpleToken <: Token { }
```

A token that does not contain any information beyond its existence.

```js
interface JavaScriptExpression <: string { }
interface JavaScriptIdentifier <: string { }
```

JavaScript expressions and identifiers.

```js
interface EscapableToken <: Token {
  mustEscape: boolean; // if the value must be HTML-escaped before being buffered
}
```

A token that need or need not be escaped.

## End of Stream

```js
interface EOS <: SimpleToken {
  type: "eos";
}
```

Denotes the end of input.

```jade
```

```js
{ type: 'eos', line: 1 }
```

## New line

```js
interface Newline <: SimpleToken {
  type: "newline";
}
```

Denotes a new line.

```jade

```

```js
{ type: 'newline', line: 2 }
{ type: 'eos',     line: 2 }
```

## Indentation

```js
interface Indent <: SimpleToken {
  type: "indent";
  val: number;    // number of indented characters from the start of the line
}
```

An increase of indentation. One such token is emitted for each level of indentation.

If one such token is emitted, no separate newline token is issued.

```js
interface Outdent <: SimpleToken {
  type: "outdent";
}
```

A decrease of indentation. One such token is emitted for each level of indentation being restored.

If one such token is emitted, no separate newline token is issued.

```jade
| abc
 | def
    | ghi
```

```js
{ type: 'text',    line: 1, val: 'abc' }
{ type: 'indent',  line: 2, val: 1 }
{ type: 'text',    line: 2, val: 'def' }
{ type: 'indent',  line: 3, val: 4 }
{ type: 'text',    line: 3, val: 'ghi' }
{ type: 'outdent', line: 3 }
{ type: 'outdent', line: 3 }
{ type: 'eos',     line: 3 }
```

## Text

```js
interface Text <: ValueToken {
  type: "text";
}
```

A piece of plain text.

```jade
| abc
p abc
```

```js
{ type: 'text',    line: 1, val: 'abc' }
{ type: 'newline', line: 2 }
{ type: 'tag',     line: 2, val: 'p' }
{ type: 'text',    line: 2, val: 'abc' }
{ type: 'eos',     line: 2 } 
```

### HTML Text

```js
interface TextHTML <: Text {
  type: "text-html";
}
```

A piece of auto-detected HTML text.

```jade
<ul></ul>
```

```js
{ type: 'text-html', line: 1, val: '<ul></ul>' }
{ type: 'eos',       line: 2 } 
```

## Code

```js
interface Code <: ValueToken, EscapableToken {
  type: "code";
  buffer: boolean; // whether the code should appear when rendered
}
```

A piece of buffered or unbuffered code.

```jade
- var a = 0
= a
```

```js
{ type: 'code',    line: 1, val: 'var a = 0', mustEscape: false, buffer: false }
{ type: 'newline', line: 2 }
{ type: 'code',    line: 2, val: 'a',         mustEscape: true,  buffer: true }
{ type: 'eos',     line: 2 }
```

### Interpolated Code

```js
interace InterpolatedCode <: Code {
  type: "interpolated-code";
  val: JavaScriptExpression;
}
```

A piece of code that is interpolated within a body of text.

```jade
p #{a}
```

```js
{ type: 'tag',               line: 1, val: 'p' }
{ type: 'interpolated-code', line: 1, val: 'a', mustEscape: true, buffer: true }
{ type: 'eos',               line: 1 }
```

## Tag

```js
interface Tag <: ValueToken {
  type: "tag";
}
```

A tag.

```jade
p
```

```js
{ type: 'tag', line: 1, val: 'p' }
{ type: 'eos', line: 1 }
```

### Interpolated Tag

```js
interface Interpolation <: Tag {
  type: "interpolation";
  val: JavaScriptExpression;
}
```

A tag whose name is interpolated at runtime.

```jade
#{myVar}
```

```js
{ type: 'interpolation', line: 1, val: 'myVar' }
{ type: 'eos',           line: 1 }
```

## Attributes

```js
interface StartAttributes <: SimpleToken {
  type: "start-attributes";
}
interface EndAttributes <: SimpleToken {
  type: "end-attributes";
}
```

Denote the beginning and ending of a series of attributes. Between these two tokens, only Attributes are allowed.

```js
interface Attribute <: ValueToken, EscapableToken {
  type: "attribute";
  name: string;                         // the name of the attribute
  val: JavaScriptExpression | boolean;  // JavaScript expression returning the value of the attribute
}
```

An individual attribute.

```jade
a(href="https://google.com/" contentEditable class!=alreadyEscapedClass)
```

```js
{ type: 'tag',              line: 1, val: 'a' }
{ type: 'start-attributes', line: 1 }
{ type: 'attribute',        line: 1, name: 'href',            val: '"https://google.com/"', mustEscape: true }
{ type: 'attribute',        line: 1, name: 'contentEditable', val: true,                    mustEscape: false }
{ type: 'attribute',        line: 1, name: 'class',           val: 'alreadyEscapedClass',   mustEscape: false }
{ type: 'end-attributes',   line: 1 }
{ type: 'eos',              line: 1 }
```
