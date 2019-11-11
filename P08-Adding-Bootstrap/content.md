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
    <!-- We'll use the compiled and minified CSS from version 4.3.1 for this tutorial -->
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
</head>
<body>
>
    {% block content %}{% endblock %}
>
    <!-- We'll use jquery version 3.3.1 for this tutorial -->
    <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" crossorigin="anonymous"></script>
    <!-- We'll use popper.js version 1.14.7 for this tutorial -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.7/umd/popper.min.js" integrity="sha384-UO2eT0CpHqdSJQ6hJty5KVphtPhzWj9WO1clHTMGa3JDZwrnQq4sF86dIHNDz0W1" crossorigin="anonymous"></script>
    <!-- We'll use the compiled and minified JavaScript from version 4.3.1 for this tutorial -->
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/js/bootstrap.min.js" integrity="sha384-JjSmVgyd0p3pXB1rRibZUAYoIIy6OrQ6VrjIEaFf/nJGzIxFDsf4x0xIM+B07jRM" crossorigin="anonymous"></script>
</body>
</html>
```

Just after doing that, refresh your page. The typography and layout should change just a tiny bit if it worked.

> [info]
>
> For ease of linkage in this tutorial, we will be using version 4.3.1 of Bootstrap, and the appropriate resources that are needed to support that version of Bootstrap.
>
> If you would like to use the latest version of Bootstrap, visit their [website](https://getbootstrap.com) to find the latest version and the resources that it needs to use the BootstrapCDN.

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
    <!-- add in the navbar -->
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
    <!-- Add the container class below  -->
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
<!-- Add the following classes to their appropriate elements  -->
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
> Can you wrap each of the playlists in a `card` class? Use the [Cards Documentation](https://getbootstrap.com/docs/4.3/components/card/) for help. Again this  is the 4.3.1 specific documentation, but you can get the latest version by visiting  the [Bootstrap website](https://getbootstrap.com) and clicking  on their **Documentation** (the documentation URL changes with each version, so we can't link the latest here).

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
        <p><form method='POST' action='/playlists/{{ playlist._id }}/delete'>
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
> Now let's add some bootstrap styling to `templates/playlists_new.html` Add the following classes to the file:
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
> Make the same changes (adding classes) to `templates/playlists_edit.html`:
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
> Update `templates/partials/playlists_form.html` to have the following classes:
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
    <!-- VIDEO LINKS -->
    <div class='form-group'>
        <label for='playlist-videos'>Videos</label><br/>
        <p class='text-muted'>Add the ID of the videos you want to include in your playlist. Separate with a newline.</p>
        <textarea class='form-control' id='playlist-videos' name='videos' rows='10' />{{ "\n".join(playlist.video_ids) }}</textarea>
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
