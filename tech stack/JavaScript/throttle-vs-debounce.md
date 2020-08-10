# (Differences between debounce and throttle?)[https://css-tricks.com/debouncing-throttling-explained-examples/]

In summary:

- debounce: Grouping a sudden burst of events (like keystrokes) into a single one.

- throttle: Guaranteeing a constant flow of executions every X milliseconds. Like checking every 200ms your scroll position to trigger a CSS animation.

- requestAnimationFrame: a throttle alternative. When your function recalculates and renders elements on screen and you want to guarantee smooth changes or animations. Note: no IE9 support.

## debounce 防抖

当操作被触发之后等待 delay 时间，若在等待阶段内操作再次被触发则重置等待时间，直到等待时间倒计归零，则实际运行操作触发后的回调。常用于解决：用户输入后需要发送请求/做过滤等有一定消耗，短时间内频繁触发造成卡顿。

debounce 通常也可以通过设置 leading / trailing 选择在 delay 的哪个时刻触发回调，前者代表 delay 开始的当时就触发，后者代表 delay 结束时触发。

### 如何实现？

计时器，再次触发时重置。注意 this 的指向。

[lodash 实现](https://github.com/lodash/lodash/blob/master/debounce.js)

## throttle 节流

限定 delay 时间段内，操作触发后的回调只会执行一次，无论这段时间内操作实际被触发过多少次。

### 如何实现？

计时器/时间戳。注意 this 的指向。

[lodash 实现](https://github.com/lodash/lodash/blob/master/throttle.js)

## requestAnimationFrame

requestAnimationFrame 是另一种限制函数执行速率的办法。\_.throttle(dosomething, 16) 可以认为与其近似等价。但 requestAnimationFrame 保真度更高，毕竟它是浏览器旨在提高绘制准确性而提供的 API。

使用 rAF API 作为 throttle 函数的替代选择，有以下优缺点：

- 优点

  - 浏览器内部机制使得渲染效果更优

  - 标准 API，未来改动可能性很小，几乎没有维护成本。简洁。

- 缺点

  - 需要自己实现 start/cancel

  - 如果浏览器 tab 处于 inactive 状态，则不会被触发。不过对于 scroll/mouse/keyboard 事件这点无关紧要。

  - 部分浏览器还不支持，需要 polyfill。

  - node.js 不支持。
