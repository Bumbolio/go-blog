---
title: "What is a Goroutine?"
author: Ernesto Ramos
date: "2023-06-06"
images:
  - "/img/go-repeat.webp"
---

Many of the programs we write when starting our coding journey are single-threaded, one operation executes and then the next one executes in sequence. Single-threaded programming is easy to write and understand, the execution sequence of the program is generally determined by the order routines, such as functions, appear in our code. In order to squeeze out the maximum performance of your machine, in some cases it's beneficial to execute routines _concurrently_, or in other words execute multiple pieces of code at the same time. Modern processors have multiple cores and are capable of executing code on many threads. *Goroutine* makes it possible for us to place function calls on multiple threads and execute them concurrently. Our machine can get a lot more work done in the same amount of time when our program makes better use of the resources available!

## Multithreading using Goroutine
Let's get familiar with Goroutine with a simple example. In the code below we have three simple functions that print text to the console. In our `Main` function, we call `PrintReady`, `PrintSet`, and `PrintGo` in sequence, but we use the keyword `go` when calling the `PrintSet` function.

```go
package main

import (
  "fmt"
  "time"
)

func PrintReady() {
   fmt.Println("Ready...")
}

func PrintSet() {
   fmt.Println("Set...")
}


func PrintGo() {
   fmt.Println("Go!")
}

func main() {
	PrintReady()
  go PrintSet()
  PrintGo()

  time.Sleep(3 * time.Second)
}
```

In a typical single-threaded application, calling these three functions in sequence would print the following to the console:

```text
Ready...
Set...
Go!
```

Because `PrintSet` is being called using the `go` keyword, its execution will happen on a separate thread and our main thread will continue executing the rest of our code concurrently. The end result is that our print statements happen out of sequence since `PrintReady` and `PrintGo` are called on the main thread, while `PrintSet` is called on a separate thread and eventually finishes execution at a later time.

```text
Ready...
Go!
Set...
```

We pause the main thread for 3 seconds using `time.Sleep` to ensure we give the second thread enough time to finish before the first thread ends, otherwise our program may end before the second thread has time to finish executing. Without `time.Sleep` we may never see the result of `PrintSet`.

```text
Ready...
Go!
```

## Messaging and Waiting using Channels

Instead of forcing the main thread to wait for 3 seconds, we could wait until all of our other threads are finished executing. This way we are not wasting time waiting for a thread that may have finished executing a long time ago. We can accomplish this by using _Channels_. Channels allow threads to send messages to each other. A thread is blocked until another thread is ready to read the message. 

```go
package main

import (
  "fmt"
)

func PrintReady() {
   fmt.Println("Ready...")
}

func PrintSet(c chan bool) {
  fmt.Println("Set...")
  c <- true
}


func PrintGo() {
   fmt.Println("Go!")
}


func main() {
  c := make(chan bool)
	PrintReady()
  go PrintSet(c)
  PrintGo()

  <- c
}
```

Our channel newly created channel `c` sends the primitive type `bool`. When we call `PrintSet` we pass in our channel `c` as an argument. The function `PrintSet` then sends its result `true` when it's finished executing. We then receive the result in the main thread using the channel operator `<-`. Since we don't care about the value returned by `PrintSet`, we discard its value by not assigning it to anything via `<- c`. 

We can force the program to print our text in the correct sequence by accepting the result from our channel right after we call `PrintSet`. In this way, we can block our program from continuing to run until we have some desired results.

```go
func main() {
  c := make(chan bool)
	PrintReady()
  go PrintSet(c)
  <- c
  PrintGo()
}
```

In the above scenario, `PrintReady` and `PrintSet` run on two separate threads. `PrintGo` runs on the same thread as `PrintReady`, but it won't execute until we have received a result from `PrintSet` via the channel `c`.

## Next Steps

Now with a basic understanding of Goroutine, you can quickly start writing multithreaded code. What are some ways you could use Goroutine in your code? Leave a comment below!
