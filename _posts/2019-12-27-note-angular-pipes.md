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

Let's illustrate first this with an example.

### The usecase / A usecase example

A good usecase for pipes is a situation where:
- some data needs to be formatted to be displayed
- this formatting is only needed on the template

We will work on the following example: an app where we display a list of meals. The list looks like this:

```
meals = [
    {name: 'burger', satisfaction: 92},
    {name: 'sushi', satisfaction: 95},
    {name: 'veggie salad', satisfaction: 12}
]
```

And, we would like to display this list and the satisfaction related to each meal and to add an emoji matching the satisfaction rate. For example, this 🤩 if the satisfaction rate is equal or higher than 90%.


### How to solve our usecase ? / Design choices

We have first two very different options :
- OOP approach (?): add the emoji code to our model (either back-end or in our front-end app, on reception of the data /// complicated to send forn back end - ex: difficult encoding)
-> Defenitely an option, but we need to ask ourself if this information is relevant in our 'meal' model or not
// détailler ?
- FP approach (?): compute this information when needed (on display) without altering our model

Both approches have their pros and cons. Let's say we want to keep our model intact  for later usage and we go for the second approach.


### Pipes: a functionnal solution for a better SoC:

Before anything, we need somewhere to do a satisfaction / emoji mapping : 
```
const SATISFACTION_MAPPING = [
    [90, '🤩'], [70, '😃'], [50, '🙂'], [30, '😐'], [0, '🤢']
]
```

Again, 2 options are available:


- create a method in our component and call it from our template.

{% highlight javascript %}
    // component.ts
    convertToEmoji(rate) {
        element = SATISFACTION_MAPPING.find( (element) => rate > element[0] );
        if (0 < element.length) {
            return element[1];
        } else {
            return null;
        }
    }

{% endhighlight %}

```
<ul>
    <li *ngFor="let meal of meals">{{meal.name}} \{\{ convertToEmoji(meal.satisfaction) }}</li>
</ul>
```

- second option: creating a custom pipe:

