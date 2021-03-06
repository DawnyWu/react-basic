# React - Basic Theoretical Concepts
# React - 基本理论概念

This document is my attempt to formally explain my mental model of React. The intention is to describe this in terms of deductive reasoning that lead us to this design.

在这篇文章，是我描述头脑中react模型的一次尝试。目的是通过推导的方式描述这个模型，从而引领我们了解它的设计。

There may certainly be some premises that are debatable and the actual design of this example may have bugs and gaps. This is just the beginning of formalizing it. Feel free to send a pull request if you have a better idea of how to formalize it. The progression from simple -> complex should make sense along the way without too much library details shining through.

这其中会有很多前提条件是有争议的，例子的设计也可能有bug或缺陷。这只是我们形式化描述他的开始。
如果你有更好的描述它的办法请给我们提交pull request。过程会从简单->复杂，其中不会展示太多实现的细节。

The actual implementation of React.js is full of pragmatic solutions, incremental steps, algorithmic optimizations, legacy code, debug tooling and things you need to make it actually useful. Those things are more fleeting, can change over time if it is valuable enough and have high enough priority. The actual implementation is much more difficult to reason about.

React.js的实现中充满了实用的解决办法，渐进的步骤，算法的优化，老旧代码的处理，debug工具和其他的有用的东西。那些东西是飞速变化的。具体实现更加的难以推导的。

I like to have a simpler mental model that I can ground myself in.

我希望能够有一个简单的思维模型来

## Transformation
## 转化

The core premise for React is that UIs are simply a projection of data into a different form of data. The same input gives the same output. A simple pure function.

UI是数据在不同状况下的映射而已，这是React重要的前提。相同的输入要有相同的输出。组件要是一个"纯函数"（pure function）。

```js
function NameBox(name) {
  return { fontWeight: 'bold', labelContent: name };
}
```

```
'Sebastian Markbåge' ->
{ fontWeight: 'bold', labelContent: 'Sebastian Markbåge' };
```

## Abstraction
## 抽象

You can't fit a complex UI in a single function though. It is important that UIs can be abstracted into reusable pieces that doesn't leak their implementation details. Such as calling one function from another.

你不应当把一个复杂的UI放到一个函数里。 UI一定要抽象成为可以复用的组件，并且应当封装好，不暴露实现细节,这是十分重要的。比如在一个函数中调用另一个函数。

```js
function FancyUserBox(user) {
  return {
    borderStyle: '1px solid blue',
    childContent: [
      'Name: ',
      NameBox(user.firstName + ' ' + user.lastName)
    ]
  };
}
```

```
{ firstName: 'Sebastian', lastName: 'Markbåge' } ->
{
  borderStyle: '1px solid blue',
  childContent: [
    'Name: ',
    { fontWeight: 'bold', labelContent: 'Sebastian Markbåge' }
  ]
};
```

## Composition
## 组合

To achieve truly reusable features, it is not enough to simply reuse leaves and build new containers for them. You also need to be able to build abstractions from the containers that *compose* other abstractions. The way I think about "composition" is that they're combining two or more different abstractions into a new one.

为了达到真正的可复用，仅仅复用小的组件，然后在它们之上建立container是不够的。你要能够在多个container之上建立抽象，其中的container又可以*组合*其他的抽象。
我觉得“组合”意思是将两个或多个抽象合并为一个抽象。

```js
function FancyBox(children) {
  return {
    borderStyle: '1px solid blue',
    children: children
  };
}

function UserBox(user) {
  return FancyBox([
    'Name: ',
    NameBox(user.firstName + ' ' + user.lastName)
  ]);
}
```

## State
## 状态

A UI is NOT simply a replication of server / business logic state. There is actually a lot of state that is specific to a specific projection and not others. For example, if you start typing in a text field. That may or may not be replicated to other tabs or to your mobile device. Scroll position is a typical example that you almost never want to replicate across multiple projections.

