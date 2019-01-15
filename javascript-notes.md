# Javascript Notes

Javascript is a programming language used primarily for web programming and web applications. 

## Basic JavaScript

### General Functions:
- `confirm(<string>)`{.javascript} : Creates a confirmation box (dialog box)
- `prompt(str_1, str_2)`{.javascript} : Gets input from user through prompt dialog box. str_1 is in the dialog box and str_2 is in the text box
- `str.length`{.javasctipt} : Gets the length of a string
- `console.log(str)`{.javascript} : Print to console. effectively print to stdout
- `str.substring(x, y)`{.javascript} : Get substring including x not including y
- `var x = y`{.javascript} : Declare a variable
- `var rA = [el_1, el_2, el_3, ...]`{.javascript} : Declare an array
- `var rA = new Array()`{.javascript} : Declare empty array
- `isNaN(someVal>)`{.javascript} : Returns true if the parameter is not a number
- `typeof(var)`{.javascript} : Returns a string representing the type of the variable
- `someObject.hasOwnProperty(prrop)`{.javascript} : Returns true if the object has the specified property

Javascript variables have three states `const`, `let`, and `var`. `const` describes a variable that won't get reassigned. `let` and `var` have subtle differences. `let` variables are defined within the current block (scope). `var` variables are defined throughout the entire program. 


### Logical Operators:
- `&&` : and
- `||` : or
- `!` : not

### Control flow:
```JS
if (<boolean>) {
	//action
} else if {
	//action
} else {
	//action
}

switch(var) {
	case val_1:
		//action
		break;
	case val_2:
		//action
		break;
	...
	default:
		//action
		break;
}
```

### Function Declaration
```JS
let foo = (params) => {
	// instructions
	return ret_value
};

var foo = function(param) {
	//instructions
	return ret_value
};

function foo(param) {
	// instructions
	return ret_value
}
```


### Object Declaration
```JS
// Creating empty object
var object = { key_one: val_one, key_two: val_two };
var object = new Object();
// add key value pairs by:
object[key_n] = val_n;

class ClassName {
	constructor(params) {
		this.param1 = param1;
		this.param2 = param2;
	}
	// define static and instance methods
}

let var = new ClassName(parameters);
// Note parameters can also be methods
// Create an instance method.
ClassName.prototype.foo = function(<parameters>) {
	//function body
};

// Set inheritance
ClassName.prototype = new Parent_Class();
```

### Loops
```JS
for (let i = 0; i <= end_val; i+=n) { 
	//loop body
}

for (var x in y) { //for each loop
	//loop body
}

while (some_bool) { //while loop
	//loop body
}

do { //do while loop
	//loop body
} while (some_bool);
```

## JQuery
JQuery is another means of using javascript to create dynamic, interactive webpages.
Setup
in the head add a javascript script:

```HTML
<script type = "text/javascript" src = "path/source.js"></script>
```

(note that the script tag is not self closing)

In source.js (can be any js file name) add the basic line
```Javascript
$(document).ready(function() {
	//function body
});
```
Explanation:
- `$(PARAM)` - turns param into JQuery object
- `document` - tells JQuery to act on the html document
- `ready(param)` - tells JQuery to act as soon as the html document is "ready". The parameter is the event that happens as soon as the html document is ready (ie a function).

The command `$('CSS SELECTOR')` creates a JQuerry object out of an HTML element in much the same way we would in CSS.
It is convention to declare all JQuery variables starting with $

```Javascript
$target = $('div')
```

We can select multiple selectors by using comma delimination

```Javascript
$('SELECTOR_1, SELECTOR_2, ..., SELECTOR_N')
```

JQuery selectors can be quite complex. See [a comprehensive guide](https://www.w3schools.com/jquery/jquery_ref_selectors.asp) on how to create selectors 

When selecting a specific JQuery object we can use $(this) to refer to that element.

```Javascript
$('div').mouseenter(function() {
	$(this).hide() //just hides the div that was entered by the mouse
}

$('div').mouseenter(function() {
	$('div').hide() //hides all divs
}
```

The "target" element selector can also be passed into the function as a param

```Javascript
$('div').mouseenter((e) {
	$(e).hide() //just hides the div that was entered by the mouse
}
```

We can also instantiate new HTML elements as JQuery objects as follows

```Javascript
$var = $('FULL TEXT OF HTML ELEMENT');
```

Writing apps with JQuery is relatively straight forward. After defining your base HTML, use the CSS selectors to target specific elements and apply JQuery functions to those objects to create animations and affects. See the [jquery api](http://api.jquery.com/). 

## React

## Vue

