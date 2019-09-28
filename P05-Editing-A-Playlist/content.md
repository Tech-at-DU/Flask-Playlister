---
title: "Edit Route: Editing and Updating a Resource"
slug: editing-and-deleting-a-playlist
---

Now we're checking off the seven **Resourceful Routes**. But we're not done until all of them are complete.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |
| /playlists/:id     | GET       | show    |
| /playlists/:id/edit     | GET       | edit    |
| /playlists/:id     | PUT/PATCH | update  |
| /playlists/:id     | DELETE    | Destroy |

Normally in a site like YouTube, we would only want authors of playlists to have the permission to edit or delete a playlist. However, because we do not have authentication yet, we're just going to let anyone edit and delete playlists.

# Edit Link

We want people to be able to edit and update playlists, so let's again start from the user's perspective. Edit and Update are similar to New and Create. First we need a link to the edit route that renders the `playlists_edit` template, and then we submit that edit form to the update route which will redirect to the show action.

> [action]
>
> So let's make the edit link in `templates/playlists_show.html`:
>
```html
<!-- templates/playlists_show.html -->
{% extends 'base.html' %}
>
{% block content %}
<a href='/'>Back to Home</a>
>
<h1>{{ playlist.title }}</h1>
<h2>{{ playlist.description }}</h2>
{% for video in playlist.videos %}
    <iframe width='420' height='315' src='{{ video }}'></iframe>
{% endfor %}
>
<p><a href='/playlists/{{ playlist._id }}/edit'>Edit</a></p>
{% endblock %}
```

Ok, now if we click that edit link, we'll see that the route is not found. So let's make our edit action. The edit action is like the show action because we look up the `playlist` by its `_id` in the url parameter, but then we render the information in a template as editable form elements.

> [action]
>
> Add an edit route in `app.py`:
>
```python
# app.py
>
...
>
@app.route('/playlists/<playlist_id>/edit')
def playlists_edit(playlist_id):
    """Show the edit form for a playlist."""
    playlist = playlists.find_one({'_id': ObjectId(playlist_id)})
    return render_template('playlists_edit.html', playlist=playlist)
```

And of course we'll need that `playlists_edit` template. This template is a bit weird for three reasons:

1. **PUT vs. POST** - Although our update action will be expecting a PUT HTTP action, HTML forms cannot take an action attribute of `PUT`. Therefore, we'll make it a `POST` action instead.
1. **`value=''`** - we are using the `value` html attribute to pass in the values of the playlist we are trying to edit.
1. **`<textarea>{{}}</textarea>`** - the `<textarea>` HTML tag does not have a `value` attribute, so its contents must go between its open and close tags.

> [action]
>
> Add a `templates/playlists_edit.html` template:
>
```html
<!-- templates/playlists_edit.html -->
{% extends 'base.html' %}
>
{% block content %}
<form method='POST' action='/playlists/{{playlist._id}}'>
    <fieldset>
        <legend>Edit Playlist</legend>
        <!-- TITLE -->
        <p>
            <label for='playlist-title'>Title</label><br>
            <input id='playlist-title' type='text' name='title' value='{{ playlist.title }}'/>
        </p>
>
        <!-- DESCRIPTION -->
        <p>
            <label for='description'>Description</label><br>
            <input id='description' type='text' name='description' value='{{ playlist.description }}' />
        </p>
>
        <!-- VIDEO LINKS -->
        <p>
            <label for='playlist-videos'>Videos</label><br>
            <p>Add videos in the form of 'https://youtube.com/embed/KEY'. Separate with a newline.</p>
            <textarea id='playlist-videos' name='videos' rows='10' />{{ "\n".join(playlist.videos) }}</textarea>
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

# Update Route

We can now add our `update` route:

> [action]
>
> Add the update route to `app.py`:
>
```python
# app.py
>
...
>
@app.route('/playlists/<playlist_id>', methods=['POST'])
def playlists_update(playlist_id):
    """Submit an edited playlist."""
    updated_playlist = {
        'title': request.form.get('title'),
        'description': request.form.get('description'),
        'videos': request.form.get('videos').split()
    }
    playlists.update_one(
        {'_id': ObjectId(playlist_id)},
        {'$set': updated_playlist})
    return redirect(url_for('playlists_show', playlist_id=playlist_id))
```

# DRY Code & Sub Templates

Did you notice that the code of our `playlists_new` and `playlists_edit` have a lot of similarities? Pretty much everything inside the `form` tag is the same. Let's use a **Partial Template** to pull that code out into its own template.

> [action]
>
> First make a folder called `partials` inside the `templates` folder. Now in that `partials` folder create the `playlists_form.html`.
>
```html
<!-- templates/partials/playlists_form.html -->
>
<fieldset>
<legend>{{ title }}</legend>
<!-- TITLE -->
<p>
    <label for='playlist-title'>Title</label><br>
    <input id='playlist-title' type='text' name='title' value='{{ playlist.title }}'/>
</p>
>
<!-- DESCRIPTION -->
<p>
  <label for='description'>Description</label><br>
  <input id='description' type='text' name='description' value='{{ playlist.description }}' />
</p>
>
<!-- VIDEO LINKS -->
<p>
  <label for='playlist-videos'>Videos</label><br>
  <p>Add videos in the form of 'https://youtube.com/embed/KEY'. Separate with a newline.</p>
  <textarea id='playlist-videos' name='videos' rows='10' />{{ "\n".join(playlist.videos) }}</textarea>
</p>
</fieldset>
>
<!-- BUTTON -->
<p>
    <button type='submit'>Save Playlist</button>
</p>
```

And now we can use this partial to replace that information in both our new and edit templates.

> [action]
>
> Update `templates/playlists_new.html` and `templates/playlists_edit.html` to use the partial:
>
```html
<!-- templates/playlists_new.html -->
{% extends 'base.html' %}
>
{% block content %}
<form method='POST' action='/playlists'>
    {% include 'partials/playlists_form.html' %}
</form>
{% endblock %}
```
>
```html
<!-- templates/playlists_edit.html -->
{% extends 'base.html' %}
>
{% block content %}
<form method='POST' action='/playlists/{{playlist._id}}'>
    {% include 'partials/playlists_form.html' %}
</form>
{% endblock %}
```

Finally, notice how we included a `{{ title }}` in `templates/partials/playlists_form.html`. We need to ensure that gets populated correctly based on whether a user is editing a playlist or creating a new one.

> [action]
>
> Update your `new` and `edit` routes in `app.py` to include the `title` parameter:
>
```python
# app.py
...
>
@app.route('/playlists/new')
def playlists_new():
    """Create a new playlist."""
    return render_template('playlists_new.html', playlist={}, title='New Playlist')
>
...
>
@app.route('/playlists/<playlist_id>/edit')
def playlists_edit(playlist_id):
    """Show the edit form for a playlist."""
    playlist = playlists.find_one({'_id': ObjectId(playlist_id)})
    return render_template('playlists_edit.html', playlist=playlist, title='Edit Playlist')
```

Triumph! DRY code. (Don't Repeat Yourself)

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Users can edit, update, and destroy playlists'
$ git push
```

# Stretch Challenge: "Cancel" buttons

> [challenge]
>
> Sometimes people might start making a resource and then want to cancel. Can you add a "Cancel" button next to the "Save Playlist" button? What will it do? Where will it link to?