UI不是 服务器/业务逻辑 状态的简单复制。有很多的状态仅仅对应一个映射。举个例子，如果你在文本框打字，可能你不大想要把它复制到其他的tab或者你的手机上。Scroll position也是一个典型的例子，你不大想要把它复制到其他映射中。

We tend to prefer our data model to be immutable. We thread functions through that can update state as a single atom at the top.
我们倾向于我们的数据模型是不可变的。

```js
function FancyNameBox(user, likes, onClick) {
  return FancyBox([
    'Name: ', NameBox(user.firstName + ' ' + user.lastName),
    'Likes: ', LikeBox(likes),
    LikeButton(onClick)
  ]);
}

// Implementation Details

var likes = 0;
function addOneMoreLike() {
  likes++;
  rerender();
}

// Init

FancyNameBox(
  { firstName: 'Sebastian', lastName: 'Markbåge' },
  likes,
  addOneMoreLike
);
```

*NOTE: These examples use side-effects to update state. My actual mental model of this is that they return the next version of state during an "update" pass. It was simpler to explain without that but we'll want to change these examples in the future.*

*注意：这个例子更新状态时有副作用。实际上我脑海中的模型应当是经过“更新”返回下一个版本的状态。现在这样只是为了方便解释，不过以后我们会修改这个例子。

## Memoization
## 记忆


Calling the same function over and over again is wasteful if we know that the function is pure. We can create a memoized version of a function that keeps track of the last argument and last result. That way we don't have to reexecute it if we keep using the same value.

一次又一次的调用纯函数是一种浪费。我们可以参照函数最后一次运行的参数和结果，生成一个有记忆能力（memoized version）的函数。这样我们就可以使用记忆的值而不用重复执行了函数了。

```js
function memoize(fn) {
  var cachedArg;
  var cachedResult;
  return function(arg) {
    if (cachedArg === arg) {
      return cachedResult;
    }
    cachedArg = arg;
    cachedResult = fn(arg);
    return cachedResult;
  };
}

var MemoizedNameBox = memoize(NameBox);

function NameAndAgeBox(user, currentTime) {
  return FancyBox([
    'Name: ',
    MemoizedNameBox(user.firstName + ' ' + user.lastName),
    'Age in milliseconds: ',
    currentTime - user.dateOfBirth
  ]);
}
```

## Lists

Most UIs are some form of lists that then produce multiple different values for each item in the list. This creates a natural hierarchy.
大部分的UI都是某种形式的list, list里的各个元素会生成不同的值。这就会产生层次结构。

To manage the state for each item in a list we can create a Map that holds the state for a particular item.

为了管理list中不同元素的状态，我们可以生成一个Map来保存各个元素的状态

```js
function UserList(users, likesPerUser, updateUserLikes) {
  return users.map(user => FancyNameBox(
    user,
    likesPerUser.get(user.id),
    () => updateUserLikes(user.id, clicksPerUser.get(user.id) + 1)
  ));
}

var likesPerUser = new Map();
function updateUserLikes(id, likeCount) {
  likesPerUser.set(id, likeCount);
  rerender();
}

UserList(data.users, likesPerUser, updateUserLikes);
```

*NOTE: We now have multiple different arguments passed to FancyNameBox. That breaks our memoization because we can only remember one value at a time. More on that below.*

## Continuations

Unfortunately, since there are so many list of lists all over the place in UIs, it becomes quite a lot of boilerplate to manage that explicitly.

不幸的是，由于有许多list遍布在UI各个地方，我们要用许多模板代码显式的管理他们。

We can move some of this boilerplate out of our critical business logic by deferring execution of a function. For example, by using "currying" (`bind` in JavaScript). Then we pass the state through from outside our core functions that are now free of boilerplate.

我们可以通过延迟函数的执行来将一些模板代码从业务逻辑中移除出去。比如，用"currying" (`bind` in JavaScript)。之后我们将状态从核心函数外部传入，这样我们就不需要模板代码了。



This isn't reducing boilerplate but is at least moving it out of the critical business logic.

这个例子并没有减少模板代码，但是至少我们将它移出了核心业务逻辑。

