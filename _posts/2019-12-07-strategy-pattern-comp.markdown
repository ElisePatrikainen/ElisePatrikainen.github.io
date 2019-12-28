---
layout: post
title:  "Strategy pattern in Angular with dynamic components loading"
date:   2019-12-05 21:16:17 +0100
categories: jekyll update
---

<h3>Introduction:</h3>

**Strategy pattern**: behvioural pattern which promotes the selection of an algorithm at runtime. It comes from the book ‘Design Patterns’ from the ‘Gang of Four’.

**Dynamic component instanciation**: loading 'manually' a component. We will use for this the 
`ComponentFactoryResolver`class: an Angular Core injectable able to map a component type to its factory.

**The idea**: use the factory resolver to instanciate one relevant component (strategy pattern) among a selection of components.

<br/>
<h3>An example usecase:</h3>

Let's take the example of a dashboard displaying several widgets.

The first architecture idea which comes in mind is this: one component displaying all the widget components.

{% highlight html %}
<!-- dashboard.component.html -->
<div>
  <app-widget1></app-widget1>
  <app-widget2></app-widget2>
  <app-widget3></app-widget3>
  <app-widget4></app-widget4>
</div>
{% endhighlight %}

This could do the job, but of course there are non negligible limitations with this approach. One is particularily obvious: the list of widgets is **fixed** 👎. In real life, we would probably expect the list to change according to the usecase.
<br/>

Threrfore, we need something **more flexible**. A good solution seems to be wrapping our widgets into a ‘dump’ component, a **‘wrapper’**, whose job would be to **instantiate the relevant componant**, according to a value passed as input.

<br/>
Let's review our DashboardComponent:

{% highlight javascript %}
// dashboard.component.ts
 
// nb: this array would be probably be fetched from back-end 
// or computed somewhere else (SoC as always)
widgets = ['widget1', 'widget2', 'widget3', 'widget4']
{% endhighlight %}

{% highlight html %}
<!-- dashboard.component.html -->
<div *ngFor="let widget of widgets">
  <app-widget-wrapper [input]="widget"></app-widget-wrapper>
</div>
{% endhighlight %}

So now let’s see how to instantiate the relevant widget component in the WidgetWrapperComponent class.

<br/>
<h3>The pragmatic approach:</h3>

{% highlight javascript %}
// widget-wrapper.component.ts
@Input() widget;
{% endhighlight %}

{% highlight html %}
<!-- widget-wrapper.component.html -->
<app-widget1 *ngIf="'widget1' === widget"></app-widget1>
<app-widget2 *ngIf="'widget2' === widget"></app-widget2>
<app-widget3 *ngIf="'widget3' === widget"></app-widget3>
<app-widget4 *ngIf="'widget4' === widget"></app-widget4>
{% endhighlight %}


This is already better. But this approach is still not very satisfying:
- no good separation of concerns (SoC) : the wrapper template is not the ideal place to store the widget list
- coupling between the wrapper and the widgets => it requires to modify the template each time the widget list changes
- not elegant 

<br/>
<h3>The ‘strategy’ approach: </h3>
Here is where the strategy pattern and the factory resolver come into play. The idea is, **instead of listing** all the possible widgets in the template, to **directly instanciate the relevant component**.

<br/>
More accurately, we will :
- pass the relevant component type to the `ComponentFactoryResolver`, which will fetch the required component factory
- pass this component factory to a `ViewContainerRef`(a representation of an HTML container), which will instantiate our component and append it to the container
<br/>
> Nb: before anything else, please add the dynamically loaded components into the `entryComponents` meta-property (?) of the NgModule to prevent the blocking error `No component factory found for Widget1Component`.

<br/>
#### 1. Map components name keys to components types : ####
First we need a mapping between this :

{% highlight javascript %}
// dashboard.component.html
widgets = ['widget1', 'widget2', 'widget3', 'widget4']
{% endhighlight %}

and the widget component types. So we can do a simple mapping in a separate file :

{% highlight javascript %}
// widget-mapping.ts
 
import {Widget1Component} from './widgets/widget1/widget1.component';
import {Widget2Component} from './widgets/widget2/widget2.component';
 
export const DASHBOARD_WIDGET_MAPPING = {
    widget1: Widget1Component,
    widget2: Widget2Component
  }
{% endhighlight %}

Which will be imported in the `WidgetWrapperComponent`.

<br/>
#### 2. Get the component factory : ####

Now, in each instance of `WidgetWrapperComponent`, we need to pass the component references to the `ComponentFactoryResolver`, which will fetch the component factory.

{% highlight javascript %}
// widget-wrapper.component.ts
@Input('widget') widget;
 
constructor(private componentFactoryResolver: ComponentFactoryResolver) { }
 
ngAfterViewInit() {
    const factory = this.componentFactoryResolver.resolveComponentFactory(
      DASHBOARD_WIDGET_MAPPING[this.widget]);
}
{% endhighlight %}

<br/>
#### 3. Create a new component instance and append it to the template : ####

For this, we need to:
- define a ‘container’ in our template, which will host our new component instance
- represent this container as a `ViewContainerRef` instance, and use its API to create the widget component instance and insert it in the template

{% highlight html %}
<!-- widget-wrapper.component.html -->
<div #widgetContainer></div>
{% endhighlight %}

> we cound use any html / other (?) element - point sur ngTempte ??

{% highlight javascript %}
// widget-wrapper.component.ts
@ViewChild('widgetContainer', {read: ViewContainerRef}) widgetViewRef: ViewContainerRef;
 
ngAfterViewInit() {
    const factory = this.componentFactoryResolver.resolveComponentFactory(
      DASHBOARD_WIDGET_MAPPING[this.widget]);
    // instantiate + append the component
    const componentRef = this.widgetViewRef.createComponent(factory);
}
{% endhighlight %}

> we need to do this in the context (?) of the AfterViewInit hook, because we need the view to have been inited for our `ViewContainerRef` to have been defined.

A complete demo can be found at ...