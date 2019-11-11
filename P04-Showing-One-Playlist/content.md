---
title: "Show Route: See One Resource"
slug: showing-one-playlist
---

We are building out all the **Resourceful Routes** for our `Playlist` resource.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |
| /playlists/:id     | GET       | show    |

We've already completed the index, new, and create actions. Now you will add a show action that will display a single resource via its id.

Now let's setup the **show** action so we give each single playlist its own page and unique url path.

# Show One Playlist

Remember always start with what the user will see and do. To create the show action, you will want to start by making a link to the playlist from our index action template. Your route has to follow the `/playlists/:id` structure.

MongoDB automatically creates an `_id` attribute on anything you save. So we can use that `_id` attribute for our `:id` in the route. This is called the **Url or Request Parameter** and we access it in Flask using a parameter inside of the controller route.

> [action]
>
> Update `templates/playlists_index.html` to the following:
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
    # Edit this line to include a link
    <h2><a href='/playlists/{{ playlist._id }}'>{{ playlist.title }}</a></h2>
    <small>{{ playlist.description }}</small>
{% endfor %}
{% endblock %}
```

What happens if you click on one of those links? A friendly error! Let's do what it says and make the route.

> [action]
>
> Add the `/playlists/<playlist_id>` route to `app.py`:
>
```python
@app.route('/playlists/<playlist_id>')
def playlists_show(playlist_id):
    """Show a single playlist."""
    return f'My ID is {playlist_id}'
```

Now what happens if you go to that route?

# Get a Single Playlist from MongoDB

Ok, time to add a template with an actual `playlist` object! To get the playlist from our MongoDB database, we can use the `find_one()` method. We will have to convert the string value of its `_id` to an `ObjectId`.

> [action]
>
> Create the `/playlists/<playlist_id>` route in `app.py`:
>
```python
# Add this with the rest of your import statements
from bson.objectid import ObjectId
>
...
>
@app.route('/playlists/<playlist_id>')
def playlists_show(playlist_id):
    """Show a single playlist."""
    playlist = playlists.find_one({'_id': ObjectId(playlist_id)})
    return render_template('playlists_show.html', playlist=playlist)
```

Now if we go to the route, we'll see the error that no template `playlists_show` is found. Great! Let's make it.

> [action]
>
> Create `templates/playlists_show.html`:
>
```html
<!-- templates/playlists_show.html -->
{% extends 'base.html' %}
>
{% block content %}
<h1>{{ playlist.title }}</h1>
<h2>{{ playlist.description }}</h2>
{% for video in playlist.videos %}
    <iframe width='420' height='315' src='{{ video }}'></iframe>
{% endfor %}
{% endblock %}
```

Now what do you see? All the links to playlists should work now!

# Add a Back Link

This is good, the show action is working, but there is a bit of a problem. Once you are on the show action page, you can't get back home.

> [action]
>
> Let's fix that by putting in a "Back" link in `templates/playlists_show.html`:
>
```html
<!-- templates/playlists_show.html -->
{% extends 'base.html' %}
>
{% block content %}
<!--Add the below link to go back -->
<a href='/'>Back to Home</a>
>
<h1>{{ playlist.title }}</h1>
<h5>{{ playlist.description }}</h5>
{% for video in playlist.videos %}
    <iframe width='420' height='315' src='{{ video }}'></iframe>
{% endfor %}
{% endblock %}
```

That's better. What else could we do now that we have this show route?

# Update the Create Action's Redirect

It makes sense from the user's perspective that after we create a new playlist, we should be automatically redirected to it, no?

> [action]
>
> Change the create route to redirect to the show path in `app.py`:
>
```python
@app.route('/playlists', methods=['POST'])
def playlists_submit():
    """Submit a new playlist."""
    video_ids = request.form.get('videos').split()
    videos = video_url_creator(video_ids)
    playlist = {
        'title': request.form.get('title'),
        'description': request.form.get('description'),
        'videos': videos,
        'video_ids': video_ids
    }
    playlist_id = playlists.insert_one(playlist).inserted_id
    # The below line is the one that changed!
    return redirect(url_for('playlists_show', playlist_id=playlist_id))
```

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Users can see single playlists'
$ git push
```

Now our user experience is getting very smooth, and our code is getting more and more complete. Onward!
