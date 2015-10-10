# The Jade AST (WIP)

## Node objects

```js
interface Node {
  type: string;
  line: number | null; // line number of the start position of the node
  filename: string | null; // the name of the file the node originally belongs to
}
```

## Blocks

```js
interface Block <: Node {
  type: "Block";
  nodes: [ Node ];
  yield: boolean | null; // if the Block represents a `yield` operator
}
```

Generic container of Nodes.

FIXME: use a separate type for yields

## Abstract Node Types

```js
interface AttributedNode <: Node {
  attrs: [ Attribute ];                      // all the individual attributes of the node
  attributeBlocks: [ JavaScriptExpression ]; // all the &attributes expressions effective on this node
}

interface BlockNode <: Node {
  block: Block | null;
}

interface ExpressionNode <: Node {
  expr: JavaScriptExpression;
}

interface PlaceholderNode <: Node { }

interface ValueNode <: Node {
  val: string;
}

interface Attribute {
  name: string;               // the name of the attribute
  val: JavaScriptExpression;  // JavaScript expression returning the value of the attribute
  mustEscape: boolean;        // if the value must be HTML-escaped before being buffered
}

interface JavaScriptExpression <: string { }
interface JavaScriptIdentifier <: string { }
```

## Doctypes

```js
interface Doctype <: ValueNode {
  type: "Doctype";
}
```

Doctype declaration.

## Comments

```js
interface CommonComment <: ValueNode {
  buffer: boolean; // whether the comment should appear when rendered
}
```

An abstract type containing all Comments.

```js
interface Comment <: CommonComment {
  type: "Comment";
}
```

Single line comment.

```js
interface BlockComment <: BlockNode, Comment {
  type: "BlockComment";
}
```

A multi-line comment.

## Text

```js
interface Text <: ValueNode {
  type: "Text";
}
```

A piece of plain text.

## Tag

```js
interface Tag <: AttributedNode, BlockNode {
  type: "Tag";
  name: string;               // the name of the tag (or if `buffer` is true, the JavaScriptExpression of the name of the tag)
  selfClosing: boolean;       // if the tag is explicitly stated as self-closing
  buffer: boolean;            // if the tag name should be interpreted at runtime
  isInline: boolean;          // if the tag is defined as an inline tag as opposed to a block-level tag
}
```

An HTML tag.

FIXME: better name for buffer

## Code

```js
interface Code <: BlockNode, ValueNode {
  type: "Code";
  buffer: boolean;            // if the value of the piece of code is buffered in the template
  mustEscape: boolean;        // if the value must be HTML-escaped before being buffered
  isInline: boolean;          // whether the node is the result of a string interpolation
}
```

A piece of code.

## Code Helpers

### Conditional

```js
interface Conditional <: Node {
  type: "Conditional";
  test: JavaScriptExpression;
  consequent: Block;
  alternate: Block | null;
}
```

An if/unless/else expression.

FIXME: use ExpressionNode

### Case/When

```js
interface Case <: BlockNode, ExpressionNode {
  type: "Case";
  block: WhenBlock;
}
```

A case expression.

```js
interface WhenBlock <: Block {
  nodes: [ When ];
}
```

A Block containing When nodes.

```js
interface When <: BlockNode, ExpressionNode {
  type: "When";
  expr: JavaScriptExpression | "default";
}
```

A when/default expression.

### While

```js
interface While <: BlockNode {
  type: "While";
  test: JavaScriptExpression;
}
```

A while loop construct.

FIXME: use ExpressionNode

### Each

```js
interface Each <: BlockNode {
  type: "Each";
  obj: JavaScriptExpression;        // the object or array that is being looped
  val: JavaScriptIdentifier;        // the variable name of the value of a specific object property or array member
  key: JavaScriptIdentifier | null; // the variable name, if any, of the object property name or array index of `val`
  alternative: Block | null;        // the else expression
}
```

A each loop construct.

FIXME: use `alternate` rather than `alternative`

## Mixin

```js
interface Mixin <: AttributedNode, BlockNode {
  type: "Mixin";
  name: JavaScriptIdentifier;       // the name of the mixin
  call: boolean;                    // if this node is a mixin call (as opposed to mixin definition)
}
```

A mixin definition or call.

### MixinBlock

```js
interface MixinBlock <: PlaceholderNode {
  type: "MixinBlock";
}
```

A placeholder for adding a block in mixin.

## File Operations

### FileReference

```js
interface FileReference <: Node {
  type: "FileReference";
  path: string;
}
```

A reference to a file path.

```js
interface FileNode <: Node {
  file: FileReference;
}
```

A node with file reference information.

### Include

```js
interface Include <: BlockNode, FileNode {
  type: "Include";
  filter: string;
  attrs: [ Attribute ];
}
```

An include statement.

### Extends/NamedBlock

```js
interface Extends <: FileNode {
  type: "Extends";
}
```

An extends statement.

```js
interface NamedBlock <: PlaceholderNode {
  type: "NamedBlock";
  name: string;
  mode: "replace" | "append" | "prepend";
  nodes: [ Node ];
}
```

Declaring or providing a named block for inheritance.

## Filter

```js
interface Filter <: BlockNode {
  type: "Filter";
  name: string;
  attrs: [ Attribute ];
}
```

A filter instance.
