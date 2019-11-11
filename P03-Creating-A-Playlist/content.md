---
title: "Create Route: Saving a New Resource"
slug: creating-a-playlist
---

Remember we are doing Resourceful and RESTful architecture to our MVC structured app. We've made some views in the `templates` folder, instantiated a model, and put our controller logic (our routes) into the `app.py` file. As our model and controller logic grows we can move them into other files. For this app it is so simple, that we'll just leave all that logic in the `app.py` file.

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
<!-- New link here -->
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
> Add the following to `app.py` in place of your existing Flask import:
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

# Adding a New Attribute to our Playlist

The new action will be our form for making a new playlist. For now playlists just have two attributes: `title` and `description`. We can always add more! Let's add an attribute called `video_ids` so that we can use the ids to construct YouTube URLs in order to save links to the videos themselves.

First, how do we find a video ID for a YouTube video?

> [action]
>
> Watch this quick video to see where to find them:
>
> ![ms-video](assets/youtube_id.mov)

First let's add what the user sees - the `playlists_new.html` form input field.

> [action]
>
> Create the `templates/playlists_new.html` file:
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
        <!-- VIDEO IDS -->
        <p>
            <label for='playlist-video-ids'>Videos</label><br>
            <p>Add the ID of the videos you want to include in your playlist. Separate with a newline.</p>
            <textarea id='playlist-video-ids' name='video_ids' rows='10'></textarea>
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

# URL Helper Function

We'll have to change the `playlists_submit` route as well so that it takes in our new data field. But there's a bit of a problem: right now we're only taking in IDs, and the route doesn't know how to construct a URL. But we do!!

> [action]
>
> at the top of `app.py` (after we've defined what `app` is), write a helper function called `video_url_creator` that takes in a list of IDs as input and outputs a list of YouTube embed links:
>
```py
def video_url_creator(id_lst):
    videos = []
    for vid_id in id_lst:
        # We know that embedded YouTube videos always have this format
        video = 'https://youtube.com/embed/' + vid_id
        videos.append(video)
    return videos
```

<!-- -->


> [challenge]
>
> Is there a way to optimize the above function? How would you improve it?

Alright, now that we can create YouTube links, let's finish up that Submit route!

# Finish the Submit Route

We'll need to use our helper function to create a list of videos from our video IDs we collected in the form. Once we do that, we can create our playlist object with a proper `videos` field. We'll also create a `video_ids` field to store our ids.

> [action]
>
> Create the `submit` route in `app.py`:
>
```python
# Note the methods parameter that explicitly tells the route that this is a POST
@app.route('/playlists', methods=['POST'])
def playlists_submit():
    """Submit a new playlist."""
    # Grab the video IDs and make a list out of them
    video_ids = request.form.get('video_ids').split()
    # call our helper function to create the list of links
    videos = video_url_creator(video_ids)
    playlist = {
        'title': request.form.get('title'),
        'description': request.form.get('description'),
        'videos': videos,
        # storing the IDs in here for now, we'll use them later!
        'video_ids': video_ids
    }
    playlists.insert_one(playlist)
    return redirect(url_for('playlists_index'))
```

You can create any attributes you like for your model and use various data types, such as strings, numbers, and dates.

Can you resubmit the form? What happens now?

If all went well, you should have created a playlist and can view it from the `playlists_index` page! Great work! Let's keep building this out by diving into showing individual playlists beyond the `playlists_index`page.

> [info]
>
> Confused why we added a `video_ids` field to the playlist? We'll come back to that in a couple chapters.

<!-- -->

> [challenge]
>
> 1. Instead of having one textarea for all video IDs and separating by newline, create one textarea for each video ID. Start with one textarea, and allow users to be able to add one additional textarea for each ID they want to add
> 1. Add a way for users to delete textareas if they make a mistake, or no longer want to add that video to their playlist

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
