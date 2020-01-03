---
layout: post
title:  "A note on Angular pipes"
date:   2019-12-05 21:16:17 +0100
categories: jekyll update
---

In my opinion, pipes are being underrated in the Angular world: in the documentation, the community litterature or,
(on a smaller scale), in the projects I have worked on so far. Therefore I wanted to do something to contribute in
pipe’s popularity and interest through this article.
<br/>
I will expose in this articles the reasons why I like pipes :
 - Better Soc (Separation of Concerns) => maintainability, clarity, testability..
 - Better optimization (parag 2)
 - Foster functional programming (because are by default pure) (parag 3)

### What is a pipe ?
> As the name may infer (Linux pipes ?), a pipe is primarily a Decorator on a class which is used on template side. The idea is to output a data (linux pipes? Node pipes?).
Often used for formatting. Most useful angular pipes are implemented in the Angular core package (and these are often enough), but we can also easily create our own pipes:

Ex: Angular pipe qqcpnque
template + code

To declare a pipe, the process is the followg :
Add the required meta-data
Implement the `PipeTransform`interface
Declare in ngModule, `declarations` property
This can be done automatically by executing the CLI command: `ng g p <pipeName>`

Appart from the useful syntax, in terms of code design, I find them also useful to gather ‘helper’ / ‘formatting’ code in an appropriate place => better SoC.

They are also more optimized, in terms of application efficiency, then ‘by hands’ solutions, as we will see in the next paragraph.

### Pipes: a functionnal solution for a better SoC:

To illustrate this point, let's start with an example. Here is the usecase: we need to format a string to concat an input (a name) to 'Hello '.
// to match an id with a name (let's assume we have mapping available).

// add a flag according to provenance to each plate

// or a custom emoji according to the rating of products

Object oriented = attach the emoji code in the "meal" object
functional = apply a separated traetment to get the flag

(1st level of functional / OOP)

2 Options are available:

First, we can create a method in our component and call it from our template.


{% highlight javascript %}
    // component.ts
    optionsIds = ['burger', 'sushi', 'pizza'];
    optionsNames = ['burger', 'sushi', 'pizza'];
    convertIdToName(id) {
        return options[id];
    }

{% endhighlight %}

```
<ul>
    <li *ngFor="let id of optionsIds">{{convertIdToName(id)}}</li>
</ul>
```

// disavdvantage : of ngSwitch => what if several times ? Space + non changeable

**Nb: I know what you might be thinking, 'unusefully complicated' => might have been simpler to directly use the `optionsName` in the template
or, if the id had been needed, to merge the 2
