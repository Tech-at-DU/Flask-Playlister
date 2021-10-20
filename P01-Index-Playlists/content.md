# See All Playlists

Following our **User Stories** we are going to define a single **Resource** in this app called a `Playlist` so that we have something to CRUD:

1. Users can view all playlists (index)
1. Users can create a playlist (new/create)
1. Users can view one playlist (show)
1. Users can delete a playlist (destroy)
1. Users can edit a playlist (edit/update)
1. Users can comment on playlists (comments#create)
1. Users can delete comments on playlists (comments#destroy)

A **Resource** is an abstract object that we use to organize data, code, and the features of our app. For example, a `User` resource we can keep track of logging in and out, email and passwords, and people's birthdays. In a blog, we might have an `Article` or `Post` resource where we would track the titles and bodies of articles and keep track of the code for publishing and sharing them.

Resources can also be related to each other. For example, articles may have comments; a building resource might have floors, and the floors may have units; a user might have friends (like in Facebook).

All resources have a few actions in common that are called **Resourceful Routes**.

Review this table of routes, as we'll be referencing it frequently in this tutorial, and you should be familiar with what each route does:

| URL              | HTTP Verb | Action  | What it Does |
|------------------|-----------|---------|---------------|
| /playlists          | GET       | index   | See all playlists |
| /playlists/new      | GET       | new     | See new playlist form |
| /playlists          | POST      | create  | Create a new playlist |
| /playlists/:id      | GET       | show    | See one playlist |
| /playlists/:id/edit | GET       | edit    | See an edit playlist form |
| /playlists/:id      | PATCH/PUT | update  | Update a playlist |
| /playlists/:id      | DELETE    | destroy | Delete a playlist |

The table above shows the resources, routes, and actions that might be used on a playlist creating site like Playlister.

We're going to start with the index action and then show, and then we'll look at the create, edit, update, and delete.

# Adding a /playlists Route

Let's make a route to `/playlists` for the index action where we can see all the playlists that we've created. Eventually this will be on our root route, but we can start making it its own separate path.

Add the following to `app.py` after the '/' route:

```python
# OUR MOCK ARRAY OF PROJECTS
playlists = [
    { 'title': 'Cat Videos', 'description': 'Cats acting weird' },
    { 'title': '80\'s Music', 'description': 'Don\'t stop believing!' }
]
>
@app.route('/playlists')
def playlists_index():
    """Show all playlists."""
    return render_template('playlists_index.html', playlists=playlists)
```

Notice how we are making a mock array of playlists, and sending that in as an object into the template `playlists=playlists`. So the variable `playlists` will be available in our template.

## Stretch Challenge

Throughout these tutorials, you'll see sections designated as **stretch challenges**. Stretch Challenges are provided for those who want to dive deeper on a topic, or get more practice. They are not required to complete the tutorial, but we encourage you to try them! More practice is always beneficial!

Can you make changes to your `playlists` array in `app.py` and see it reflected in your `playlists_index` template?

# Errors are Your Friends!

If you refresh `localhost:5000/playlists` right now what do you see? An error! **That's ok!** Errors are our friends. What does the error say? I bet it says something like "I can't find the template 'playlists_index'". That makes sense because we haven't made it yet! It's ok to complete coding tasks in an order that throws a predictable error.

> Errors often tell you the next step you need to take. In this case the error is telling you that the template does not exist. Let's make it!

So to fix this error, let's add the template `templates/playlists_index.html`. We're going to use the Jinja2 `{% for %}` iterator to loop over our array of playlists and display each one's title.

Create the `templates/playlists_index.html` template and add the following to it:

```html
<!-- templates/playlists_index.html -->
{% extends 'base.html' %}
>
{% block content %}
    <h1>Playlists</h1>
    {% for playlist in playlists %}
        <h2>{{ playlist.title }}</h2>
        <small>{{ playlist.description }}</small>
    {% endfor %}
{% endblock %}
```

If you refresh `localhost:5000/playlists` now what do you see?

<details>
<summary>Solution</summary>
<br>
You should see the mock playlists we wrote into code. Can you add to them or change them?
</details>

# Setting the Root Route - '/'

_For this next update, we want you to try it on your own!_

Let's update the `/playlists` route to be our root route. Just change the path from `/playlists` to `/` and delete or comment out the hello world root route we made before.

Now navigate to `localhost:5000`. Do you see the mock playlists displayed?

# Now Commit

```bash
$ git add .
$ git commit -m 'Users can see all playlists'
$ git push
```

# Next

Click [here](../P02-Adding-MongoDB/content.md) to move onto the next section about adding MongoDB.
