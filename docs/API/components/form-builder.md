# Form JS APIs

The following JS APIs are available in a screen once a **Form** component is dropped into the screen.

## Edit a data source entry

To use the **Form** component for editing a data source entry, provide a `dataSourceEntryId` query parameter to the page with a valid data source entry ID.

```
?dataSourceEntryId=123
```

## Retrieve an instance

Since you can have many forms in a screen, we provide a handy function to grab a specific instance by its form name or the first one available in the page when no input parameter is given.

### `Fliplet.FormBuilder.get()`

Retrieves the first or a specific form instance.

```js
// Gets the first form instance
Fliplet.FormBuilder.get()
  .then(function (form) {
    // Use form to perform various actions
  });

// Gets the first form instance named 'foo'
Fliplet.FormBuilder.get('foo')
  .then(function (form) {
    // Use form to perform various actions
  });
```

The `form` instance variable above makes available the following instance methods.

## Instance methods

### `form.load(Function)`

Allows to overwrite all form field values with new data. Useful if you manually want to populate the form based on your data.

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.load(function () {
    return {
      Name: 'Nick',
      Email: 'nick@example.org'
    };
  });
});
```

You can also return a `Promise` if you're loading the data asynchronously. In the following example we are populating a form from an entry in a Fliplet data source:

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.load(function () {
    return Fliplet.DataSources.connect(123).then(function (connection) {
      return connection.findById(456);
    });
  });
});
```

For more details, check the JS API documentation on the `Fliplet.DataSources` namespace.

<strong>Retrieve information of signed in user:</strong>

`form.load` can also be used in conjunction with `Fliplet.Session` to populate a form with the logged user's data:

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.load(function () {
    return Fliplet.Session.passport().data().then(function (response) {
      // response.user.id
      // response.user.email
      // response.user.firstName
      // response.user.lastName

      // Simply return the user when your fields have the same name as the user's columns
      return response.user;

      // Otherwise, you can do some basic mapping:
      return {
        'Email address': response.user.email
      };
    });
  });
});
```

---

### `form.instance`

Property that references a `Vue` object with more advanced properties of the form. For example, you can loop all form fields using the `instance.fields` array:

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.instance.fields.forEach(function (field) {
    // field is an object with "type", "name" and "value"
  });
});
```

#### Programmatically submit a form

You can submit a form programmatically by calling the `onSubmit()` method of the **Vue** instance available via the `instance` attribute of the form:

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.instance.onSubmit();
});
```

#### Programmatically clear (reset) a form

You can clear (reset) a form programmatically by calling the `reset()` method of the **Vue** instance available via the `instance` attribute of the form:

```js
Fliplet.FormBuilder.get().then(function (form) {
  form.instance.reset();
});
```

---

### form.fields.get()

Get the form fields

```
var fields = form.fields.get();
```

### `form.fields.set()`

Sets the form fields programmatically, e.g.:

```js
form.fields.set([
  {
    "_type": "flInput",
    "name": "name",
    "label": "Name"
  },
  {
    "_type": "flEmail",
    "name": "email",
    "label": "Email address"
  },
  {
    "_type": "flSelect",
    "name": "enquiryType",
    "label": "What is your enquiry about?",
    "options": [{ "id": "Support" }, { "id": "Feedback" }]
  },
  {
    "_type": "flCheckbox",
    "name": "How would you like us to contact you",
    "label": "How would you like us to contact you",
    "options": [{ "label": "By phone" }, { "label": "By email" }]
  },
  {
    "_type": "flRadio",
    "name": "What's the best time to contact you",
    "label": "What's the best time to contact you",
    "options": [{ "label": "In the morning" }, { "label": "In the afternoon" }]
  },
  {
    "_type": "flTextarea",
    "name": "message",
    "label": "How can we help you today?"
  },
  {
    _type: 'flParagraph',
    name: 'text1',
    content: 'Text of the paragraph'
  },
  {
    "_type": "flButtons",
    "name": "buttons"
  }
]);
```

---

### `form.field(String)`

Retrieves a form field by its name (not label) as defined in Fliplet Studio in the form settings.

```js
Fliplet.FormBuilder.get().then(function (form) {
  // Get the field with name 'foo'
  var field = form.field('foo');
});
```

## Field methods

### `field().get()`

Gets the value of a form field.

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // gets the input value
    var value = form.field('foo').get();
  });
```

