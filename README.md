# Server-side Templating w/ Nunjucks!

Let’s look at the basics of Nunjucks, “a simple and powerful JavaScript Template Engine”, built by Mozilla for NodeJS.

First off, a templating engine creates web pages (or views) dynamically by combining variables and programming logic with HTML. Essentially, you can add placeholders (or tags) to your HTML that are replaced by actual code defined from your router or controller. In general, tags, for the majority of templating engines, fall within one of two categores-

- Variables/Output Tags
  - surrounded by double curly brackets ``
  - output the results of a logic tag or a variable to the end user
- Logic Tags
  - surrounded by {% ... %}
  - handle programming logic like loops and conditionals

Before diving in,
- grab the basic project structure from Github
- install the dependencies via `npm i`
- then run the server

Pay attention to where we initialize Nunjucks and set it as the templating language in app.js:
```js
//import package
var nunjucks = require('nunjucks');
// nunjucks configure method
nunjucks.configure(viewFolders, {
    express: app,
    autoescape: true
});

//express view engine settings
app.engine('html', nunjucks.render) ;
app.set('view engine', 'html') ;
```

### Output Tags

Let’s start with some basic examples…

First, we can pass variables from our route handlers/view functions directly to the templates.

Update the index.html file:
```html
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  <h1>{{title}}</h1>
</body>
</html>
```
Now, we can pass in a variable called title to the template from app.js:

```js
// *** main routes *** //
app.get('/', function(req, res) {
  res.render('index.html', {title: 'Nunjucks Primer!'});
});
```
Fire up the server and test this out. Nice. Try adding another variable to the template.

index.html:
```html
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  <h1>{{title}}</h1>
  <p>{{description}}</p>
</body>
</html>
```
app.js:
```js
app.get('/', function(req, res) {
  var title = 'Nunjucks Primer!'
  var description = 'Nunjucks is "a simple, powerful, and extendable JavaScript Template Engine" for NodeJS.'
  res.render('index.html', {title: title, description: description});
});
```
Keep in mind that all variable outputs are automatically escaped except for function outputs:

```js
// *** main routes *** //
app.get('/', function(req, res) {
  var title = 'Nunjucks Primer!'
  var description = 'Nunjucks is "a simple, powerful, and extendable JavaScript Template Engine" for NodeJS.'
  function allthethings() {
    return '<span>All the things!</span>';
  }
  res.render('index.html', {
    title: title,
    description: description,
    allthethings: allthethings
  });
});
```
Don’t forget to call the function in the template - <p></p>

Please see the official documentation for more info on output tags.

Filters

Filters, which are just simple methods, can be used to modify the output value. To illustrate some examples, add another route handler to app.js, like so:

```js
app.get('/filter', function(req, res) {
  res.render('filter.html', {
    title: 'Hello, World!',
    nameArray: ['This', 'is', 'just', 'an', 'example']
  });
});
```

Now just add a new template, filter.html, adding in a number of filters:
```html
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  <h1>{{title | upper}}</h1>
  <p>{{nameArray | join(' ')}}</p>
</body>
</html>
```
Check out all the available filters. You can also extend the functionality of Nunjucks by adding your own custom filters!

Logic Tags

As the name suggests, logic tags let you use, well, logic in your templates.

IF statements

Here’s a simple example…

app.js:
```js
app.get('/logic', function(req, res) {
  var title = 'Nunjucks Logic!'
  res.render('logic.html', {title: title});
});
```
logic.html:
```html
<!DOCTYPE html>
<html>
<head>
  <title>{% if title %}{{title}}{% endif %}</title>
</head>
<body>
  {% if title %}
    <h1>{{title}}</h1>
  {% endif %}

  <h2>{{ "true" if foo else "false" }}</h2>
</body>
</html>
```
Test out some more examples of if, elif, and else.

Loops

How about a for loop?

app.js:

```js
app.get('/logic', function(req, res) {
  var title = 'Nunjucks Logic!'
  var numberArray = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  res.render('logic.html', {title: title, numberArray: numberArray});
});
```

logic.html:


```html
<!DOCTYPE html>
<html>
<head>
  <title>{% if title %}{{title}}{% endif %}</title>
</head>
<body>
  {% if title %}
    <h1>{{title}}</h1>
  {% endif %}
  <ul>
    {% for num in numberArray %}
      <li>{{ num }}</li>
    {% endfor %}
  </ul>
</body>
</html>
```
Need to reverse the loop?

Simply add a filter:


```html
<!DOCTYPE html>
<html>
<head>
  <title>{% if title %}{{title}}{% endif %}</title>
</head>
<body>
  {% if title %}
    <h1>{{title}}</h1>
  {% endif %}
  <ul>
    {% for num in numberArray | reverse %}
      <li>{{ num }}</li>
    {% endfor %}
  </ul>
</body>
</html>
```

What would a basic loop and filter look like?

```html
<!DOCTYPE html>
<html>
<head>
  <title>{% if title %}{{title}}{% endif %}</title>
</head>
<body>
  {% if title %}
    <h1>{{title}}</h1>
  {% endif %}
  <ul>
    {% for num in numberArray %}
      {% if num % 2 === 0 %}
        <li>{{ num }}</li>
      {% endif %}
    {% endfor %}
  </ul>
</body>
</html>
```
You could also write a custom filter for this if you needed to do the same filtering logic a number of times throughout your application.

There’s also a number of helper methods available with loops:

- loop.index: returns the current iteration of the loop (1-indexed)
- loop.index0: returns the current iteration of the loop (0-indexed)
- loop.revindex: returns the number of iterations from the end of the loop (1-indexed)
- loop.revindex0: returns the number of iterations from the end of the loop (0-indexed)
- loop.key: returns the key of the current item
- loop.first: returns true if the current object is the first in the object or array.
- loop.last: returns true if the current object is the last in the object or array.
- loop.length: total number of items


Try some of these out:

```html
{% for num in numberArray | reverse %}
  {% if num % 2 === 0 %}
    <li>{{ num }} - {{loop.index}}</li>
  {% endif %}
{% endfor %}
```

### Template Inheritence

Logic tags can also be used to extend common code from a base template to child templates. You can use the block tag to accomplish this.

Create a new HTML file called layout.html:

<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  {% block content %}{% endblock %}
</body>
</html>
Did you notice the {% block content %}{% endblock %} tags? These are like placeholders that child templates fill in.

Add another new file called test.html:
```html
{% extends "layout.html" %}
{% block content %}
  <h3> This is the start of a child template</h3>
{% endblock %}
```

So, the blocks - {% block content %}{% endblock %} correspond to the block placeholders from the layout file, and since this file extends from the layout, the content defined here is placed in the corresponding placeholders in the layout.
