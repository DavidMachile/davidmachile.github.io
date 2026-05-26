---
layout:     post
title:      setNeedsUpdateConstraints 和 updateConstraintsIfNeeded 先后调用区别
subtitle:   iOS技术成长之路
date:       2024-11-22
author:     DM
header-img: img/post-bg-ioses.jpg
catalog:    true
tags:       AI iOS
---

在 iOS 开发中，setNeedsUpdateConstraints 和 updateConstraintsIfNeeded 是与 Auto Layout 布局系统密切相关的两个方法，它们的作用和调用时机有所不同，因此先后调用它们确实会产生区别。

## setNeedsUpdateConstraints

- 作用：setNeedsUpdateConstraints 是 UIView 类的一个方法，调用它会标记视图（及其子视图）为需要更新其 Auto Layout 约束系统。这意味着在下一个布局更新周期中，系统会调用视图的 updateConstraints 方法来更新或重新计算这些约束。
- 调用时机：当视图的约束需要更新时（例如，由于视图的内容或尺寸发生了变化），应调用此方法。它不会立即触发约束的更新，而是将更新推迟到下一个布局周期。

## updateConstraintsIfNeeded

- 作用：updateConstraintsIfNeeded 方法会立即检查视图是否需要更新约束，如果需要，则立即调用 updateConstraints 方法来更新约束。这与 setNeedsUpdateConstraints 不同，后者只是标记视图为需要更新约束，而不立即执行更新。

- 调用时机：当需要立即更新视图的约束时（例如，在动画块中或在某些需要立即反映约束变化的场景中），应调用此方法。

## 先后调用的区别

### 1.先调用 setNeedsUpdateConstraints 后调用 updateConstraintsIfNeeded：

- 在这种情况下，setNeedsUpdateConstraints 会标记视图为需要更新约束。随后调用 updateConstraintsIfNeeded 会检查这个标记，并立即执行约束的更新（如果确实需要更新的话）。
- 这种调用方式适用于那些需要在未来某个时刻（但不是立即）更新约束，但随后又因为某种原因需要立即更新约束的场景。

### 2.仅调用 updateConstraintsIfNeeded：

- 如果仅调用 updateConstraintsIfNeeded 而没有先调用 setNeedsUpdateConstraints，那么只有当视图的约束实际上已经标记为需要更新时（例如，由于之前已经调用了 setNeedsUpdateConstraints，或者由于视图的某些属性变化自动触发了约束的更新），updateConstraintsIfNeeded 才会执行约束的更新。
  如果视图的约束没有标记为需要更新，那么调用 updateConstraintsIfNeeded 将不会有任何效果。

### 3.先调用 updateConstraintsIfNeeded 后调用 setNeedsUpdateConstraints（通常不推荐）：

- 在这种情况下，updateConstraintsIfNeeded 可能会立即检查约束是否需要更新，但由于 setNeedsUpdateConstraints 尚未被调用，所以除非之前已经有其他操作标记了约束为需要更新，否则 updateConstraintsIfNeeded 不会执行任何更新。
- 随后调用 setNeedsUpdateConstraints 只是标记了视图为需要在下一个布局周期中更新约束，而不会立即触发更新。
- 因此，这种调用方式通常没有实际意义，也不推荐这样做。

<font style="color:red">综上，setNeedsUpdateConstraints 和 updateConstraintsIfNeeded 的先后调用确实会产生区别。在大多数情况下，应先调用 setNeedsUpdateConstraints 来标记视图为需要更新约束，然后在需要立即更新约束时调用 updateConstraintsIfNeeded。</font>
