# Modifications Done by pug-load

When pug-load is used (`postLoad` plugin hook and after), the AST is extended
as following.

## File Operations

### FileReference

```js
extend FileReference {
  fullPath: string;  // resolved path after being processed by `resolve` function in pug-load
  str: string;       // raw content of file
  ast: Block | null; // if the file is a Pug file, the parsed AST of the file
}
```
