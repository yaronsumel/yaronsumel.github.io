### How to get the underlying array of a slice ?

#### underlying what?
as we know, slice it's just `A descriptor of an array segment`. [read about it](https://blog.golang.org/go-slices-usage-and-internals).

#### slice structure (src/runtime/slice.go)

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

#### getting the address of the array

```go
sliceHeader := (*reflect.SliceHeader)(unsafe.Pointer(&x))
log.Printf("%p",unsafe.Pointer(sliceHeader.Data))
```

#### small example to see slice growing in action


```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

// create slice with zero length and capacity of 5
var x = make([]string, 0, 5)

func main() {
	// use unsafe to get ptr of x
	// cast into SliceHeader
	sliceHeader := (*reflect.SliceHeader)(unsafe.Pointer(&x))
	// iterate and make our slice grow bigger
	for t := 0; t < 10; t++ {
		// append something into our slice to grow our slice
		// when slice is growing out of it's cap size
		// underlying data array will be copied to different address with bigger cap size
		x = append(x, "1")
		fmt.Printf("cap %d slice:%p array:%p \r\n ", cap(x), &x, unsafe.Pointer(sliceHeader.Data))
	}
}
```

#### Output

```
 len 1 cap 5   slice:0x50c3f0 array:0xc042058050
 len 2 cap 5   slice:0x50c3f0 array:0xc042058050
 len 3 cap 5   slice:0x50c3f0 array:0xc042058050
 len 4 cap 5   slice:0x50c3f0 array:0xc042058050
 len 5 cap 5   slice:0x50c3f0 array:0xc042058050
 len 6 cap 10  slice:0x50c3f0 array:0xc0420400a0  <-- new underlying array
 len 7 cap 10  slice:0x50c3f0 array:0xc0420400a0
 len 8 cap 10  slice:0x50c3f0 array:0xc0420400a0
 len 9 cap 10  slice:0x50c3f0 array:0xc0420400a0
 len 10 cap 10 slice:0x50c3f0 array:0xc0420400a0
 ```
