{{#template name="blaze-step08"}}

# Storing temporary UI state in Session

In this step, we'll add a client-side data filtering feature to our app, so that users can check a box to only see incomplete tasks. We're going to learn how to use `Session` to store temporary reactive state on the client.

First, we need to add a checkbox to our HTML:

```html
<!-- add the checkbox to <header> right below the h1 -->
<label class="hide-completed">
  <input type="checkbox" checked="{{dstache}}hideCompleted}}" />
  Hide Completed Tasks
</label>
```

Then, we need an event handler to update a `Session` variable when the checkbox
is checked or unchecked. `Session` is a convenient place to store temporary UI
state, and can be used in helpers just like a collection.

```js
// Add to Template.body.events
"change .hide-completed input": function (event) {
  Session.set("hideCompleted", event.target.checked);
}
```

Now, we need to update `Template.body.helpers`. The code below has a new if
block to filter the tasks if the checkbox is checked, and a helper to make sure
the checkbox represents the state of our Session variable.

```js
// Replace the existing Template.body.helpers
Template.body.helpers({
  tasks: function () {
    if (Session.get("hideCompleted")) {
      // If hide completed is checked, filter tasks
      return Tasks.find({checked: {$ne: true}}, {sort: {createdAt: -1}});
    } else {
      // Otherwise, return all of the tasks
      return Tasks.find({}, {sort: {createdAt: -1}});
    }
  },
  hideCompleted: function () {
    return Session.get("hideCompleted");
  }
});
```

Now if you check the box, the task list will only show tasks that haven't been completed.

### Session is a reactive data store for the client

Until now, we have stored all of our state in collections, and the view updated automatically when we modified the data inside these collections. This is because Meteor.Collection is recognized by Meteor as a _reactive data source_, meaning Meteor knows when the data inside has changed. `Session` is the same way, but is not synced with the server like collections are. This makes `Session` a convenient place to store temporary UI state like the checkbox above. Just like with collections, we don't have to write any extra code for the template to update when the `Session` variable changes &mdash; just calling `Session.get(...)` inside the helper is enough.

### One more feature: Showing a count of incomplete tasks

Now that we have written a query that filters out completed tasks, we can use the same query to display a count of the tasks that haven't been checked off. To do this we need to add a helper and change one line of the HTML.

```js
// Add to Template.body.helpers
incompleteCount: function () {
  return Tasks.find({checked: {$ne: true}}).count();
}
```

```html
<!-- display the count at the end of the <h1> tag -->
<h1>Todo List ({{dstache}}incompleteCount}})</h1>
```

{{/template}}