```js
function FancyUserList(users) {
  return FancyBox(
    UserList.bind(null, users)
  );
}

const box = FancyUserList(data.users);
const resolvedChildren = box.children(likesPerUser, updateUserLikes);
const resolvedBox = {
  ...box,
  children: resolvedChildren
};
```

## State Map

We know from earlier that once we see repeated patterns we can use composition to avoid reimplementing the same pattern over and over again. We can move the logic of extracting and passing state to a low-level function that we reuse a lot.

从前面的内容我们了解到，当我们看到重复的pattern时，我们可以用组合的方法避免一次又一次的重复实现同一种pattern。

```js
function FancyBoxWithState(
  children,
  stateMap,
  updateState
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState
    ))
  );
}

function UserList(users) {
  return users.map(user => {
    continuation: FancyNameBox.bind(null, user),
    key: user.id
  });
}

function FancyUserList(users) {
  return FancyBoxWithState.bind(null,
    UserList(users)
  );
}

const continuation = FancyUserList(data.users);
continuation(likesPerUser, updateUserLikes);
```

## Memoization Map

Once we want to memoize multiple items in a list memoization becomes much harder. You have to figure out some complex caching algorithm that balances memory usage with frequency.

一旦我们想要记忆一个list中的多个元素， 记忆就会变得十分困难。你不得不要想出一些复杂的缓存算法来平衡内存的使用率

Luckily, UIs tend to be fairly stable in the same position. The same position in the tree gets the same value every time. This tree turns out to be a really useful strategy for memoization.

幸运的是， UI在一些位置相对稳定。每次树上的同一个位置总是得到同一个值。这个树变成了记忆有用的策略。

We can use the same trick we used for state and pass a memoization cache through the composable function.

我们可以用相同的技巧，通过组合函数传递记忆缓存

```js
function memoize(fn) {
  return function(arg, memoizationCache) {
    if (memoizationCache.arg === arg) {
      return memoizationCache.result;
    }
    const result = fn(arg);
    memoizationCache.arg = arg;
    memoizationCache.result = result;
    return result;
  };
}

function FancyBoxWithState(
  children,
  stateMap,
  updateState,
  memoizationCache
) {
  return FancyBox(
    children.map(child => child.continuation(
      stateMap.get(child.key),
      updateState,
      memoizationCache.get(child.key)
    ))
  );
}

const MemoizedFancyNameBox = memoize(FancyNameBox);
```

## Algebraic Effects

It turns out that it is kind of a PITA to pass every little value you might need through several levels of abstractions. It is kind of nice to sometimes have a shortcut to pass things between two abstractions without involving the intermediates. In React we call this "context".

有的时候通过几个抽象层传递数据感觉有点蛋疼。这时候我们需要有一种方法能够让我们越过中间过程，在两个组件之间传递数据。这种方法在React中叫做“context” 

Sometimes the data dependencies doesn't neatly follow the abstraction tree. For example, in layout algorithms you need to know something about the size of your children before you can completely fulfill their position.

有时候数据的依赖并不是完全的按照抽象树的样子。比如说，在布局算法中你在计算子组件的位置之前要先知道子组件的大小。

Now, this example is a bit "out there". I'll use [Algebraic Effects](http://math.andrej.com/eff/) as [proposed for ECMAScript](https://esdiscuss.org/topic/one-shot-delimited-continuations-with-effect-handlers). If you're familiar with functional programming, they're avoiding the intermediate ceremony imposed by monads.

```js
function ThemeBorderColorRequest() { }

function FancyBox(children) {
  const color = raise new ThemeBorderColorRequest();
  return {
    borderWidth: '1px',
    borderColor: color,
    children: children
  };
}

function BlueTheme(children) {
  return try {
    children();
  } catch effect ThemeBorderColorRequest -> [, continuation] {
    return continuation('blue');
  }
}

function App(data) {
  return BlueTheme(
    FancyUserList.bind(null, data.users)
  );
}
```