---

### `field().set()`

Sets the value of a form field to one of the following:

  - a **literal value** (e.g. a `String`, a `Number`, a `Boolean` or an `Object`)
  - a value from the **user's profile**
  - a value from the **device shared storage**
  - a value from the **current app's private storage**
  - a value from a **query parameter**
  - a value as **result of a function** (optionally returning a promise when asynchronous)

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // Sets the value to a literal value
    form.field('field-1').set('foo');

    // Sets the value to the current user's email
    form.field('field-1').set({ source: 'profile', key: 'email' });

    // Sets the value with the content of the "foo" query parameter
    form.field('field-1').set({ source: 'query', key: 'foo' });

    // Sets the value from the key named "foo" in the device shared storage
    form.field('field-1').set({ source: 'storage', key: 'foo' });

    // Sets the value from the key named "foo" in the device app's private storage
    form.field('field-1').set({ source: 'appStorage', key: 'foo' });

    // Sets the value as a result of a function
    form.field('field-1').set(function () {
      return 'foo' + 'bar';
    });

    // Sets the value as a result of an asynchronous method. In this example,
    // we're populating the field with the full name of a user in a data source
    form.field('field-1').set(function () {
      return Fliplet.DataSources.connect(123).then(function (connection) {
        return connection.findOne({ where: { email: 'foo@example.org' } });
      }).then(function (entry) {
        return _.get(entry, 'data.fullName');
      });
    });

  });
```

---

### `field().val()`

Gets or sets the value of a form field.

<p class="warning"><strong>DEPRECATION NOTICE:</strong> uses of <code>val()</code> are considered deprecated for new apps. We do recommend using the new <code class="success">set()</code> and <code class="success">get()</code> methods described in the sections above.</p>

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // set the input value
    form.field('foo').val('bar');

    // gets the input value
    var value = form.field('foo').val();
  });
```

---

### `field().change(Function)`

Attaches an event listener to be fired whenever a field value changes.

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // registers a callback to be fired whenever the field value changes and also on init
    form.field('foo').change(function (val) {
      // val was changed
    });
  });
```

The callback will also be fired when the form initialized with a value. If you want to avoid this behavior, pass `false` as second parameter of the `change()` method:

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // registers a callback to be fired whenever the field value changes
    form.field('foo').change(function (val) {
      // val was changed
    }, false);
  });
```

---

### `field().toggle(Boolean)`

Shows or hides a field.

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    form.field('foo').toggle(); // Toggles field visibility
    form.field('bar').toggle(true); // Shows field
    form.field('baz').toggle(false); // Hides field
  });
```

<p class="quote"><strong>Note</strong>: toggling the field visibility will <strong>revert its value to the default value</strong> set in the form configuration. To disable this, pass a false boolean as second parameter of the toggle method at all times: <code>field.toggle(false, false)</code>.</p>

Shows or hides fields based on another field's value

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    // registers a callback to be fired whenever the field value changes
    form.field('foo').change(function (val) {
      // shows the field "bar" when the value of "foo" is greater than 10
      form.field('bar').toggle(val > 10);

      // shows the field "baz" when the value of "foo" is 50
      form.field('baz').toggle(val === 50);
    });
  });
```

---

### `field().options(Array)`

Programmatically sets the options of a dropdown or radio or checkbox field.

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    form.field('foo').options(['John', 'Nick', 'Tony']);
  });
```

You can also specify a different label from the value (`id`) as follows:

```js
Fliplet.FormBuilder.get()
  .then(function (form) {
    form.field('foo').options([
      { id: 'john', label: 'John Doe' },
      { id: 'nick', label: 'Nick Smith' }
    ]);
  });
```

## Hooks

### isFormInvalid

Runs when the form is found in valid after submission. The `invalidFields` parameter contains a collection of fields that are invalid.

```js
/* Suppress the notification for required fields */
Fliplet.Hooks.on('isFormInvalid', function(invalidFields) {
  return Promise.reject(); // With no parameter
});

