# Option Parsing in Parameter Lists in ES6 With Destructuring (and the case against it)

## Problem
You have a constructor function, and you want reasonable defaults for when certain fields if the options are missing, set to `undefined`, or `null`. You may want to save the settings of the option object into `this` in your constructor as well.

tl;dr:
* Not worth it, because destructuring throws exceptions on null destructuring targets.
* Destructuring for options is still a fine way to do things, but not in the parameter list.

## Requirements

* Have familiarity with destructuring in ES6
* Have familiarity with option parsing techniques in js other than destructuring.

## What is destructuring?

Destructuring is great. It's the counterpart to defining object literals in javasript. It's the ability to take apart an object with a literal expression.

If you come from a language that supports destructuring, like Clojure or Rust, you know how to nice destructuring can be. Here's an example of object destructuring:

```javascript
// We'll assign variables to properties of this object
const animal = {
        	giraffe: {
         		description: {
            		look: "orange themed slim cow", 
                	taste: "poisonous"
            	}
         	}
        };

// Without destructuring
var tasteA = animal.giraffe.description.taste;
var lookA = animal.giraffe.description.look;

// With destructuring
var {giraffe: {description: {look: lookB, taste: tasteB}}} = animal;
```

However, as this is a different syntax than has previously existed in javascript, you shouldn't be surprised that destructuring has its own kinds of exceptions.

```javascript
const nullThing = null;

// Blocks start with '{', so without the parens, this will look like a start of a block to the js parser, unless we wrap in parens
({attemptedDestructuring} = nullThing); 
// TypeError: Cannot match against 'undefined' or 'null'.
```

Under the hood, there is casting `nullThing` to an object, and it's not the one exposed to you by javascript's `Object()`. This internal casting throws exceptions on null, and this can be problematic for option parsing.

When parsing passed in options in a function, you could imagine doing something like

```javascript
const MyStream = function({
	verbose: myVerbose = false,
    hyperMode: myHyperMode = true
}) {
// ... function body
}

const streamInstance = new MyStream(null); // Exception!

```

Alright, we can combine some other syntax, the one for default values!

```javascript
const MyStream = function({
	verbose: myVerbose = false,
    hyperMode: myHyperMode = true
} = {}) {
// ... function body
}

const streamInstance = new MyStream(null); // Exception!
```

Maybe I put in this example in for me only...but when a function is given null, the default argument syntax **will not give you the default value**. However, if `undefined` is passed in, the default value will be used. If you pass in nothing: `new MyStream();`, you also won't get an exception for this same reason.


## Working around the null issue

### Rely on proven techniques

You can choose not to use destructuring.

The classic:
```javascript
// default to false
this.verbose = ('verbose' in options) ? options.verbose : false;
// default to an existing function within someObj
this.method = someObj[options.methodName] || someObj.defaultFunc;
```

It's slightly more verbose but you get more control.

### Destructuring can still work...but you wouldn't
The simplest solution if you still want to destructure is to simply ensure that the options object passed in is not null.

```javascript
function publicFacingFunction(options) {
    return MyStream(options = options || {})
}

const MyStream = function({
    verbose: myVerbose = false,
    hyperMode: myHyperMode = true
} = {}) {
// ... function body
}

var instance = new MyStream({verbose: true});


instance = new MyStream({});


instance = new MyStream(null);
```

But if you're doing that, why not just do it without the wrapper like so?:

```javascript
functionThatDestructuresOptions (options) {
	options = options || {};
    // Destructure here, where options 
    // is guaranteed not undefined nor null
    
    ({
    	verbose: myVerbose = false,
    	hyperMode: myHyperMode = true
	} = options);
}
```

Furthermore, if you want to save properties from the options object to `this`, you can't do that in the parameter list, because you don't have/can't get a reference to `this` in the parameter list.

```javascript
const Mystream = function({
	verbose: this.verbose = false
	hyperMode: this.hyperMode = false
}) {
	// function body
};

var instance = new MyStream({verbose: true, hyperMode: false});
// SyntaxError: Unexpected token this
```

So when destructuring options objects, just do it within the function body. In the end you save a small amount of characters. 
```javascript
	({
		objectMode: this.objectMode = true,
		verbose: this.verbose = false,
        // leaving trailing commas is actually legal, but don't do this
		hyperMode: this.hyperMode = false, 
	} = options)

```

If you have a lot of options, you'll refactor your options parsing into its own function anyway.



