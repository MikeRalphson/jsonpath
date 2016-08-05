# Transforming JSON

#### Portions Copyright (c) 2006 Stefan Gössner

JSON is a lightweight text format for data interchange. It is often better suited for structured data than XML.

A frequently requested task with JSON data is its transformation to other formats, especially to XML or HTML for further processing.

The most obvious way to achive this, is to use a programming language (ECMAscript, Ruby,…) and the DOM-API.

In XML we can transform documents by another XML document containing transformation rules (XSLT) and applying these rules using an XSLT-processor.

Adopting that concept I have been experimenting with a set of transformation rules (written in JSON).

As a result in analogy to XML/XSLT the combination JSON/JSONT can be used to transform JSON data into any other format by applying a specific set of rules.

# Introducing JSONT
Let's start with a simple JSON object

```javascript
{ "link": {"uri":"http://company.com", "title":"company homepage" }}
```
which we want to transform into a HTML link element.

```html
<a href="http://company.com">company homepage</a>
```
For doing this we can write a corresponding rule

```javascript
{ "link": "<a href=\"{link.uri}\">{link.title}</a>" }
```
and using a processor like jsonT(data, rules) we can apply the given rule to the JSON data resulting in the output string above.

# Basic Rules
A set of transformation rules is written using the object literal notation. So each rule is a name/value pair. The rule name usually is an expression for accessing an object member. The rule value is either a string or a function with a single argument, which are evaluated at transformation time.

"name": "transformation string"
"name": function(arg){ … }

The transformation string itself can contain one or more expressions enclosed in curly braces

{expr}

which always resolve to a string value.

```If expr references a rule name, it results in either the transformation string or the return value of the implicit transformation function of that rule.
If expr evaluates to a primitive data type, its value is converted to a string.
If expr evaluates to an array/object, each array element/object member is processed accordingly.
The shortcut $ as part of the expr is substituted by the rule name.
If expr has the explicit form @name(expr), the function belonging to the rule name is called and its return value is converted to a string.
The outer JSON object can be accessed using the keyword self.
```
Rule names for array elements use the syntax name[*]. When using the $ shortcut in transformation string, the '*' resolves to the actual array index.

Object members, which have no transformation rule assigned and are not directly or indirectly referenced, as well as expressions evaluating to undefined don't create output.

# Some examples
vector geometry

```javascript
{ "line": { "p1": {"x":2, "y":3},
            "p2": {"x":4, "y":5} }}
+

{ "self": "<svg>{line}</svg>",
  "line": "<line x1=\"{$.p1.x}\" y1=\"{$.p1.y}\"" +
                "x2=\"{$.p2.x}\" y2=\"{$.p2.y}\" />" }
=

<svg><line x1="2" y1="3"x2="4" y2="5" /></svg>
simple array

["red", "green", "blue"]
+

["self": "<ul>\n{$}</ul>",
 "self[*]": "  <li>{$}</li>\n"]
=

<ul>
  <li>red</li>
  <li>green</li>
  <li>blue</li>
</ul>
two-dimensional array and implicit function rule

{ "color": "blue",
  "closed": true,
  "points": [[10,10],[20,10],[20,20],[10,20]] }
+

{ "self": "<svg><{closed} stroke=\"{color}\" points=\"{points}\" />"+
          "</svg>",
  "closed": function(x){return x ? "polygon" : "polyline";}, 
  "points[*][*]": "{$} " }
=

<svg><polygon stroke="blue" points="10 10 20 10 20 20 10 20 " /></svg>
```
