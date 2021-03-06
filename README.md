# express-json-views
A JSON view engine for express to render objects, lists with views, sub-views and helpers.

[![npm version](https://badge.fury.io/js/express-json-views.svg)](https://badge.fury.io/js/express-json-views)

## Motivation
Even though ```express``` can send JSON responses out of the box, in a lot
of cases the response object or database models needs to be formatted. Think like DTOs in Java.
Unnecessary or sensitive information like passwords need to be omitted,
values need to be formatted and sub-objects/lists need to be
rendered as well, which also should be able to utilize the already mentioned features.

### Features
- Omit values
- Rename object keys
- Format values
- Sub views and lists
- View caching

## Setup
```bash
$ npm install --save express-json-views
```

```js
var app = require('express')();
var viewEngine = require('express-json-views');

app.engine('json', viewEngine({
       helpers: require('./views/helpers')
   }));
app.set('views', __dirname + '/views');
app.set('view engine', 'json');
```

## Features & examples

### Defining views
Views are defined using json files and located in a single directory in your project.
```json
{
	"id": {},
	"slug": {
		"format": "getPostSlug"
	},
	"title": {},
	"content": {
		"from": "content_text"
	},
	"author": {
		"view": "user"
	},
	"comments": {
		"view": "comment"
	},
	"published_date": {
		"format": "customDateFormat",
		"dateFormat": "YYYY-MM-DD"
	}
}
```
The keys of the view are the keys which will be rendered into to resulting object.
The default configuration ```{}``` just copies the value from the passed data.
Possible configurations are:

- **from**: Uses this value to lookup the value in the data object instead of the view key.
- **format**: Calls a helper function with this name. Passes the value and the full object as arguments.
- **view**: Defines the view if this value should be rendered with a different view. Value must be an ```Array``` or objects or an ```Object```.

### Rendering objects and lists
You don't have to worry about whether you want to render a single object or a list of objects.
The view engine detects if you pass an array and will iterate over it. call ```res.render(...)```
and pass the view name and an object with a property called **data** which contains your data.
```js

app.get('/posts', function (req, res) {

	var posts; // Query from database

	res.render('post', {
		data: posts
	});

});
```

### Helpers
To define helpers you pass an object of functions when creating the view engine.
```js
var helpers = {
	date: function (value, object) {
		if (!value) {
        	return null;
        }
        return dateFormat(value, 'yyyy-mm-dd');
	},
	customDateFormat: function(value,object,options){
		if (!value) return null;
		var format = options.dateFormat || "yyyy-mm-dd";
		return dateFormat(value,format);
	}
};

app.engine('json', viewEngine({ helpers: helpers }));
```

Helper functions get 2 arguments, value and the full object. To calculate a value based on other values
in the object you can use the second argument.
```js
// value: the value of the field
// object: the object of the data where the value come from
// options passed to the parameter of view
var helpers = {
	comment_count: function (value, object,options) {
		// Value will be null since this is a dynamic property and not present in the data object
		return object.comments ? object.comments.length : 0;
	}
};
```
If you return a ```Promise``` the view engine will resolve use it. Though I would strongly advise you, not to do async calls here.

## Test
```bash
$ npm test
```
