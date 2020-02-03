---
title: Using Golang TimeTicker for a set duration
---

Problem: As part of me learning and experimenting with golang and working on a small [gauge-reportserver](https://github.com/sitture/gauge-reportserver) plugin, I stumbled across a problem where I had to repeat a task for a certain period.

<!--more-->

Solution:

1. Create a `time.Ticker` so that it can hold a channel that ticks at set intervals.
2. Use a `time.Timer` to stop the ticker after a set period of time.

[Timers](https://gobyexample.com/timers) and [Tickers](https://gobyexample.com/tickers) let you execute code in the future, once or repeatedly.

### Ticker

The following will create a new ticker and `tick tick` with a `500 milliseconds` interval.

```golang
ticker := time.NewTicker(500 * time.Millisecond)
defer func() { ticker.Stop() }()
for {
    select {
    case <-ticker.C:
        fmt.Printf("tick tick")
}
```

The above will go on forever unless stopped, but we needed to `timeout` the ticker after a set duration which can easily be achieved by `time.After` that waits for a specified duration and then sends the current time on the returned channel.

### Timer

We can add `time.After(10 * time.Second)` to the `select` to end the ticker after 10 seconds.

```golang
ticker := time.NewTicker(500 * time.Millisecond)
defer func() { ticker.Stop() }()
for {
    select {
    case <-ticker.C:
        fmt.Printf("tick tick")
    case <-time.After(10 * time.Second):
        fmt.Println("10 seconds! End Ticker")
        return
    }
}
```

However, running the above would still tick forever because a `select` blocks until one of it's cases is ready and then it executes the case but `time.After(10 * time.Second)` was never getting a chance to be ready due to interval being shorter, that would only work if that was less than 5 seconds.

This can be resolved by by having `timer` declared before starting the `for` loop.

The final solution looks like this where a `ticker` is created that ticks every `500 milliseconds` and `timer` added to stop the ticker after `10 seconds` have elapsed.

```golang
ticker := time.NewTicker(500 * time.Millisecond)
defer func() { ticker.Stop() }()
timer := time.After(10 * time.Second)
for {
    select {
    case <-ticker.C:
        fmt.Printf("tick tick")
    case <-timer:
        fmt.Println("10 seconds! End Ticker")
        return
    }
}
```

Hopefully, the above is helpful to others. Good Luck! :)
