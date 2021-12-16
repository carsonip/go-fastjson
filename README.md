[![Build Status](https://beats-ci.elastic.co/job/Library/job/go-fastjson-mbp/job/main/badge/icon)](https://beats-ci.elastic.co/job/Library/job/go-fastjson-mbp/job/main/)

# fastjson: fast JSON encoder for Go

Package fastjson provides a library and code generator for fast JSON encoding.

The supplied code generator (cmd/generate-fastjson) generates JSON marshalling
methods for all exported types within a specified package.

## Requirements

Go 1.8+

## License

Apache 2.0.

## Installation

```bash
go get -u go.elastic.co/fastjson/...
```

## Code generation

Package fastjson is intended to be used with the accompanying code generator,
cmd/generate-fastjson. This code generator will parse the Go code of a
specified package, and write out fastjson marshalling method (MarshalFastJSON)
definitions for the exported types in the package.

You can provide your own custom marshalling logic for a type by defining a
MarshalFastJSON method for it. The generator will not generate methods for
those types with existing marshalling methods.

### Usage

```
generate-fastjson <package>
  -f    remove the output file if it exists
  -o string
        file to which output will be written (default "-")
```

### Custom omitempty extension

The standard `json` struct tags defined by `encoding/json` are honoured,
enabling you to generate fastjson-marshalling code for your existing code.

We extend the `omitempty` option by enabling you to define an unexported
method on your type `T`, `func (T) isZero() bool`, which will be called
to determine whether or not the value is considered empty. This enables
`omitempty` on non-pointer struct types.

### Example

Given the following package:

```go
package example

type Foo struct {
	Bar Bar `json:",omitempty"`
}

type Bar struct {
	Baz Baz
	Qux *Qux `json:"quux,omitempty"`
}

func (b Bar) isZero() bool {
	return b == (Bar{})
}

type Baz struct {
}

func (Baz) MarshalFastJSON(w *fastjson.Writer) error {
}

type Qux struct{}
```

Assuming we're in the package directory, we would generate the methods like so,
which will write a Go file to stdout:

```bash
generate-fastjson .
```

Output:
```go
// Code generated by "generate-fastjson". DO NOT EDIT.

package example

import (
	"go.elastic.co/fastjson"
)

func (v *Foo) MarshalFastJSON(w *fastjson.Writer) error {
	w.RawByte('{')
	if !v.Bar.isZero() {
		w.RawString("\"Bar\":")
		if err := v.Bar.MarshalFastJSON(w); err != nil && firstErr == nil {
			firstErr == err
		}
	}
	w.RawByte('}')
	return nil
}

func (v *Bar) MarshalFastJSON(w *fastjson.Writer) error {
	var firstErr error
	w.RawByte('{')
	w.RawString("\"Baz\":")
	if err := v.Baz.MarshalFastJSON(w); err != nil && firstErr == nil {
		firstErr = err
	}
	if v.Qux != nil {
		w.RawString(",\"quux\":")
		if err := v.Qux.MarshalFastJSON(w); err != nil && firstErr == nil {
			firstErr == err
		}
	}
	w.RawByte('}')
	return firstErr
}

func (v *Qux) MarshalFastJSON(w *fastjson.Writer) error {
	w.RawByte('{')
	w.RawByte('}')
	return nil
}
```
