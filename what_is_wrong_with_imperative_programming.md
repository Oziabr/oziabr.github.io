# What is wrong with imperative programming

### tl;dr

It is **flexible**.

While it allow programmers to solve any task by a numerous ways, it doesn't
force them to be consistent in the way they use in their code.

This is the main reason for decline in velocity and maintainability in most
large projects. And also for lots of bad blood between programmers.

1. [Imperative solution](#imperative-solution)
    1. [Example setup](#example-setup)
    1. [Iteration](#iteration)
    1. [Adding element](#adding-element)
    1. [Summary](#summary)
1. [Declarative alternative](#declarative-alternative)
    1. [Template engines (backend)](#template-engines-backend)
    1. [Angularjs template (frontend)](#angularjs-template-frontend)
    1. [Conclusions and thoughts](#conclusions-and-thoughts)
1. [Conclusions and thoughts](#conclusions-and-thoughts)

## Imperative solution

### Example setup

Consider the simle task of adding list to the page.

Say we have a list:
```js
var list = ['Apples', 'Oranges', 'Bananas'];
```

And an element on the page:
```html
<ul id='parent'></ul>
```

We will also assume a structure for our code (best case scenario), in which
iteration is separated from addition:
```js
function populateList (list) {
  // iteratively call addItem(item)
}
function addItem (item) {
  // locate parent (I'm leaving this hardcoded to simplify examples)
  // add single item to the list
}
populateList(list);
```

### Iteration

- Classic for-loop
```js
for (var i = 0; i < list.length; i++) {
  addItem(list[i]);
}
```
- Classic but uncommon while-loop
```js
var i = list.length;
while (i) addItem(list[list.length - (i--)]);
```
- functional way
```js
list.forEach(addItem);
```
- using jQuery iterator
```js
$.each(list, function(i, el) { addItem(el); });
```
- by invoking [recursion]
```js
function populateList (list) {
  if (!list.length) return;
  addItem(list[0]);
  populateList(list.slice(1));
}
```

### Adding element

- Clasic JS
```js
var el = document.getElementById('parent');
var li = document.createElement('li');
li.appendChild(document.createTextNode(item));
el.appendChild(li);
```
- using jQuery
```js
$("<li>"+item+"</li>").appendTo('#parent');
```

### Summary

As you can see we just got 10 different ways to implement same basic functionality.
But this is just atop of the problem:
- not every coder writes pure solutions, quite often implementation is a mix of different
ways
- code structure also a subject to change:
  - there can be no structure at all, and this gets coded into parent routine (worst case -
to be copy-pasted later)
  - it can be joined into one procedure
  - it can be implemented as a method(s) of an Object
  - or as something less common, like generator

## Declarative alternative

As an oposite to imperative coding, each example in this section is complete solution, the
difference only between used components. Note that it is simple, obvious and not a subject
to discussion.

### Template engines (backend)

- handlebars
```handlebars
<ul id='parent'>
  {{#each list}}
    <li>{{this}}</li>
  {{/each}}
</ul>
```
- Jade/Pug
```jade
ul#parent
  - each item in list
    li= item
```

### Angularjs template (frontend)

```html
<ul id='parent'>
  <li ng-repeat='item in list'>{{item}}</li>
</ul>
```

## Conclusions and thoughts

This shows the value of declarative code:
- hard to implement same functionality differently
- can be done by less qualified personel
- can be easily understood by any coder
- not inciting holy-wars

All this gives you maintainability, clearer estimates, happy
teammates and excludes [bus factor].

Meanwhile, to achieve same quality of imperative code, one should
went a great distance:
- stick to best practices, as well as to underlying platform style-guide
and conventions (if you have a platform, otherwise you'll have to build
the platform and write guides yourself)
- educate team in the former
- implement instruments for code analysis and enforcement of rules



[recursion]: https://www.refactoring.com/catalog/replaceRecursionWithIteration.html
[bus factor]: https://en.wikipedia.org/wiki/Bus_factor
