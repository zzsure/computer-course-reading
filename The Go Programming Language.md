# 1. Tutorial

## 1.1 Hello, world
- go run helloworld.go
- A program will not compile if there are missing imports or if there are unnecessary ones.
- Go does not require semicolons at the ends of statements or declarations, except where two or more appear on the same line.

## 1.2 Command-Line Arguments
Command-line arguments are avaliable to a program in a variable named Args that is part of the os package; thus its anywhere outside the os package is os.Args.

## 1.3 Finding Duplicate Lines
Printf has over a dozen such conveersions, which Go programmers call verbs. This table is far from a complte specification but illustrates many of the features that are available:
- %d  decimal integer
- %x, %o, %b  integer in hexadecimal, octal, binary
- %f, %g, %e  floating-point number
- %t  boolean: true or false
- %c  rune(Unicode code point)
- %s  string
- %q  quoted string "abc" or rune 'c'
- %v  any value in a natural format
- %T  type of any value
- %%  literal percent sign(no operand)

## 1.4 Animated GIFs
A struct is a group of values called fields, often of different types, that are collected together in a single object that can be treated as a unit.

## 1.5 Fetching a URL
Go provides a collection of packages, grouped under net, that make it easy to send and receive information through the Internet, make low-level network connections, and set up servers, for which Go's concurrency features are particularly useful.

## 1.6 Fetching URLs Concurrently
A goroutine is a concurrent function execution. A channel is a communication mechanism that allows one goroutine to pass values of a specified type to another goroutine. The function main runs in a goroutine and the go statement creates additional goroutines.

## 1.7 A Web Server
