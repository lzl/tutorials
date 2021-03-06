{{#template name="blaze-step11"}}

# Filtering data with publish and subscribe

Now that we have moved all of our app's sensitive code into methods, we need to learn about the other half of Meteor's security story. Until now, we have worked assuming the entire database is present on the client, meaning if we call `Tasks.find()` we will get every task in the collection. That's not good if users of our application want to store privacy-sensitive data. We need a way of controlling which data Meteor sends to the client-side database.

Just like with `insecure` in the last step, all new Meteor apps start with the `autopublish` package. Let's remove it and see what happens:

```bash
meteor remove autopublish
```

When the app refreshes, the task list will be empty. Without the `autopublish` package, we will have to specify explicitly what the server sends to the client. The functions in Meteor that do this are `Meteor.publish` and `Meteor.subscribe`.

Let's add them now.

```js
// At the bottom of simple-todos.js
if (Meteor.isServer) {
  Meteor.publish("tasks", function () {
    return Tasks.find();
  });
}
```

```js
// At the top of our client code
Meteor.subscribe("tasks");
```

Once you have added this code, all of the tasks will reappear.

Calling `Meteor.publish` on the server registers a _publication_ named `"tasks"`. When `Meteor.subscribe` is called on the client with the publication name, the client _subscribes_ to all the data from that publication, which in this case is all of the tasks in the database. To truly see the power of the publish/subscribe model, let's implement a feature that allows users to mark tasks as "private" so that no other users can see them.

### Implementing private tasks

First, let's add another property to tasks called "private" and a button for users to mark a task as private. This button should only show up for the owner of a task. It will display the current state of the item.

```html
<!-- add right below the code for the checkbox in the task template -->
{{dstache}}#if isOwner}}
  <button class="toggle-private">
    {{dstache}}#if private}}
      Private
    {{dstache}}else}}
      Public
    {{dstache}}/if}}
  </button>
{{dstache}}/if}}

<!-- modify the li tag to have the private class if the item is private -->
<li class="{{dstache}}#if checked}}checked{{dstache}}/if}} {{dstache}}#if private}}private{{dstache}}/if}}">
```

We need to modify our JavaScript code in three places:

```js
// Define a helper to check if the current user is the task owner
Template.task.helpers({
  isOwner: function () {
    return this.owner === Meteor.userId();
  }
});

// Add an event for the new button to Template.task.events
"click .toggle-private": function () {
  Meteor.call("setPrivate", this._id, ! this.private);
}

// Add a method to Meteor.methods called setPrivate
setPrivate: function (taskId, setToPrivate) {
  var task = Tasks.findOne(taskId);

  // Make sure only the task owner can make a task private
  if (task.owner !== Meteor.userId()) {
    throw new Meteor.Error("not-authorized");
  }

  Tasks.update(taskId, { $set: { private: setToPrivate } });
}
```

{{> step11SelectivelyPublish}}

{{/template}}
