---
layout: post
title:  "[Angular] On the usage of FormArray"
date:   2020-04-05 21:16:17 +0100
categories: Angular
permalink: /angular/formArray
---

<br/>
`FormArray` is a class exported by the `FormsModule`, which is quite similar with the `FormGroup`. 
<br/>
<br/>
> For reminder, the `FormGroup` gathers and the values of child controls (which can be instances of FormControl, FormGroup or FormArray), and **reduces** them into a **non-iterable object**.

<br/>
As expected, `FormArray` outputs values into an object of type **array**:

{% highlight javascript %}
    const form = new FormArray([
      new FormControl('bana'),
      new FormControl('na'),
    ])
    console.log(form.value);    // console output: ["bana", "na"]
{% endhighlight %}

FormArray expose a quite useful API: 
 - `push`
 - `at`
 - ...

** the interesting usecase ** (and the reason why I got recently more interested into): make dynamic fields
-> addable
-> removable

<!-- This is particularly interesting when, for example, we need a form to propose an undefined number of  -->

<!-- Nb: also checks validity of each child, same ad FormGroup.

Interestinw when: 
Expected output value is an array
Input value is an array ?  -->
<br/>
<br/>
<h3>1. FormsArray in template driven :</h3>
<br/>
There is **no current formal implementation** of formsArray in template driven
Here is the [github issue][github-issue] 
The closest hack I could find consists in **naming the child controls** with increasing **numbers** (to simulate indexation), and mapping the output to a propper array.

<br/>

#### FormArray with simple controls:

{% highlight html %}
<!-- component.html -->
<form #form="ngForm" (ngSubmit)="submit(form.value)">
    <input *ngFor="let a of array; let i = index" ngModel name={ {i}}>
    <button type="submit">Submit</button>
</form>
{% endhighlight %}

{% highlight javascript %}
// component.ts
  array = ['a', 'b', 'c']

  submit(value) {
    console.log(value);     // console output: {0: "", 1: "", 2: ""}
    const arrayValue = Object.values(value);
    console.log(arrayValue);    // console output: ["", "", ""]
  }
{% endhighlight %}

<br/>

#### FormArray with group controls:
Nb: the hack consists in 'indexing' the directive `ngModelGroup`.

{% highlight html %}
<!-- component.html -->
<form #form="ngForm" (ngSubmit)="submit(form.value)">
    <div *ngFor="let array of array; let i = index" [ngModelGroup]="i">
        <input ngModel name="control1">
        <input ngModel name="control2">
    </div>
    <button type="submit">Submit</button>
</form>
{% endhighlight %}

{% highlight javascript %}
// component.ts
  array = ['a', 'b']

  submit(value) {
    const arrayValue = Object.values(value);
    console.log(arrayValue);    
    // console output: 
    // [{control1: "", control2: ""}, {control1: "", control2: ""}]
  }
{% endhighlight %}
 
 Main problem: when we do modifications on the formGroup, for example when removing a control. The index
 of the control will change, but not its name, which will trigger issues.

<br/>
<br/>
<h3>2. FormsArray in reactive forms :</h3>

Nb: to declare a reactive FormArray Group, we need to wrap it first into a formGroup parent:
[github-issue2]

As previously we can declare a 'simple' or 'more complex' formArray.

<br/>

#### FormArray with simple controls:

The main idea consists in:
 - using the `formBuilder.array()` method (or instantiating a new `FormArray` class) to build our reactive form
 - using the `formArrayName` directive
 - attributing an index as the `formControlName` values

{% highlight javascript %}
// formArray.component.ts
form = this.formBuilder.group({
    formArray: this.formBuilder.array([
      this.formBuilder.control(''),
      this.formBuilder.control('') 
    ])
});
{% endhighlight %}

{% highlight html %}
<form [formGroup]="form">
    <div formArrayName="formArray" 
        *ngFor="let control of form.get('formArray').controls; let i = index">
        <input [formControlName]="i">
    </div>
</form>

{{form.get('formArray').value | json}}
{% endhighlight %}

<br/>

#### FormArray with group controls:

Same procedure, but this time we are using the `formGroupName` directive instead of the `formControlName` directive.

{% highlight javascript %}
  // formArray.component.ts
  form = this.formBuilder.group({
    formArray: this.formBuilder.array([
      this.formBuilder.group(this.formGroupFactory()),
      this.formBuilder.group(this.formGroupFactory()),
    ])
  });

  private formGroupFactory() {
    return {
      field1: this.formBuilder.control(''),
      field2: this.formBuilder.control('')
    };
  }
{% endhighlight %}

{% highlight html %}
<form [formGroup]="form">
    <div formArrayName="formArray" 
         *ngFor="let control of form.get('formArray').controls; let i = index">
        <div [formGroupName]="i">
            <input formControlName="field1">
            <input formControlName="field2">
        </div>
    </div>
</form>

{{form.get('formArray').value | json}}
{% endhighlight %}


<br/>
<br/>
<h3>3. Dynamic fields with FormArrays :</h3>

#### Add fields:

Nb: from now, I will use an 'alias' of my FormArray instance, which I set through a getter. This also inforces the typing (otherwise TS would by default infer this is a FormArray)

{% highlight javascript %}
  get formArray() {
    return this.form.get('formArray') as FormArray;
  }
{% endhighlight %}

To add filed, one can simply use the method `push`:



[github-issue]: https://github.com/angular/angular/issues/9615
[github-issue2]: https://github.com/angular/angular/issues/30264