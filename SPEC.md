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
}
```

Generic container of Nodes.

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
interface CommonTag <: AttributedNode, BlockNode {
  selfClosing: boolean;       // if the tag is explicitly stated as self-closing
  isInline: boolean;          // if the tag is defined as an inline tag as opposed to a block-level tag
}
```

### Regular Tag

```js
interface Tag <: CommonTag {
  type: "Tag";
  name: string;               // the name of the tag
}
```

An HTML tag.

### Interpolated Tag

```js
interface InterpolatedTag <: CommonTag, ExpressionNode {
  type: "InterpolatedTag";
}
```

A tag whose name is interpolated at runtime.

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

### Each

```js
interface Each <: BlockNode {
  type: "Each";
  obj: JavaScriptExpression;        // the object or array that is being looped
  val: JavaScriptIdentifier;        // the variable name of the value of a specific object property or array member
  key: JavaScriptIdentifier | null; // the variable name, if any, of the object property name or array index of `val`
  alternate: Block | null;          // the else expression
}
```

A each loop construct.

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

## Yield

```js
interface YieldBlock <: PlaceholderNode {
  type: "YieldBlock";
}
```

A placeholder for `yield`s in included file.

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

#### Jade Include

```js
interface Include <: BlockNode, FileNode {
  type: "Include";
}
```

#### Raw Include

```js
interface RawInclude <: FileNode {
  type: "RawInclude";
  filters: [ IncludeFilter ];
}
```

A raw inclusion with optional filter(s) applied. `.filters` can contain zero
or more `IncludeFilter`s, applied in ascending order from `[0]`.

```js
interface IncludeFilter <: FilterNode {
  type: "IncludeFilter";
}
```

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
interface FilterNode <: Node {
  name: string;
  attrs: [ Attribute ];
}
```

A generic node denoting a filter.

```js
interface Filter <: FilterNode, BlockNode {
  type: "Filter";
}
```

A filter instance.
