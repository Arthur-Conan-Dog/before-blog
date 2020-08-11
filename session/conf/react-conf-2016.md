## [A Cartoon Intro to Fiber](https://www.youtube.com/watch?v=ZCuYPiUIONs)

Fiber: new reconciliation algorithm for react.

### Where does reconciliation algorithm come from?

React reconciliation: reconciler + renderer.

reconciler: uses the only one reconciliation algorithm, killer feature of react.

renderer: used to target other host platforms besides the DOM.

### Stuck

user code <=> react <=> main thread

Latter updates could be stuck behind previous large but low priority updates caused by user code(eg. render method).

=> Why? The main thread can only do one thing at a time.

=> Since we could not affect the user side code, react find a way to make main thread more efficient in its work by minimizing and batching the dom changes.

### How previous reconciler works?

#### Elements, Instances and DOM

Elements: { type: Button, props: {...}, chidren: {...} }

Instances: instance created from Element, used to managing what the previous state was and what the next state is and what changes need to make.

DOM nodes: used to tell the browser what it needs to do.

#### Stack reconciler

Construct above things layer by layer: create the element => create instances => DOM nodes / update instances => update DOM nodes, recursively calls mount/update until gets to the bottom of the tree.

Problem: the main thread is getting stuck underneath all of these calls at the bottom of the call stack.

### Fiber reconciler

#### Fiber

```js
{
  stateNode // which instance it's tracking

  // relationships with other nodes in the tree
  child     // child node
  return    // back up to the parent
  sibling   // sibling node
}
```
HostRoot: inject point of React, <div id="react-app"/>

#### Phases

If we want to make the interruptions happen, we need to find a way to avoid the UI inconsistant problem, since the browser may not have the enough info to paint the other nodes that haven't been called update at that moment.

Phase 1: render/reconciliation, interruptible

Phase 2: commit, not interruptible

#### Example: click to square the numbers

handleClick calls setState => react add a list of updates to an update queue => schedule the work that has to happen. As for setState, react considers it as deferred works and schedule it to happen later.(How?)

How to schedule the updates to happen later? => use `requestIdleCallback`

work loop function: gives react the ability to be interruptible. It tracks two things: next unit of work & remaining time.

##### Phase 1

(pic)

1. Clone HostRoot from current working tree => workInProgress tree, including the potential existed child pointer and update queue then check whether the time has expired.

2. Same clone process.

  * `List` has an update queue
  * => copy update queue & call funs, get new state, clear update queue.
  * => put a tag on `List` that represents there needs to have a change made to DOM tree.
  * => has children, keep processing, call render.

3. Check whether children in the current tree can be reused, if not, clone.

At this point, user clicks 'zoom font' button. => a event callback added to the queue that main thread is going to take care of. Since react still has some time, it will keep processing.

4. Update `Item`s and `div`s

  * <Item num={1} />: note that if `shouldComponentUpdate` returns false, the tag won't be added to the fiber node.

  * <Item num={2} />: `shouldComponentUpdate` returns true, gets a tag, clone its children -- `div`. => `div`'s text context changed from 2 => 4, gets a tag, and it gets added to a list on the parent item which tracks of all of the fibers underneath it that have changes. => has no child/sibling, `return`. merge self's list of changes(effect list) with parents'.

  * <Item num={3} />: Time's up! gives control back to the main thread.

5. Main thread does layout, changes font size and comes back to react. Note that nothing in the content changes, react will handle that later.

6. Update last `Item` & `div` => completeUnitOfWork called on the `List` => complete `HostRoot`

  * effect list on `List`: div -> Item -> div -> Item -> List

7. end of the phase 1: gets a pending commit

##### Phase 2

1. keeps going through the effect list on the HostRoot and updating the DOM.

2. switch pointers: current <=> workInProgress

  * double buffering: saves time on memory allocation and garbage collection.

3. go through effect list one more time: rest of the life-cycle hooks(DidMount/Update); update refs; handle error boundries.

4. finish commit

#### Solve trapped behind other larger commits problem

Priorities:(each update gets a priority assigned)

  * Synchronous: same as stack reconciler

  * Task: before next tick

  * Animation: before next frame

  * High: pretty soon

  * Low: minor delays ok, data fetching etc.

  * Offscreen: render in case, prep for display/scroll.

How priority works: when a higher priority task shows up, it gets bumped into the head of the update queue, and react will restart its work from the HostRoot and commit, and starts its work again at the beginning of the lower priority update.

Introduces new problems:

  * some lifecycle hooks might be triggered multiple times
  
  * starvation problem: save works done for low priority tasks that not touched by higher priority tasks.

#### Summary

Using cooperative scheduling, breaking up the works into small unit of work, and this process can be paused. This make updates play together better by allowing more high priority updates to jump in line before the lower priority updates.

Fasten startup speed

Handling work in parallel: add more workers