/* Customize notification for required fields */
Fliplet.Hooks.on('isFormInvalid', function(invalidFields) {
  return Promise.reject('Go and fill in all required fields');
});

/* Ensure the notification for required fields appears */
Fliplet.Hooks.on('isFormInvalid', function(invalidFields) {
  return Promise.reject('');
});

/* Allow form submission to continue despite the form is invalid */
Fliplet.Hooks.on('isFormInvalid', function(invalidFields) {
  return Promise.resolve(); // or return nothing
});
```

### beforeFormSubmit

Capture data when the form is submitted. You can modify the data and also avoid data from being saved to a data source or continuing its flow.

```js
Fliplet.Hooks.on('beforeFormSubmit', function(data) {
  // Add your code here

  // e.g. mutate the data before it's sent
  data.foo = 'bar'

  // Return a promise if this callback should be async.
  // Reject the promise to stop the form from submitting the data or continuing.
  // If the rejection does not contain an error, no error will be displayed to the user.
});
```

If you reject the hook, the form will be stopped from submitting the data. Rejecting the promise with an error will display such error while a simply rejection will not display any message to the user:

  - `return Promise.reject("An error message")` this will display the error message
  - `return Promise.reject("")` this won't display anything to the user

---

### afterFormSubmit

Runs when a form has been submitted and has finished its processing.

```js
Fliplet.Hooks.on('afterFormSubmit', function(response) {
  // form data has been saved and submitted
  // response contains "formData" and "result"
});
```

---

### onFormSubmitError

Runs when a form could not be submitted because of an error.

```js
Fliplet.Hooks.on('onFormSubmitError', function(error) {
  // form encountered an error when saving or submitting
});
```

### beforeRichFieldInitialize

Runs when a **Rich text** field type is about to initialize. You can use this hook to make changes to the `config` initialization object which is then passed to **TinyMCE** after the hook runs.

```js
Fliplet.Hooks.on('beforeRichFieldInitialize', function (data) {
  // data.field
  // data.config

  // e.g. extend TinyMCE 4.8.1 initialization config properties
  data.config.toolbar = data.config.toolbar + ' || superscript subscript';
});
```

This is the default config options which we use for **TinyMCE** ([version 4.8.1](https://www.tiny.cloud/docs-4x/)) initialization:

```js
{
  theme: 'modern',
  mobile: {
    theme: 'mobile',
    plugins: [ 'autosave', 'lists', 'autolink' ],
    toolbar: [ 'bold', 'italic', 'underline', 'bullist', 'numlist', 'removeformat' ]
  },
  plugins: [
    'advlist autolink lists link directionality',
    'autoresize fullscreen code paste'
  ].join(' '),
  toolbar: [
    'bold italic underline',
    'alignleft aligncenter alignright alignjustify | bullist numlist outdent indent',
    'ltr rtl | link | removeformat code fullscreen'
  ].join(' | '),
  image_advtab: true,
  menubar: false,
  statusbar: false,
  inline: false,
  resize: false,
  autoresize_bottom_margin: 0,
  autofocus: false,
  branding: false
}
```

Please refer to the [TinyMCE 4](https://www.tiny.cloud/docs-4x/) documentation for a list of all available options.

---

## Events

### Reset (clear button pressed)

This event is fired when the clear button is pressed or the form is cleared programmatically.

```js
Fliplet.FormBuilder.on('reset', function () {

});
```

---

## Examples

### Updating data source entries

The following example shows how to:

1. populate a form from a data source entry
2. on submit, update the same data source entry

```js
var dataSourceId = 123;
var entryId = 456;

Fliplet.DataSources.connect(dataSourceId).then(function (connection) {
  // 1. Load the form in edit mode from a dataSource
  Fliplet.FormBuilder.get().then(function (form) {
    form.load(function () {
      return connection.findById(entryId);
    });
  });

  // 2. Bind a hook to update the data once the form is submitted
  Fliplet.Hooks.on('beforeFormSubmit', function(data) {
    return connection.update(entryId, data);
  });
});
```

[Back to API documentation](../../API-Documentation.md)
{: .buttons}
