---
title: "Styling with Bootstrap"
slug: adding-bootstrap
---

Now that you have the user's experience mapped out and the functionality built, you can style your pages using Twitter's project [Bootstrap](http://getbootstrap.com/). Bootstrap is the web's most popular **CSS Framework**.

The main elements we will be using are the...

* Grid
* Navbar
* Forms
* Inputs
* Buttons

There is a lot more that you can explore and even extend and modify.

# Adding Bootstrap with a Content Delivery Network (CDN)

It is easy to get started with bootstrap quickly by using the CDN links bootstrap maintains.

We'll add the `<link>` to bootstrap's css in our `<head>` tag in the `base.html` file. And although we aren't going to use any of bootstrap's JavaScript's components, for completeness's sake, we'll put the JavaScript `<script>` tag just before the closing `</body>` tag.

> [action]
>
> Add the bootstrap links to `templates/base.html`:
>
```html
<!-- templates/base.html -->
>
<!doctype html>
<html>
<head>
    <meta charset='utf-8'>
    <title>Playlister</title>
    <!-- Latest compiled and minified CSS -->
    <link rel='stylesheet' href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css'>
</head>
<body>
>
    {% block content %}{% endblock %}
>
    <!-- Latest compiled and minified JavaScript -->
    <script src='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js'></script>
</body>
</html>
```

Just after doing that, refresh your page. The typography and layout should change just a tiny bit if it worked.

# Adding a Navbar

Add the most common navigational component - a top navbar. We'll have it have a button to create a new playlist so that anywhere a user navigates to they can always make a new playlist.

> [action]
>
> Make a new partial. Create a new file `navbar.html` in the `templates/partials` directory.
>
```html
<!-- templates/partials/navbar.html -->
>
<nav class='navbar navbar-default'>
    <div class='container-fluid'>
        <div class='navbar-header'>
            <a class='navbar-brand' href='/'>
                Playlister
            </a>
        </div>
        <div class='pull-right'>
            <a href='/playlists/new' class='btn btn-default navbar-btn'>New Playlist</a>
        </div>
    </div>
</nav>
```
>
>Include this in `templates/base.html`
>
```html
...
<body>
>
    {% include 'partials/navbar.html' %}
>
    {% block content %}
 ...
```

# Adding a Grid Container

People argue about the usefulness of some of the more complex parts of Bootstrap, but everyone agrees that to build websites you need a grid system. Bootstrap ships with a 12-column grid. Meaning you can break up the whole page, or any element into 12 columns and that way construct an appealing layout.

> [action]
>
> We'll start by wrapping the `{% block content %}` with a `container` class. In `templates/base.html`:
>
```html
<!-- templates/base.html -->
...
<body>
    ...
    <div class='container'>
        {% block content %}{% endblock %}
    </div>
...
```

Refresh and see that this gives us some satisfying gutters on the sides of the page. If we change the screen size by resizing the window, we'll see that these gutters vanish when our screen shrinks beyond a certain size. This is because Bootstrap's grid system is **Responsive** meaning it changes depending on the size of the screen that is being shown on. Pretty neat!

# Adding a Responsive Grid with Cards

Now that you have a container, style your `playlists_index` template to have each playlist take up 1/4 of the `container`. First  wrap the `{% for %}` block in a `row` class and then wrap each playlist in its own `col-sm-3` class.

`col-sm-3` means: On screens larger than a tablet, have this element take up three cells out of twelve that are the whole width of its parent element. Since 3/12 = 1/4 this will give us each playlist taking up one quarter of the `container`'s width.

> [action]
>
> Add the class to the following `div` in `templates/playlists_index.html`:
>
```html
<!-- templates/playlists_index.html -->
{% extends 'base.html' %}
>
{% block content %}
<h1>Playlists</h1>
>
<div class='row'>
    {% for playlist in playlists %}
        <div class='col-md-3'>
            <h2><a href='/playlists/{{ playlist._id }}'>{{ playlist.title }}</a></h2>
            <small>{{ playlist.description }}</small>
        </div>
    {% endfor %}
</div>
{% endblock %}
```

<!-- -->

> [challenge]
>
> Can you wrap each of the playlists in a `card` class? Use the [Cards Documentation](https://getbootstrap.com/docs/4.1/components/card/) for help.

# Styling the Show Template

The show template should make the text relatively narrow, because people don't like reading all the way across the screen. Use an `offset` grid class to push a column over to the right.

> [action]
>
> Add the following classes to `templates/playlists_show.html`:
>
```html
<!-- templates/playlists_show.html -->
{% extends 'base.html' %}
>
{% block content %}
<a href='/'>Back to Home</a>
<div class='row'>
    <div class='col-sm-6 col-sm-offset-3'>
        <h1>{{ playlist.title }}</h1>
        <h2>{{ playlist.description }}</h2>
        {% for video in playlist.videos %}
            <div class="card"><div class="card-body">
                <iframe width="420" height="315" src="{{ video }}"></iframe>
            </div></div>
        {% endfor %}
        <p><a href='/playlists/{{ playlist._id }}/edit'>Edit</a></p>
        <p><form method='POST' action='/playlists/{{ playlist._id }}'>
            <input type='hidden' name='_method' value='DELETE'>
            <button class='btn btn-primary' type='submit'>Delete</button>
        </form></p>
    </div>
</div>
{% endblock %}
```

# Styling the Forms

> [action]
>
> Now let's add some bootstrap styling to `templates/playlists_new.html`:
>
```html
<!-- templates/playlists_new.html -->
{% extends 'base.html' %}
>
{% block content %}
<div class='row'>
    <div class='col-sm-6 col-sm-offset-3'>
        <form method='POST' action='/playlists'>
            {% include 'partials/playlists_form.html' %}
            <!-- BUTTON -->
            <div class='form-group'>
                <button class='btn btn-primary' type='submit'>Save Playlist</button>
            </div>
        </form>
    </div>
</div>
{% endblock %}
```
>
> Make the same changes to `templates/playlists_edit.html`:
>
```html
<!-- templates/playlists_edit.html -->
{% extends 'base.html' %}
>
{% block content %}
<div class='row'>
    <div class='col-sm-6 col-sm-offset-3'>
        <form method='POST' action='/playlists/{{playlist._id}}'>
            <input type='hidden' name='_method' value='PUT'/>
            {% include 'partials/playlists_form.html' %}
            <!-- BUTTON -->
            <div class='form-group'>
                <button class='btn btn-primary' type='submit'>Save Playlist</button>
            </div>
        </form>
    </div>
</div>
{% endblock %}
```

Now add some Bootstrap classes to the `playlists-form.handlebars` partial. These class will make your form and form elements look better. Notice that each form element is wrapped in a `div` with the class `form-group` and form elements, like `input` get the class `form-control`.

> [action]
>
> Update `templates/partials/playlists_form.html` to the following:
>
```html
<fieldset>
    <legend>Edit Playlist</legend>
    <!-- TITLE -->
    <div class='form-group'>
        <label for='playlist-title'>Title</label><br>
        <input class='form-control' id='playlist-title' type='text' name='title' value='{{ playlist.title }}'/>
    </div>
>
    <!-- DESCRIPTION -->
    <div class='form-group'>
        <label for='description'>Description</label><br>
        <input class='form-control' id='description' type='text' name='description' value='{{ playlist.description }}' />
    </div>
>
    <!-- DESCRIPTION -->
    <div class='form-group'>
        <label for='playlist-videos'>Videos</label><br/>
        <p class='text-muted'>Add videos in the form of 'https://youtube.com/embed/KEY'. Separate with a newline.</p>
        <textarea class='form-control' id='playlist-videos' name='videos' rows='10' />{{ "\n".join(playlist.videos) }}</textarea>
    </div>
</fieldset>
```

Looking pretty good now, huh?

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Added vanilla bootstrap'
$ git push
```

Almost done! Let's finish it off.
