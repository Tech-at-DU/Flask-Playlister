---
title: "Create Route: Saving a New Resource"
slug: creating-a-playlist
---

Remember we are doing Resourceful and RESTful architecture to our MVC structured app. We've made some views in the `views` folder, instantiated a model, and put our controller logic (our routes) into the `app.js` file. As our model and controller logic grows we can move them into other files. For this app it is so simple, that we'll just leave all that logic in the `app.js` file.

As for the **Resourceful Routes**, we've created just one: the index action for our `Playlist` resource.

Remember the previous table that showed a set of routes for Playlister?

Here it is again for playlist:

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /playlists          | GET       | index   |
| /playlists/new      | GET       | new     |
| /playlists          | POST      | create  |
| /playlists/:id      | GET       | show    |
| /playlists/:id/edit | GET       | edit    |
| /playlists/:id      | PATCH/PUT | update  |
| /playlists/:id      | DELETE    | destroy |


What do the routes look like for the app your are currently building?


| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |

Now we want to create two more for the new action, and the create action.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |


# Linking to New Playlist Route

First things first - what does the user see?

The user will have to click "New Playlist" to create a new playlist. So let's put a link into the `playlists_index` template.

> [action]
>
> Add the below link to `templates/playlists_index.html`:
>
```html
<!-- templates/playlists_index.html -->
{% extends 'base.html' %}
>
{% block content %}
<h1>Playlists</h1>
>
<a href='/playlists/new'>New Playlist</a>
>
{% for playlist in playlists %}
    <h2>{{ playlist.title }}</h2>
    <small>{{ playlist.description }}</small>
{% endfor %}
{% endblock %}
```

When you click this link you should see a friendly error reminding you that we haven't made the route for `/playlists/new` yet.

Remember that errors are not bad. They are like sign posts that tell you what to do next. So let's add our route to this new path.

# New Playlist Form

Now we have to make a route to the `/playlists/new` path, and have it render a `new_playlist` template.

> [action]
>
> Make a `/playlists/new` route in `app.py`:
>
```python
@app.route('/playlists/new')
def playlists_new():
    """Create a new playlist."""
    return render_template('playlists_new.html')
```

If we navigate our browsers to `/playlists/new` we'll get a friendly little error reminding us that we don't have a template called `playlists_new.html` yet. So let's put that into our `templates` folder.

> [action]
>
> Create `templates/playlists_new.html` to be the following:
>
```html
<!-- templates/playlists_new.html -->
{% extends 'base.html' %}
>
{% block content %}
<form method='POST' action='/playlists'>
    <fieldset>
    <legend>New Playlist</legend>
    <!-- TITLE -->
    <p>
        <label for='title'>Title</label><br>
        <input type='text' name='title' />
    </p>
    <p>
        <label for='description'>Description</label><br>
        <input type='text' name='description' />
    </p>
    </fieldset>
>
    <!-- BUTTON -->
    <p>
        <button type='submit'>Save Playlist</button>
    </p>
</form>
{% endblock %}
```

<!-- -->

> [info]
> Notice a few things about our form. The form tag has an HTML attribute called `action` that has a value equal to the path we want our form to submit its data to. `/playlists` is the route to our create action that we will add to our server in the next step.

Now if you navigate to `/playlists/new` you should see our new form looking great!

# Playlist Create Action

If you try to submit our form now, we see the form sends a **POST** request to the url `/playlists`, but we have not made a route that detects post requests to that path. So we see our friendly error:

```
The requested URL was not found on the server.
```

Our server has no route called `/playlists` that accepts a POST HTTP method. So let's make one!

First, you need to get ready to accept form data using Flask's built-in `request.form`. Let's go ahead and import that, as well as `redirect` and `url_for` which we'll use to redirect the user to another page on our site.

> [action]
>
> Add the following to `app.py` in place of your existing import:
>
```python
from flask import Flask, render_template, request, redirect, url_for
```

Now we can create a new route in `app.py` to accept a `POST` request.

> [action]
>
> Add a new route for `/playlists`.
>
```python
@app.route('/playlists', methods=['POST'])
def playlists_submit():
    """Submit a new playlist."""
    print(request.form.to_dict())
    return redirect(url_for('playlists_index'))
```

`request.form` gives us a dictionary containing the data that was sent in the page's HTML form. When you submit your form, you should see it log in in your terminal like this:

```
{ 'title': 'Creating a Playlist',
  'description': 'a sample playlist description' }
```

# Using Form Data to Create a Playlist

So now let's use that form data to save a new playlist to our MongoDB database using PyMongo. After we create the playlist, let's redirect to the root path to see our new playlist.

> [action]
>
> Write the `submit` route in `app.py`:
>
```python
@app.route('/playlists', methods=['POST'])
def playlists_submit():
    """Submit a new playlist."""
    playlist = {
        'title': request.form.get('title'),
        'description': request.form.get('description')
    }
    playlists.insert_one(playlist)
    return redirect(url_for('playlists_index'))
```

When you submit your form, what do you see?

# Adding a New Attribute to our Model

The new action will be our form for making a new playlist. For now playlists just have two attributes: `title` and `description`. We can always add more! Let's add an attribute called `video_urls` so that we can save links to the videos themselves.

First let's add what the user sees - the `playlists_new.html` form input field.

> [action]
>
> Add a section for description in `templates/playlists_new.html`:
>
```html
<!-- templates/playlists_new.html -->
{% extends 'base.html' %}
>
{% block content %}
<form method='POST' action='/playlists'>
    <fieldset>
        <legend>New Playlist</legend>
        <!-- TITLE -->
        <p>
            <label for='playlist-title'>Title</label><br>
            <input id='playlist-title' type='text' name='title' />
        </p>
>
        <!-- DESCRIPTION -->
        <p>
            <label for='description'>Description</label><br>
            <input id='description' type='text' name='description' />
        </p>
>
        <!-- VIDEO LINKS -->
        <p>
            <label for='playlist-videos'>Videos</label><br>
            <p>Add videos in the form of 'https://youtube.com/embed/KEY'. Separate with a newline.</p>
            <textarea id='playlist-videos' name='videos' rows='10'></textarea>
        </p>
    </fieldset>
>
    <!-- BUTTON -->
    <p>
        <button type='submit'>Save Playlist</button>
    </p>
</form>
{% endblock %}
```

We'll have to change the `playlists_submit` route as well so that it takes in our new data field.

> [action]
>
> Change the `submit` route in `app.py` to include a `videos` field:
>
```python
@app.route('/playlists', methods=['POST'])
def playlists_submit():
    """Submit a new playlist."""
    playlist = {
        'title': request.form.get('title'),
        'description': request.form.get('description'),
        'videos': request.form.get('videos').split()
    }
    playlists.insert_one(playlist)
    return redirect(url_for('playlists_index'))
```

You can create any attributes you like for your model and use various data types, such as strings, numbers, and dates.

Can you resubmit the form? What happens now?

If all went well, you should have created a playlist and can view it from the `playlists_index` page! Great work! Let's keep building this out by diving into showing individual playlists beyond the `playlists_index`page.


# Now Commit

```bash
$ git add .
$ git commit -m 'Users can create playlists'
$ git push
```

# Stretch Challenge: Adding a Rating Attribute

> [challenge]
>
> Can you add a rating attribute that is a number, and then use `<select>` and `<option>`'s elements to have a dropdown for users to select a `0-5` stars rating? You might have to google for an example to see how this works.
