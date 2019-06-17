# logtags: key/value annotations for Go contexts

This package provides a way to attach key/value annotations
to a Go `context.Context`.

This feature is used e.g. in CockroachDB to annotate contexts with
user-facing details from the call stack, for use in logging output.

**How to use:**

- adding k/v data to a context:

  ```go
  func AddTag(ctx context.Context, key string, value interface{}) context.Context
  ```
  
  For example:
  
  ```go
  func foo(ctx context.Context) {
	  ctx = logtags.AddTag(ctx, "foo", 123)
	  bar(ctx)
  }
  ```

- retrieving k/v data from a context:

  ```go
  func FromContext(ctx context.Context) *Buffer
  func (b *Buffer) Get() []Tag
  func (t *Tag) Key() string
  func (t *Tag) Value() interface{}
  ```

**How it works:**

`logtags` stores the provided key/value pairs into an object of type
`Buffer`, then uses Go's standard `context.WithValue` to attach the
buffer.

An instance of `Buffer` inside a context is immutable. When adding a
new k/v pair to a context that already carries a `Buffer`, a new one
is created with all previous k/v pair, and the new k/v pair is added
to that.

The `FromContext()` function retrieves the topmost (most recent)
`Buffer`.

**Advanced uses:**

To add multiple k/v pairs in one go, without using quadratic space in
`Buffer` instances:

1. manually instantiate a `Buffer`.
2. use `func (*Buffer) Add(key string, value interface{})` to populate the buffer.
3. use `func AddTags(ctx context.Context, tags *Buffer) context.Context` to embark
   all the k/v pairs at once.

To format all the contained k/v pairs in a `Buffer`, use its `String()` method:
- when the `value` part is `nil`, `String()` just displays the key.
- when the `value` part is non-nil, `String()` renders it to a string.
