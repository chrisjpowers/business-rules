# Business Rules for JavaScript

When you're working in a business environment (especially an "enterprisy" environment),
business rules make the world go 'round. Business-level decision makers need to be able
to alter application behavior and logic with minimal technical invasiveness.

Usually, business rule engines come packaged as $100K` Java installations and a gaggle
of business consultants. Let's just call this a lighter-weight solution.

The goals of this library are to:

1. Give JavaScript developers drop-in jQuery UI widgets for building business rule interfaces.
2. Give developers a rule engine that can be ported to any server-side language for running business rules built with the UI tools.

## Examples

Checkout the examples directory if you just want to dive in, it demonstrates all the features
pretty well!

## `$.fn.conditionsBuilder()`

The `$.fn.conditionsBuilder()` method has two forms, the first being:

  $("#myDiv").conditionsBuilder({fields: [...], data: {...});

The first form creates a `ConditionsBuilder` object for the given DOM element with the passed fields and data.
The `fields` param is an array of objects that define the factors that can be used in conditional statements.
Each has a `label`, `value` and an array of `operators`. It may also have an `options` array of objects
with `label` and `value`.

Each `operator` is an object with `label`, `value` and `fieldType`. The `fieldType` can be:

* `"none"` - The operator does not require further data entry (ie `"present"`, `"blank"`).
* `"text"` - User will be presented with an input of `type=text`.
* `"textarea"` - User will be presented with a textarea.
* `"select"` - User will be presented with a select dropdown populated with the parent field's `options` array.

The `data` param is an object that will be used to initially populate the UI (ie if business rules have already
been created and the user wants to edit them). If the `data` option is not passed, the UI will be
generated without any initial conditions.

The object passed as `data` should be a "conditional object", meaning it has a single key of `all` or `any`
and a value of an array of nodes. These nodes can either be rule objects or nested conditional objects.

A rule object has `field`, `operator` and `value` strings. The `field` should match a field's `value` property,
the `operator` should match an operator's `value` property, and the `value` is an arbitrary string value
entered by the user in the UI.

Once the UI has been built by the `ConditionsBuilder` and the user has entered information, the data can
be retrieved by using the second form of the `conditionsBuilder` method:

  var data = $("#myDiv").conditionsBuilder("data");

This will serialize the entered conditionals into a data object. This object can be persisted and then later
used to create a new `ConditionsBuilder` for editing. This data object will also be used to instantiate
a `BusinessRules.RuleEngine` object for running the conditional logic.


## `$.fn.actionsBuilder()`

The `$.fn.actionsBuilder` has an identical API to `$.fn.conditionsBuilder`, but it uses a different data structure.
The `fields` property should be an array of action objects. Each action object has a `label` and `value`.
An action object may have a `fields` property that is an array of action objects, allowing for nested action data. 
All action objects that are not "top level" should also have a `fieldType` of `text`, `textarea` or `select`.

Here's an example of what a "Send Email" action could look like:

  $("#myDiv").actionsBuilder({fields: [
    {label: "Send Email", value: "sendEmail", fields: [
      {label: "To", value: "to", fieldType: "text"},
      {label: "CC", value: "cc", fieldType: "text"},
      {label: "BCC", value: "bcc", fieldType: "text"},
      {label: "Subject", value: "subject", fieldType: "text"},
      {label: "Body", value: "body", fieldType: "textarea"}
    ]}
  ]});

Action objects with a `fieldType` of `select` should not have a `fields` property -- rather they have an `options`
property with a `label` and `value` for each option. That option object, however, can have a `fields` property.
This allows you to specify nested fields that will only be displayed if the given option has been selected.

Building on the last example, this allows the user to specify an email template, or use a custom Subject and Body:

  $("#myDiv").actionsBuilder({fields: [
    {label: "Send Email", value: "sendEmail", fields: [
      {label: "To", value: "to", fieldType: "text"},
      {label: "CC", value: "cc", fieldType: "text"},
      {label: "BCC", value: "bcc", fieldType: "text"},
      {label: "Email Template", value: "template", fieldType: "select", options: [
        {label: "Welcome Email", value: "welcomeEmail"},
        {label: "Followup Email", value: "followupEmail"},
        {label: "Custom Email", value: "customEmail", fields: [
          {label: "Subject", value: "subject", fieldType: "text"},
          {label: "Body", value: "body", fieldType: "textarea"}
        ]}
      ]}
    ]}
  ]});

In this example, the "Subject" and "Body" fields will only be displayed if the user has selected the "Custom Email"
template option.

To get the data out of the UI, run the `actionsBuilder` method with `"data"`:

  var data = $("#myDiv").actionsBuilder("data");

Each action data object has a `name` that matches the corresponding field's `value`, and a `value` property with the
user-entered value. It may also have a `fields` array of nested action data objects, which correspond to the nested
field structure of the builder.
