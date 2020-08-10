# [Concurrency is not Parallelism](https://www.youtube.com/watch?v=cN_DpYBzKso)

[Slides](https://talks.golang.org/2012/waza.slide#17)

> In computer science, concurrency is the ability of different parts or units of a program, algorithm, or problem to be executed out-of-order or in partial order, without affecting the final outcome. -- [Concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science))

> Parallel computing is a type of computation in which many calculations or the execution of processes are carried out simultaneously. -- [Parallel Computing](https://en.wikipedia.org/wiki/Parallel_computing)

Concurrency 是一种思想，一种 break problem 的模式，一种 Model，而 Parallelism 是一种处理具体 Task 的方法。

> Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.

两者难以区分是因为在某些问题域的情况下的描述可能是重合的。

Rob 以 Gophers 运书为例：

books => incinerator

1. one gopher 运书

2. two gophers + one pile of books + one incinerator

=> 会快一些，但为 books 和 incinerator 引入了 I/O 瓶颈，gopher 之间也需要交流。

3. two gophers + two piles of books + two incinerators

=> 让两边独立。这是一种 concurrent composition of two procedures，但并不一定是 parallel。比如，同一时间只能有一只 gopher 工作，从 design 上讲这个模型仍然是 concurrency 的。

4.1 Another design: three gophers + one pile of books + one incinerator

=> 一只装书 一只运书 一只烧书

=> Furthermore: add a one gopher to return empty cart 一只归还空的小推车

> We improved performance by adding a concurrent procedure to the existing design. More gophers doing more work; it runs better. This is a deeper insight than mere parallelism.

4.2 Another design: two gophers + one pile of books + one staging pile in the middle + one incinerator

一只运到中间 另一只从中间运到炉子

5. 组合上述的所有方法

学到了什么：

> There are many ways to break the processing down. That's concurrent design.

> Once we have the breakdown, parallelization can fall out and correctness is easy.

映射到现实中，可以理解为 book pile => web content, gopher => CPU, cart => rendering / networking, incinerator => consumer like proxy, browser，这个例子就变成了一个 concurrent design for a scalable web service。
