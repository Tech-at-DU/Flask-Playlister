---
title: "Adding a Route for Comments"
slug: adding-route-for-comments
---

Now it's time to add a route to handle comments.

# Creating Comments

You need to define a route. Here is the list of the current routes with a new one added for the comment form at the bottom.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |
| /playlists/:id     | GET       | show    |
| /playlists/:id/edit| GET       | edit    |
| /playlists/:id     | PUT/PATCH | update  |
| /playlists/:id     | DELETE    | Destroy |
| /playlists/comments | POST      | create  |

The Comment route uses the POST method since creating a Comment will be sending data and creating new resources.

# Setting the Action on the Comment Form

Get the form to call the new route by adding an action. Make sure to set the method as `post`, use the route defined above.

> [action]
>
> Set the method to `post` in the comment form on `templates/partials/comment_form.html`:
>
```HTML
<!-- templates/partials/comment_form.html -->
>
<form action='/playlists/comments' method='post'>
    <fieldset>
        ...
```

# Associating Comments with Playlists through Playlist Id

Comments belong to a single playlist, so we need to keep track of this relationship. We need to **Associate** a comment with a playlist. The comment will be a **child** to the **parent** playlist. This relationship can be expressed through these two phrases:

1. Playlists have many comments
1. Comments belong to a playlist

We'll associate a comment with a playlist by attaching a `playlist_id` attribute to each comment that points to its parent playlist id. We have to set this at the time of creation, so we can do this by adding a hidden input field that sends in the parent playlist id.

# Add a Hidden Field

Let's include the `_id` of the playlist with the form. One way you can accomplish this is using a **hidden form field**. A hidden field is an input tag that has a value that isn't visible in the browser.

> [action]
>
> Add a hidden field to the form and set the value to the id of the playlist in `templates/partials/comment_form.html`:
>
```HTML
<!-- templates/partials/comment_form.html -->
>
<form action='/playlists/comments' method='POST'>
    <!-- Add the hidden input -->
    <input type='hidden' value='{{ playlist._id }}' name='playlist_id'/>
    <fieldset>
        ...
```

The hidden field gets a value from the `playlist._id` and has a name of `playlist_id`.

Submitting this form will include the id of the playlist along with the title and content input by the user writing the comment.

# Define a Route  

Define a new route in express to handle this new form. We can do this inside of `app.py`.

> [action]
>
> Create a submit route for comments in `app.py`:
>
```python
>
@app.route('/playlists/comments', methods=['POST'])
def comments_new():
    """Submit a new comment."""
    return 'playlists comment'
```

This should print the message 'playlists comment' to the browser when the form is submitted. This is a good test to check if everything is working so far.

# Create a Comments MongoDB Collection

So far, we only have one collection in our MongoDB database: the `playlists` collection. Let's add one for comments.

> [action]
>
> At the top of your `app.py` file, add another database collection after `playlists`:
>
```python
# app.py
...
db = client.get_default_database()
playlists = db.playlists
# Add this line:
comments = db.comments
```

# Adding a reference to a Playlist inside a Comment

Every Comment will need to have reference to a Playlist. This reference is the `_id` of the Playlist. Remember, id's are unique so every Playlist can ask for all Comments that reference its id. Or conversely, any Comment will be able to refer to the Playlist the Comment was written for through the the playlist id it owns.


> [action]
>
> Open `app.py` and modify your Submit route for comments:
>
```python
# Add this header to distinguish Comment routes from Playlist routes
########## COMMENT ROUTES ##########
>
@app.route('/playlists/comments', methods=['POST'])
def comments_new():
    """Submit a new comment."""
    # Replace the one liner from before with the below code:
    comment = {
        'title': request.form.get('title'),
        'content': request.form.get('content'),
        'playlist_id': ObjectId(request.form.get('playlist_id'))
    }
    print(comment)
    comment_id = comments.insert_one(comment).inserted_id
    return redirect(url_for('playlists_show', playlist_id=request.form.get('playlist_id')))
```

Test your work. Write a comment and submit the form. The `console.log` call will allow you to see your comment in the console. If there are any errors they should also appear in the console.

# Loading Comments for a Playlist

You still can't see comments in the browser! Let's tackle that problem next.

This is a chicken and egg problem. We'd like to see comments but can't see any if they don't exist in the database. For this reason we chose to write the code that create comments first. Now we will write the code to display comments.

> [action]
>
> Open `app.py` and make these changes to a playlist's show route, so that it also shows comments for a single playlist:
>
```python
# app.py
@app.route('/playlists/<playlist_id>')
def playlists_show(playlist_id):
    """Show a single playlist."""
    playlist = playlists.find_one({'_id': ObjectId(playlist_id)})
    # Add the below line:
    playlist_comments = comments.find({'playlist_id': ObjectId(playlist_id)})
    # Edit the return statement to be the following:
    return render_template('playlists_show.html', playlist=playlist, comments=playlist_comments)
```

Now instead of just sending a playlist to the template, we will also be sending an array of comments that are associated with that playlist.

# Displaying comments

Now that you have an array of comments, we need to display them.

A partial will be a good way to keep your code clean and organized.

> [action]
>
> Create a new file `templates/partials/comment.html`.
>
```HTML
<!-- templates/partials/comment.html -->
>
<div class='card'>
    <div class='card-block'>
        <h4 class='card-title'>{{ comment.title }}</h4>
        <p class='card-text'>{{ comment.content }}</p>
    </div>
</div>
```
>
> Now display comments using this partial in `templates/playlists_show.html`.
>
```HTML
<!-- templates/playlists_show.html -->
>
<a href='/'>Back to Home</a>
>
    ...
>
    {% include 'partials/comment_form.html' %}
>
    <hr>
>
    <!-- Add the below code -->
    <!-- Show Comments -->
    {% for comment in comments %}
        {% include 'partials/comment.html' %}
    {% endfor %}
>
  </div>
</div>
```

Use a `{% for %}` block to display one copy of the `comment.html` partial for **each** item in the comments array.

Test your work! Navigate to a single Playlist and add a comment. Submitting the comment should display a new comment below the Comment form.

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Users can see comments'
$ git push
```

# What Just Happened?

You created a relationship between two different document/records in your database. Playlists each have a unique id. By saving the id of a Playlist with a Comment, comments can find the Playlist that they are are associated with. Playlists can also find all of the Comments that are associated with their id.

This is a **one to many relationship**. This an important concept in database design, and an important tool you will use when managing data.
