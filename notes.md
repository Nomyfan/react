# tag vs. type vs. $$typeof vs. elementType

Fiber.tag(aka WorkTag)
Fiber.type function/class/HTML tag

Fiber.elementType
// TODO

ReactElement.$$typeof 用来抵御XSS攻击。
enum definitions in packages/shared/ReactSymbols.js

ReactElement.type function/class/HTML tag

# Sync update
```text
scheduleUpdateOnFiber
  -> performSyncWorkOnRoot(packages/react-reconciler/src/ReactFiberWorkLoop.old.js)
    -> renderRootSync
      -> workLoopSync
        -> performUnitOfWork
            -> beginWork
                ...-> reconcileChildren(ChildReconciler)
            ->? completeUnitOfWork
    -> commitRoot(commitRootImpl)
      -> flushPassiveEffects
```

# pushDispatcher
hooks是通过dispatcher来引用的，用来检查是否存在不合法的hook使用。
在执行renderRootSync的中间，不允许有hook调用。

# pushInteractions
跟scheduler的tracing相关，可以先不用管。

# workLoopSync
```javascript
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork(workInProgress);
  }
}
```

# beginWork
执行组件渲染。

## renderWithHooks
函数组件才会调用。会设置HooksDispatcher

# React的diff算法
> reuse的基本逻辑①：先判断key，然后判断type，相同reuse。

非ArrayLike的children，先试图从oldFiber链表中找到一个能满足①的fiber，否则新建。

ArrayLike的children，对相同slot的fiber和child采用①，否则，把剩下的oldFiber从链表转成Map<Fiber.key | Fiber.index, Fiber>，
然后逐一对child试图从Map中找到能满足①的fiber，没有则新建。

# React的fiber树是如何组织的？
fiber.child，子树第一个节点。fiber.sibling，兄弟节点。fiber.return父节点。
fiber(A)的children的return都指向fiber(A)。假设有一下这样一个JSX结构：
```jsx
function Root() {
  return (
    <>
      <C1>
        <C11/>
      </C1>
      <C2/>
    </>
  );
}
```
那么它的fiber树结构如下
```text
Root.child -> C1
Root.return -> null
Root.sibling -> null

C1.child -> C11
C1.return -> Root
C1.sibling -> C2

C11.child -> null
C11.return -> C1
C11.sibling -> null

C2.child -> null
C2.return -> Root
C2.sibling -> null
```

# React如何对整棵树进行reconcile？
```javascript
function workLoopSync() {
  while (workInProgress !== null) {
    performUnitOfWork();
  }
}
```
模拟了栈的执行，深度优先遍历。

在performUnitOfWork里调用beginWork，返回的child设置到模块变量workInProgress里。当返回的child为null时，也就是当前fiber是叶子节点，调用`completeUnitOfWork`。不出意外，`completeUnitOfWork`里调用的`completeWork`返回null，
然后检查sibling，如果不是null，设置workInProgress为它，否则设置completeWork为父节点，直到completedWork为null，此时workInProgress也为null。

## Reconcile的时候子组件导致父组件状态更新，如何添加新调度？

## 每次都必须从Root开始吗？

# React是如何拿着fiber去渲染到DOM上的？

commit阶段，在`commitMutationEffects`，根据effect的flags来判断是placement，update，deletion来操作DOM。这些操作递归调用。

# Fiber的stateNode保存了DOM的引用，那么它是何时、如何被创建的？

调用链
```text
performUnitOfWork -> completeUnitOfWork -> completeWork -> createInstance(react-dom/ReactDOMHostConfig) -> createElement(react-dom/ReactDOMComponent)
                                                        -> updateHostComponent -> prepareUpdate(react-dom/ReactDOMHostConfig)返回updatePayload设置为fiber的updateQueue
```
不论是create还是update，都markUpdate（fiber.flags设置Update标记位）。

# Fiber.flags

# What's BlocksAPI?

# What's passive effects?

# 有sideEffect的fiber是如何记录的？

# commitMutationEffects
```text
commitMutationEffects
  -> commitPlacement
  -> commitWork
  -> commitDeletion
```
