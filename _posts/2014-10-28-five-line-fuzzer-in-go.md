---
layout:     post
title:      Five Line Fuzzer in Go
date:       2014-10-28 14:00:00
categories:
  - fuzzing
  - go

---

Yesterday I looked at [Charlie Miller's simple five line fuzzer](http://flatlinesecurity.com/posts/charlie-miller-five-line-fuzzer/) written in Python. Today I wrote that same little fuzzer in Go.

<!-- more -->

### The Five Lines in Python

These are the five lines Miller shows in his slides, written in Python. There's a bit more code around that, but I think that's the basic part he talks about in his slides/presentation.

```bash
numwrites = random.randrange(math.ceil((float(len(buf)) / FuzzFactor))) + 1
for j in range(numwrites):
  rbyte = random.randrange(256)
  rn = random.randrange(len(buf))
  buf[rn] = "%c"%(rbyte)
```

### The Five Lines in Go

And this is what I wrote in go. I should note that I'm a Go newb, so I'm just hacking around for now.

```go
numWrites := len(byteArray) / fuzzFactor + 1
for i := 0; i < numWrites; i++ {
	rn := random(0, len(byteArray))
	rbyte := byte(random(0, 255))
	byteArray[rn] = rbyte
}
```

Below is the full Go file. I'm simply reading a test PDF file into a byte array, altering it randomly, and then writing it to the file system again in a new file called ```changed.pdf```.

A few notes:

* Go doesn't seem to have a built in randrange type function, the code I took I pulled from a blog post, haven't looked into it too much
* ```ioutil.ReadFile``` returns an array of bytes, took me a while to figure that out
* ```rand.Seed``` is a little confusing as to where it should occur

```go
package main

import (
	"fmt"
	"io/ioutil"
	"math/rand"
	"time"
)

func random(min, max int) int {
	return rand.Intn(max-min) + min
}

func main() {

	// http://golangcookbook.blogspot.ca/2012/11/generate-random-number-in-given-range.html
	rand.Seed(time.Now().Unix())

	byteArray, err := ioutil.ReadFile("./tests/pdfs/pdf-sample.pdf")

	if err != nil {
		fmt.Println("error reading file")
		panic(err)
	}

	fuzzFactor := 250
	numWrites := len(byteArray) / fuzzFactor + 1
	fmt.Printf("Writing %d random bytes...\n", numWrites)

	for i := 0; i < numWrites; i++ {
		rn := random(0, len(byteArray))
		rbyte := byte(random(0, 255))
		byteArray[rn] = rbyte
	}

	err = ioutil.WriteFile("./changed.pdf", byteArray, 0644)

	if err != nil {
		fmt.Println("error writing file")
		panic(err)
	} else {
		fmt.Println("Done!")
	}
}
```

### Conclusion

I really like Go, but I need more time with it to get a good understanding. I was happy to see that the five lines of Python could be converted into about the same amount of Go (plus the rand function, though I have a feeling there is a better way for that).
