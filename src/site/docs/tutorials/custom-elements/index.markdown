---
layout: default
title: "Define a Custom DOM Tag"
description: "Define a custom DOM element tag with help from the Web UI package"
has-permalinks: true
tutorial:
  id: web-components
next: fetchdata/
next-title: "Fetch Data Dynamically"
prev: templates/
prev-title: "Use Templates"
---

{% capture whats_the_point %}

* Make your own DOM element tags with the &lt;element&gt; tag.
* A custom element requires a template and can have a script.
* Use data binding to connect Dart variables to content.
* Directly attach event handlers in HTML.

{% endcapture %}

{% capture sample_links %}

<p>
Get the source code for the samples featured in this target:</p>

<ul>
  <li>
    <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/drseuss"
       target="_blank">drseuss</a>
  </li>
  <li>
    <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis"
       target="_blank">convertthis</a>
  </li>
</ul>

{% endcapture %}

{% capture content %}

<div class="tute-target-title">
<h1>{{page.title}}</h1>
<h3>Create your own, customized DOM elements.</h3>
</div>

<hr>

<aside class="alert" style="background-color:Lavender;color:SlateBlue">
  <font size="24">
  <i class="icon-bullhorn"> </i>
  </font>

  The Dart Web UI team recently
  <a href="https://groups.google.com/a/dartlang.org/forum/#!topic/web-ui/6laXXxRtA7k" target="_blank">announced</a>
  a port of the Polymer project:
  <a href="https://pub.dartlang.org/packages/polymer" target="_blank">polymer.dart</a>.

  We've converted most of the tutorial Web UI examples and compiled some
  <a href="https://github.com/dart-lang/dart-tutorials-samples/blob/master/web/to-polymer-notes.txt" target="_blank">notes</a>
  along the way.
  Check out source code for the polymer version of the
  <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08-polymer/drseuss"
     target="_blank">drseuss</a>
  example.
</aside>

<hr>

Custom elements are one kind of web component
defined in the Web Components model.
Using templates and scripts,
a custom element defines a new DOM tag
that extends an existing tag.
This target shows you how to define a custom element
and how to create an instance of that element.
Dart's implementation of custom elements is in the
<a href="https://pub.dartlang.org/packages/web_ui"
   target="blank">Web UI package</a>,
which is required to run the examples in this target.
A previous target,
<a href="/docs/tutorials/web-ui/">Get Started with Web UI</a>,
shows you how to install the Web UI package.

