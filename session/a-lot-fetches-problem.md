# A Lot Fetches Problem

## Fast and maintainable patterns for fetching from a database

From: https://sophiebits.com/2020/01/01/fast-maintainable-db-patterns.html

对于服务端程序需要在一次业务操作中执行大量 fetch 语句这样的场景，我们应该如何处理，使得代码执行速度快且可维护性高？

### Sample Code

```js
async function buildPostsSummary() {
  const posts = await fetchPosts();
  const summary = [];

  posts.forEach(post => {
    const author = await post.fetchAuthor();
    const stats = await post.fetchStats();
    const funFact = stats.viewCount > 0 ? await fetchFunFact(stats.viewCount) : undefined;

    summary.push({
      title: post.title,
      authorName: author.name,
      viewCount: stats.viewCount,
      funFact,
    })
  });

  return summary;
}
```

以上是完成构建 PostsSummary 的最简单实现。存在问题：1. 对于每一篇 post，都要执行三次 fetch 来获取相关数据。2. 按次序地循环获取每一篇 post 的相关数据，直到上一篇获取完毕才会开始处理下一篇。

### Improvement 1: Batching

将一次获取一条记录，改为一次获取多条记录。比如：

```js
async function buildPostsSummary() {
  const posts = await fetchPosts();
  const authors = await fetchManyAuthors(posts);
  const stats = await fetchManyStats(posts);

  // ...
}
```

这样避免了对于每一篇 post，都要去查询三次数据的问题。但同时也带来了新的问题：我们的代码逻辑从一次处理一条记录，变成了一次处理一堆记录，代码结构不可避免地更加复杂了。

```js
  // align data by index => a simple but intuitive version stands for the real world situation
  const nonZeroViewCounts = [];
  const nonZeroViewCountIndices = [];
  stats.forEach((stat, idx) => {
    if (stat.viewCount > 0) {
      nonZeroViewCounts.push(stat.viewCount);
      nonZeroViewCountIndices.push(idx);
    }
  });
  const nonZeroFunFacts = await fetchManyFunFacts(nonZeroViewCounts);
  const funFacts = Array(posts.length).fill(undefined);
  for (const i = 0; i < nonZeroFunFacts.length; i++) {
    const originalIndex = nonZeroViewCountIndices[i];
    funFacts[originalIndex] = nonZeroFunFacts[i];
  }

  // merge data to summary
  const summary = [];
  posts.forEach((post) => {
    summary.push({
      title: posts[i].title,
      authorName: authors[i].name,
      viewCount: stats[i].viewCount,
      funFact: funFacts[i],
    });
  });

  return summary;
}
```

这是非常恼人的写法，显然人们不会在一开始就选择这样去实现，除非性能出现问题。（就我个人的经验是这样）

### Improvement 2: Parallelism

回到之前的例子。我们之前面临的问题之一，是顺序执行了对于每一篇 post 的数据请求。如果改进这一点呢？

```js
async function buildPostsSummary() {
  const posts = await fetchPosts();
  const summary = Promise.all(posts.map(post => buildPostSummary(post)));
  return summary;
}

async function buildPostSummary(post) {
  const author = await post.fetchAuthor();
  const stats = await post.fetchStats();
  const funFact = stats.viewCount > 0 ? await fetchFunFact(stats.viewCount) : undefined;
  return {
    title: post.title,
    authorName: author.name,
    viewCount: stats.viewCount,
    funFact,
  };
}
```

这样一来，代码回到了一次处理一个东西的情况，非常容易理解，同时也减少了请求次数。如果我们有 100 篇 post，则会有 300 + 1 个请求。这些请求大部分会并行执行，所以代码整体的执行速度会比之前的例子快很多。

然而对于每一篇 post 的请求，仍然有大量的固定开销，所以这个方案会比上面的 batching 方案要慢（即使 batching 对我们来说是噩梦一般的选择）。

### Improvement 3: Automatic batching

如果能够维持这样整洁的代码风格，同时能享有 batching 的便利就好了。我们期望的是，虽然以 forEach 的写法去循环执行查询请求，但背后的应用程序框架或者说我们自己实现的基础设施层，能够为我们提供整合多次数据查询至一次 batching 的功能。

只需要 fetchAuthor 在真正去请求数据库之前等待几毫秒就可以了。如果此时发生其他对 fetchAuthor 的调用，我们则可以把这些请求合并在一次 batch 请求里。

更进一步地，我们也可以把强相关联的 stats 和 funFact 请求也合并在一次 batch 请求里。

### Improvement 4: Optimal parallelism

还可以继续优化！比如，并行请求 author 和 stats。

```js
async function buildPostSummary(post) {
  const [
    author,
    stats,
  ] = await Promise.all([
    post.fetchAuthor(),
    post.fetchStats(),
  ]);

  const funFact = stats.viewCount > 0 ? await fetchFunFact(stats.viewCount) : undefined;

  return ...;
}
```

但如果请求 author 的时间要比 stats 长呢？那在请求 funFact 之前就会存在不必要的等待。一种修改方式是将 stats 和与其相关的 funFact 请求合并：

```js
async function buildPostSummary(post) {
  const [
    author,
    [stats, funFact],
  ] = await Promise.all([
    post.fetchAuthor(),
    fetchStatsAndFunFact(),
  ]);
  return ...;

  async function fetchStatsAndFunFact() {
    const stats = await post.fetchStats();
    const funFact =
      stats.viewCount > 0
      ? await fetchFunFact(stats.viewCount)
      : undefined;
    return [stats, funFact];
  }
}
```

这样固然是最优并行，但我们的代码结构跟数据之间的依赖关系强行绑定了。如果需求变成在请求 stats 之前必须先请求 author 怎么办？现实世界中可能存在远超 3 种数据的情况，那样会让重构更难进行。

回到我们希望返回的数据结构 summary，理想情况下，我们希望能够分别考虑各种数据来源，而不是去费脑筋关注这些资源之间的依赖关系。比如：

```js
async function buildPostSummary(post) {
  async function fetchAuthorName() {
    const author = await post.fetchAuthor();
    return author.name;
  }

  async function fetchViewCount() {
    const stats = await post.fetchStats();
    return stats.viewCount;
  }

  async function fetchViewCountFunFact() {
    // note: a repeat request for stats
    const stats = await post.fetchStats();
    return
      stats.viewCount > 0
      ? await fetchFunFact(stats.viewCount)
      : undefined;
  }

  const [
    authorName,
    viewCount,
    funFact,
  ] = await Promise.all([
    fetchAuthorName(),
    fetchViewCount(),
    fetchViewCountFunFact(),
  ]);

  return { title: post.title, authorName, viewCount, funFact };
}
```

这样的实现虽然看起来“笨”了一些，但非常易于理解和扩展。唯一的问题是重复请求。

### Improvement 5: Caching (within a request)

加缓存。大量的。建议是大多数缓存的有效期不应该超过该次网络请求。原因在于：1. 请求结束内存会被释放，如果我们不用担心内存使用率，就可以放心大胆地去做缓存。2. 用户不用担心会拿到过时的结果。3. 将范围限制在一次用户请求里，会降低数据污染的概率。

缓存逻辑同样最好隐藏于应用框架背后。

当然也可以将缓存用于避免重复进行一些复杂计算。

### Conclusion

如何构建基础设施使得应用程序代码能够简单，但同时性能高又可维护？

Facilitating local reasoning.

应该让开发者能够独立地去关注代码的各个部分，而无需将整个系统都装在脑中。这是构建易于扩展的复杂系统的关键。

DataLoader 是一个值得关注的开源库，它内置了上述提到的一些技术。
