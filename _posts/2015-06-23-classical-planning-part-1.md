---
layout: post
title:  "Classical Planning (part 1)"
categories: AI, classical planning
---

I'm revisiting "Classical Planning" chapter of the [Artificial Intelligence: A Modern Approach][ai-book]. One outcome I'm targeting is implementing planning algorithm that resolves problems defined by PDDL (Planning Domain Definition Language) described in the book. I'm going to handcraft everything in Ruby, and post source code to https://github.com/xli/longjing.

The first thing that I need sort out is how to represent PDDL in Ruby. Overall, I like to use primary data structure to construct input data, and LISP style syntax can keep interpreter simple.

Hence I'll use:

1. Symbol for all of names, variables, and arguments.
2. Array to construct states/functions.
3. Hash to construct problem and action, as they need multiple parts.

Similarly, the planning output includes 2 parts:

1. Resolved status: Boolean
2. An Array describes the steps (Array of Symbols) reaching the goal.

Examples
=============

states and objects, object type (Block)

{% highlight ruby %}

[:have, :cake]          #=> Have(Cake)
[:eaten, :cake]         #=> Eaten(Cake)

[:on, :A, :table]       #=> On(A, Table)
[:block, :A]            #=> Block(A)

{% endhighlight %}

action, arguments, functions/operators (not, not equal to)

{% highlight ruby %}

{
  name: :move,
  arguments: [:b, :x, :y],
  precond: [[:on, :b, :x],
            [:clear, :b],
            [:clear, :y],
            [:block, :b],
            [:block, :y],
            [:!=, :b, :x],
            [:!=, :b, :y],
            [:!=, :x, :y]],
  effect: [
    [:on, :b, :y],
    [:clear, :x],
    [:-, :on, :b, :x],
    [:-, :clear, :y]
  ]
}

{% endhighlight %}

The Cake example in the book:

{% highlight ruby %}

{
  init: [[:have, :cake]],
  goal: [[:have, :cake], [:eaten, :cake]],
  actions: [
    {
      name: :eat,
      precond: [[:have, :cake]],
      effect: [[:-, :have, :cake], [:eaten, :cake]]
    },
    {
      name: :bake,
      precond: [[:-, :have, :cake]],
      effect: [[:have, :cake]]
    }
  ]
}

{% endhighlight %}

The solution of the Cake example:

{% highlight ruby %}

[[:eat], [:bake]]

{% endhighlight %}


The blocks world example in the book:

{% highlight ruby %}

{
  init: [[:on, :A, :table],
         [:on, :B, :table],
         [:on, :C, :A],
         [:block, :A],
         [:block, :B],
         [:block, :C],
         [:clear, :B],
         [:clear, :C]],
  goal: [[:on, :A, :B], [:on, :B, :C]],
  actions: [
    {
      name: :move,
      arguments: [:b, :x, :y],
      precond: [[:on, :b, :x],
                [:clear, :b],
                [:clear, :y],
                [:block, :b],
                [:block, :y],
                [:!=, :b, :x],
                [:!=, :b, :y],
                [:!=, :x, :y]],
      effect: [
        [:on, :b, :y],
        [:clear, :x],
        [:-, :on, :b, :x],
        [:-, :clear, :y]
      ]
    },
    {
      name: :moveToTable,
      arguments: [:b, :x],
      precond: [[:on, :b, :x],
                [:block, :b],
                [:clear, :b],
                [:!=, :b, :x]],
      effect: [
        [:on, :b, :table],
        [:clear, :x],
        [:-, :on, :b, :x]
      ]
    }
  ]
}

{% endhighlight %}

The solution of blocks world problem:

{% highlight ruby %}

[[:moveToTable, :C, :A], [:move, :B, :table, :C], [:move, :A, :table, :B]]

{% endhighlight %}

Next week I'll start implement interpreter and algorithm that solves Cake example.

[ai-book]:          http://www.amazon.com/Artificial-Intelligence-Modern-Approach-Edition/dp/0136042597