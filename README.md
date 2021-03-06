NS.UI.Forms
===========

Generate forms for Backbone models

## Status ##

Work in progress, use at your own risk...

## Features ##

* Automatically generate forms based on model schema
* On-the-fly validation
* Classic stacked layout (one field per line) or inline layout (useful for tabular list of related objects)
* Extendable, customizable

## Example ##

The file `demo.html` shows the basic setup.

## Documentation (draft) ##

### Overview ###

A **form** is a regular Backbone **view** which provides an editing UI for a **model instance**. The form engine will read the **model schema** of that instance and create an **editor** for every editable attribute.

### Dependencies ###

NS.UI.Forms depends on [BackboneJS](http://backbonejs.org/) and [LayoutManager](https://github.com/tbranyen/backbone.layoutmanager/).

The HTML tags and CSS classes in the default templates designed for use with [Twitter Bootstrap](http://twitter.github.io/bootstrap/). But you can override these templates if you use an other UI framework.

If you want to use the `date` editor, you will additionnaly need to load the Bootstrap Datepicker extension. A compatible version, untouched, is provided along with `forms.js`.

### Configuration ###

#### Explicit model schema ####

Forms are generate automatically based on your schema declaration:

    var MyModel = Backbone.Model.extend({
        // Put whatever you need at instance level
    }, {
        // Declare schema and verbose name at model level
        schema: {
            id: {title: 'ID', type: 'Number', editable: false},
            name: {title: 'Name', type: 'Text', required: true}
        },
        verboseName: 'My model'
    });

The `schema` object provides configuration options for each model attributes. The `verboseName` may be used as a form title.

Note that the field order in the displayed form will be the same as in the schema declaration.

#### Get the data back ####

The form works with an underlying model instance: it will read initial values and write user inputs into that instance. You have two options for binding a form and its instance:

    // Option 1: pass it an existing instance
    var instance = new MyModel(),
        form = new NS.UI.Form({initialData: instance});

    // Option 2: pass it a model class and let the model create an instance itself
    var form = new NS.UI.Form({model: MyModel});

Finally you will have to code what to do when the user validates the form. To do so, write an handler for the `submit:valid` event. This handler will be passed the updated instance as an argument:

    form.on('submit:valid', function(instance) {
        ...
    });

#### Controlling which fields are displayed ####

The optional `fields` arguments let you choose which fields are included in the form. Pass it an array containing either of the following:
- field name as a string (the key of the `schema` object),
- an object exposing `title` and `fields` properties.

The later option will produce a fieldset using `title` as the legend. The `fields` property is a nested field list.

### Supported editors ###

The library include several editors which fall into two categories: simple editors allow the user to input a single values whereas composite editors represent complex, multi-value structure and delegates input to sub-editors.

#### Text ####

TODO

#### Number ####

TODO

#### Boolean ####

TODO

#### Date ####

Set a field type as `Date` and form engine will display a calendar widget to pick a date. The default date format is `'dd/mm/yyyy'` but you can override it by passing a `format` option in your schema declaration.

Your data (either `initialData` or a model attribute) must be typed as a regular JavaScript `Date` object. The editor will also parse the user input and return a regular `Date` object.

NB: The Datepicker Bootstrap extension is required for this to work.

#### Select ####

Set a field's type as `Select` and it will be presented using a classic HTML dropdown list. You can feed the select's options with:
- a Collection: each collection item will result in an option with item.id as the value and item.toString() as the label;
- an array of plain objects, each object must expose two properties: `val` (the value) and `label` (the label);
- an array of literal values (number, string, ...) which will be used as the value and the label.
You can actually mix the literal values and plain objects in the same array.

Use the `multiple` option to enable or disable multiple selection.

Example:

    schema: {
        ...,
        habitat: {title: 'Habitat', type: 'Select', options: ['Wall', 'Hedge', {val: 'Path', label:'Path or track'}]},
        ...
    }


#### NestedModel ####

Use the `NestedModel` type for complex attributes.

Example:

    schema: {
        ...,
        address: {title: 'Address', type: 'NestedModel', model: Address},
        ...
    }

You have three options to describe your complex attribute:
- pass a Backbone Model class in the `model` argument, the validation process will return a new instance of that model;
- pass a Backbone Model instance in the `initialData` argument, the validation process will return that instance modified;
- pass a raw schema definition object in the `schema` argument and optionnally a hash of values in the `initialData` argument, the validation process will return a hash for user inputs.

#### List ####

TODO

#### MultiSchema ####

TODO

### Rendering ###

Editors can be rendered in stacked or inline mode.

The stacked mode is the default one. Labels are on the left and input fields on the right. Each editor is displayed on its own line.

Inline mode may be used to create editable list of objects. The form is displayed as a grid, labels appear as column headers and one row is created for each instance. Input fields are chained horizontally as row cells.

### Validation ###

Validation is triggered whenever a `blur` event occurs on a field. The
validation process runs every declared validator for each field and intercepts
`ValidationError` if any.

A `validator` is an object with a `validation()` method which takes the value
to validate as an arguments. This method either returns the validated (and
optionnaly cleaned) value or raises an exception.

As an example, this validator is enabled when you mark a field as `required`:

	NS.UI.Form.validators.Required = function() {
		this.msg = 'Blank value not allowed here';
		this.validate = function(value) {
			if (typeof value === 'undefined')
                throw new ValidationError(this.msg);
            return value;
		};
	};

You can easily implement your custom validators. For example, if you want to
ensure a number is positive, you would declare a validator like so:

    // Place this code after Form library has loaded and before you initialize your form
	NS.UI.Form.validators.Positive = function() {
		this.validate = function(value) {
			if (value < 0)
                throw new NS.UI.Form.ValidationError('Positive value is expected here');
            return value;
		};
	};

Then, you can enable this validator in your `schema` declaration:

    Person = Backbone.Model.extend({
        ...
    }, {
        schema: {
            age: {title: 'Age', type: 'Number', validators: ['Positive']},
            ...
        }
    });

### Not so FAQ ###

#### How do I set a default value for a field? ####

There is several alternative ways to achieve this:

1. If your form is initialized with a `model` instance, you can use Backbone's [`defaults`](http://backbonejs.org/#Model-defaults) option.
2. If you prefer setting a default value per instance, you can simply fill it in the corresponding attribute on the instance before passing it to the form's `initialData`.
3. You may also declare a `defaultValue` option in the `schema` definition.

Examples:

    // Solution 1
    MyModel = Backbone.Model.extend({
        defaults: {
            name: 'A default name...'
        }
    }, {
        schema: {
            name: {title: 'Name', type: 'Text'}
        },
        verboseName: 'My model'
    });

    // Solution 2
    if (! instance.has('name'))
        instance.set({name: 'A default name...'});
    form = new NS.UI.Form({initialData: instance});

    // Solution 3
    MyModel = Backbone.Model.extend({}, {
        schema: {
            name: {title: 'Name', type: 'Text', defaultValue: 'A default name...'}
        },
        verboseName: 'My model'
    });

#### How do I customize editors' template? ####

Every editor has a `templateSrc` static property with holds two template strings, one for stacked mode and another for inline mode. You are free to change these strings in order to customize UI. Use the original template string as a model.

In the following example, we leverage this feature to enable HTML5's `<input type="number">` for Number editor in stacked mode:

    // Place this code after Form library has loaded and before you initialize your form
    NS.UI.Form.editors.Number.templateSrc.stacked = 
        '<div class="control-group">' +
        '    <label class="control-label" for="<%- data.id %>"><% if (data.required) { %><b>*</b><% } %> <%- data.label %></label>' +
        '    <div class="controls">' +
        '        <input type="number" id="<%- data.id %>" name="<%- data.name %>" value="<%- data.initialData %>" />' +
        '        <div class="help-inline"></div>' +
        '        <div class="help-block"><% if (data.helpText) { %><%- data.helpText %><% } %></div>' +
        '    </div>' +
        '</div>';

Note that all properties returned by `serialize()` are available through an object named `data`. This is because we use Underscore's `template()` with the `variable` option.

Of course, you can (and you are encouraged to) use your own template loading method. If your template loading mechanism is asynchronous, you will need to ensure that loading is over before initializing the form (hint: jQuery's [deferred objects](http://api.jquery.com/jQuery.Deferred/)).

#### How do I write a custom editor? ####

TODO

## Contributors ##

Gilles Bassière, [Natural Solutions](http://natural-solutions.eu/)