* [About custom elements](#about-custom-elements)
* [Connecting the files](#connecting-the-files)
* [Instantiating a custom element](#instantiating-a-custom-element)
* [Defining a custom element](#defining-a-custom-element)
* [Providing a Dart script](#providing-a-dart-script)
* [Capturing content from the HTML page](#capturing-content-from-the-html-page)
* [Using two-way data binding](#using-two-way-data-binding)
* [Handling events](#handling-events)
* [Other resources](#other-resources)

##About custom elements

A custom element defines a new DOM tag
that extends an existing one.
You define a custom element tag
by using &lt;element&gt; and
providing it with a name for the new tag
and the name of the tag that it extends.
This code snippet extends the standard HTML &lt;button&gt; tag.
The name and extends attributes are both required.

{% prettify html %}
<element name="x-fancy-button" extends="button"> ... </element>
{% endprettify %}

To create an instance of a custom element,
use the element name as you would a regular HTML tag.
For example, to create an instance of x-fancy-button:

{% prettify html %}
<x-fancy-button> ... </x-fancy-button>
{% endprettify %}

You can also create a complex,
reusable element by bundling elements together inside the custom element.
Running below is an instance of a custom element, called `x-converter`,
that extends the &lt;div&gt; tag
and contains several elements within it.
Everything within the yellow box is part of the x-converter element.
Try it! Type a number into one of the fields and press return.

<iframe class="running-app-frame"
        style="height:100px;width:400px;"
        src="http://dart-lang.github.io/dart-tutorials-samples/web/target08/drseuss/web/out/drseuss.html">
</iframe>

You can find the complete source code for this sample on github at
<a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/drseuss" target="_blank">drseuss</a>.

This repository includes the pubspec.yaml and build.dart
files necessary to install the Web UI package and build the app.
Refer to
<a href="/docs/tutorials/web-ui/">Get Started with Web UI</a>
for instructions.

The two input fields, the two labels, the left- and right-facing arrows,
and the Dart code that implements the arithmetic calculation
work in concert to provide a single widget that converts one number to another.
This custom extension of &lt;div&gt; wraps all
of the necessary UI elements and their behavior together into a
reusable, sharable piece of code.
Using a mash-up of HTML and Dart
and supporting functionality in the Web UI package,
x-converter processes change events on both fields,
converts the value,
and puts the result in the opposite field.

Instances of this custom element can be configured 
with different labels and conversion ratios.
This particular instance of x-converter
maintains a ratio of 1:2 between the left and right numbers.

The remaining sections
describe the code that creates instances of the x-converter,
the HTML code that defines the &lt;x-converter&gt; tag,
and the Dart code that implements the behavior.

##Connecting the files

The embedded app just below contains
three x-converter elements,
each configured with different labels and
a different ratio.
Try it! Enter numbers into each of the fields.

<iframe class="running-app-frame"
        style="height:225px;width:400px;"
        src="http://dart-lang.github.io/dart-tutorials-samples/web/target08/convertthis/web/out/convertThis.html">
</iframe>

You can find the complete source code for this sample on github at
<a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis" target="_blank">convertthis</a>.

This app is implemented with these Dart and HTML files:

| File | Description|
|---|---|
| <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis/web/converter-element.html" target="_blank">converter-element.html</a> | Defines the UI for the x-converter custom element |
| <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis/web/convertercomponent.dart" target="_blank">convertercomponent.dart</a> | Defines the behavior of the x-converter custom element |
| <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis/web/convertThis.html" target="_blank">convertThis.html</a> | Creates three different instances of the x-converter |
| <a href="https://github.com/dart-lang/dart-tutorials-samples/tree/master/web/target08/convertthis/web/convertthis.dart" target="_blank">convertthis.dart</a> | Contains the Dart main() function, which in this program does nothing |
{: .table}

HTML code in `converter-element.html`
implements the look of the x-converter custom component.
Dart code in `convertercomponent.dart`
implements the behavior.
To connect the Dart code to the HTML code use the &lt;script&gt; tag
within the &lt;element&gt; tag that defines the custom element:

{% prettify html %}
<element name="x-converter" ...>
  ...
  <script type="application/dart" src="convertercomponent.dart"> </script>
</element>
{% endprettify %}

The  `convertThis.html` file defines the main HTML page
and uses the x-converter element.
The main HTML file must include `converter-element.html` to use it.
You do this by importing the file with the &lt;link&gt; tag
in the header.

{% prettify html %}
<html
  <head>
  ...
  <link rel="import" href="converter-element.html">
  </head>
  ...
</html>
{% endprettify %}

This diagram summarizes the relationships between the files:

![Connecting Dart and HTML files](images/files-relationship.png)

##Instantiating a custom element

To create the three converter elements
the converter example uses &lt;x-converter&gt;...&lt;/x-converter&gt;.
It's just as easy as creating a new div or an input field.
**Important:** You _must_ close each custom element,
using a closing tag such as &lt;/x-converter&gt;.

![HTML code to create three x-converter elements](images/html-create-converters.png)

As with other DOM elements,
custom elements can have attributes or content,
some of which might be required.
x-converter requires a simple ratio to use for the conversion
and two labels (each with a specific class name).

![Detail of HTML code that creates an x-converter instance](images/html-converter-detail.png)

The &lt;x-converter&gt; tag expects an attribute called `ratio`,
which is a template expression containing a number.
Recall that template expressions,
written as \{\{_expression_\}\},
bind the value of a Dart variable to HTML content.
In this example, the number within the curly brackets
is bound to the Dart variable called `ratio`,
which is a number in the Dart code
and is used to convert the numbers.
If you don't provide the ratio, it defaults to 0.5.

x-converter also expects two labels,
one for the left input field and one for the right.
The label for the left input field should have the class name `label-one`
and the right one should be `label-two`.
The labels have no default value
and don't appear if they are not provided.

##Defining a custom element

Define a custom tag using the &lt;element&gt; tag.
Besides the required attributes `extends` and `name`,
the element can also have a `constructor` attribute.

![A custom element requires two attributes: extends and name](images/custom-element.png)

<dl>
  <dt> extends </dt>
  <dd> Identifies the existing DOM element
  on which the custom tag is based.
  You should either extend an element that closely matches your new element,
  or you should extend div or span.
  </dd>

  <dt> name </dt>
  <dd>
  Gives a name to the new element;
  use this name as an HTML tag to create an element of this kind.
  In the Web UI implementation,
  custom element names must begin with `x-`.
  </dd>

  <dt> constructor </dt>
  <dd>
  Specifies the Dart type associated with the custom element.
  It is necessary when the name of a custom element (minus the x- prefix)
  differs from the name of the Dart class that implements its behavior.
  </dd>
</dl>

Within the &lt;element&gt; and &lt;/element&gt; tags,
a custom element definition contains two parts: a template and a script.

![Template expressed in HTML, script written in Dart](images/template-script.png)

<dl>
  <dt> &lt;template&gt; </dt>
  <dd>
  Contains HTML code
  that describes the element's structure&mdash;its user interface.
  The template comprises any valid HTML code between
  &lt;template&gt; and &lt;/template&gt;.
  The template HTML is rendered when a custom element is instantiated.
  </dd>

  <dt> &lt;script&gt; </dt>
  <dd> Specifies a Dart script.
  It contains the Dart class that implements the behavior of the element.
  Though not required,
  most custom elements have an associated script.
  </dd>
</dl>

##Providing a Dart script

To specify a script for a custom element,
use the HTML &lt;script&gt; tag within the element definition.
Identify it as Dart with the `type="application/dart"` attribute.

The Dart script must contain a class that is a subclass of WebComponent.
Usually the name of the Dart class
is the same as the name of the custom element it implements
(minus the x- prefix).

However, in the x-converter example,
the names are different:
the custom element is x-converter
and the Dart class is ConverterComponent.
The following diagram shows the connections
between the HTML and Dart code for the x-converter element.

![HTML and Dart connections](images/converter-files.png)

1. Associates a Dart script with the custom element
and provides the name of a Dart source code file.
1. Declares the Dart type associated with this custom element
and the constructor to call to instantiate the element.
Today, this must be a subclass of WebComponent.
In the future, it will be possible to use a subclass
of the HTML element declared in the extends attribute.
If the constructor is not explicitly named,
it is assumed to be the CamelCase version of the
name of the custom element minus the `x-` prefix.
In this example, it would be Converter.

##Capturing content from the HTML page

Using the &lt;content&gt; tag
you can embed nodes from the HTML page
within a custom element.
The nodes are provided when the custom element is instantiated.

Here is the complete code for the x-converter definition
alongside the code that creates one instance of it.

![The content tag uses the content specified in the instance](images/converter-html-code.png)

The HTML code within the &lt;template&gt; definition describes 
the child elements of an x-converter.
The two input fields and the other elements
are created for each instance of x-converter.

The &lt;content&gt; tag is a placeholder for the content of the created instance.
The x-converter example uses the &lt;content&gt; tag twice,
once for the left-hand label and once for the right-hand label.
In this example,
the &lt;content&gt; tag uses a CSS selector to select an element by class name.
A pair of empty tags, &lt;content&gt; &lt;/content&gt;,
captures all of the instance's children.

##Using two-way data binding

A <a href="/docs/tutorials/web-ui/">previous target</a>
described the Web UI package's two-way data binding feature.
The x-converter element uses two-way data binding
to connect the value of each input field to a Dart string variable.
With observables the Web UI's system keeps the value of the bound Dart string
in sync with the value of the input field.

![The value of the two input fields are bound to Dart strings](images/converter-data-binding.png)

##Handling events

The Web UI package provides a short-hand for
directly attaching a Dart event handler to an HTML element.
Each of the two input fields contained by the x-converter custom element
have an event handler.

![Dart event handlers are attached to the two input fields](images/converter-events.png)

##Other resources

<ul>
  <li> Check out
       <a href="/docs/cookbook/">
       <i class="icon-food"> </i> Dart Cookbook</a>.
       You'll find several
    <a href="/docs/cookbook/#web-ui">Web UI recipes</a>.
  </li>
  <li>
  Sigmund Cherem's article,
  <a href="/articles/dart-web-components/" target="_blank">Web UI Package</a>,
  contains several interactive examples on the page
  and the corresponding source code.
</li>
<li>
  The Web UI package contains several examples,
  including a version of
  <a href="https://github.com/dart-lang/web-ui/tree/master/example/todomvc"
     target="_blank">TodoMVC</a>.
</li>
</ul>

{% endcapture %}

{% include tutorial.html %}
