# Refactor Using Blueprints

Good work on completing the tutorial up to this point!

Earlier on [in this tutorial](https://github.com/MakeSchool-Tutorials/Flask-Playlister/tree/master/P03-Creating-A-Playlist), you may recall reading the following:

>
"For this app it is so simple, that we'll just leave all that logic in the `app.py` file."
>

Well... that was an example of *selective disinformation*. Because in this section, we will indeed be **refactoring** our codebase, so that our routes are spread out over several different files!

## But, why are we doing this?
This is one of the realities of agile software development: things change. As you've already seen in this tutorial, our original Flask app has quickly grown as you needed to CRUD new resources, add tests, deploy to production, and more. In industry, it's common for codebases to have thousands, if not millions, of lines of code.

To manage the chaos of these gigantic codebases, professional software engineers will often *modularize* their code, i.e. they'll use separate files and folders (also called *modules* in Python) for specific pieces of their application. This is probably just like what you do to organize your own computer to stay on track - so, why not do it for our software projects as well?

# Make an `app` Package

At the present moment, your project directory probably looks something like the following:

<img src="https://i.postimg.cc/htyF66Kw/start.png" height=300 alt="initial project structure">

Rather than having 1 large file named `app.py` why don't we turn that into a folder, containing several files for the different components of our application? 

If you follow along with me, then by the end of this section your project directory will probably look something more like this:

<img src="https://i.postimg.cc/qMHpxP7x/woohoo-final-folder-strucute.png" height=450 alt="final project structure">

Notice how `app.py` has turned into a folder named `app/`? That's a special kind of folder, called a Python *package* - because it contains an `__init__.py` module, we can use it to conveniently instaniate objects in one, modular piece of our application, and import them to be used in other modules!

> [action]
> Start by creating a new folder named `app` in th root your project directory. Then, inside that folder create a file named `__init__.py`. Inside, place the following code:
> 

```python

# app/__init__.py
from flask import Flask


def create_app():
    app = Flask(__name__)
    return app

```

# Modularize MongoDB
Next up, where shall we place the `db` object?

Since we'll need to connect to the database from several different pieces of our application (i.e. the routes we have for both the playlists and comments), it's best to place it in its own, dedicated module.

> [action] Go ahead and create a new module in the `app` package named `util.py`. 
> Place the following function inside:

```python
# app/util.py
import os

from pymongo import MongoClient


def get_db_collections():
    # point to the MongoDB URI (if it exists)
    host = os.environ.get("MONGODB", "mongodb://localhost:27017/Playlister")
    client = MongoClient(f"{host}?retryWrites=false")
    # return the collections (use in routes)
    return db.playlists, db.comments

```
For background, `util.py` is a name commonly used by Python developers when they want to make a module that's just for storing useful helper functions, or *utilities*. So, feel free to add any other functions in this `util.py` script as you go through this section!

> [action] Double check that your `app` package has both an `__init__.py` and `util.py` files (just like below) before moving to the next steps!

<img src="https://i.postimg.cc/ryxksf90/end-of-step1-init-app-pakage.png" height=300 alt=" project structure after step 1">


# Now Commit

```bash
$ git add .
$ git commit -m 'init app package'
$ git push
```

# Re-Run the Application
Alright - so how do run the application now, since we've started refactoring?

Up to now, we've primarily used the `app.py` file to both 1) instantiate and 2) run our Flask application. While this approach works for simple projects, the downside is that as your code grows, we'll find it harder and harder to debug that `app.py` file, because there's so many tasks we're relying on it to do for us. When this happens, we can say that we've *tightly-coupled* that module to both making and running our `app` variable.

The good news is, our refactored codebase will fix that problem: just as we've created a separate `app` package to handle instantiating the Flask application, **we will now make a separate module to handle running that application!**

> [action] Outside of the `app` package, make a new file called `run.py` in the root of your project.
> In that file, place the following (it should look very similar to the bottom of your previous `app.py`) file:

```python
# run.py
import os
# this line imports from app/__init__.py
from app import create_app  


if __name__ == "__main__":
    # there still needs to be a variable called "app" here (for Flask reasons)
    app = create_app()  
    app.run(debug=True, host="0.0.0.0", port=os.environ.get("PORT", 5000))

```

With that, you may use this command to see if your app can now run (at least without any errors in the Terminal):

```
(env) $ export FLASK_ENV=development; python run.py
```


# Now Commit

```bash
$ git add .
$ git commit -m 'add run.py'
$ git push
```

# Create a Blueprint for Playlists
As you know, most of the code in our `app.py` file is used for defining the routes users might like to take in our application. They've taken up a pretty big chunk of code, so it's time to modularize them as well. There's many ways to do this - in this tutorial, we'll be using what Flask calls [*blueprints*](https://exploreflask.com/en/latest/blueprints.html).

This concept is very tactical - simply put, a blueprint allows us to encapsulate a collection of views related to a given resource in our app - in this case, the playlists or comments - into a single object, usually defined in its own package.

We'll start with making the blueprint for the playlists. This will involve copying over the related routes in your `app.py`, with just a few small modifications. Let's get started!

> [action] Inside of the `app` package, make another folder named `playlists`.
> Then, turn that folder into a package by creating an `__init__.py` file inside (you can leave it blank).

> [action] Next, make another module inside the new `playlists` package named `routes.py`. Finally, go ahead and place this code inside to make the `Blueprint` object:

```python
# app/playlists/routes.py
from flask import Blueprint


# encapsulate the Playlist resource
playlists_bp = Blueprint("playlists", __name__)

```

With that done, we're now ready to add the routes for related to this resource:

> [action] First, import the code we need to connect to our Mongo instance (which will be needed by our routes):

```python
# app/playlists/routes.py
from flask import Blueprint

from app.util import get_db_collections


# grab a reference to those db collections!
playlists, comments = get_db_collections()
# encapsulate the Playlist resource
playlists_bp = Blueprint("playlists", __name__)

```

Now, you're finally ready to moving your routes into the `routes.py` module. Please note that instead of using `@app.route` to decorate each of our controller functions, we will now need to use the name of our `Blueprint` object, `@playlist_bp` - an example is shown below:

> [action] Before doing anything else, move the `templates` folder into the `app` package (on the same level as the `playlists` folder). This will make our HTML templates discoverable to our routes again.

> [action] Next, bring the route for `playlists_index` into `routes.py`, along with all the relevant imports:

```python
# app/playlists/routes.py
from flask import Blueprint, render_template

from app.util import get_db_collections


# grab a reference to those db collections!
playlists, comments = get_db_collections()
# encapsulate the Playlist resource
playlists_bp = Blueprint("playlists", __name__)

@playlists_bp.route("/")
def playlists_index():
    """Show all playlists."""
    return render_template("playlists_index.html", playlists=playlists.find())

```

Great work so far my friend. But, you may notice that if you go to [http://localhost:5000/](http://localhost:5000/), you still don't see the playlists. What gives?

A couple more steps remain: now that we've tied the routes to our `Blueprint` object rather than our Flask application, our `app` variable doesn't actually know about them, until we tell it programmatically. This is an example of *loose coupling*. 

So, go ahead and let your `app` variable know about that playlist blueprint - for context, this is also known as *registering a blueprint* in Flask:

> [action] Revisit the `__init__.py` module in the `app` package. Place the following code inside:

```python
# app/__init__.py
from flask import Flask
from app.playlists.routes import playlists_bp


def create_app():
    """Init the app and register all the Playlist/Comment routes."""
    app = Flask(__name__)
    app.register_blueprint(playlists_bp)
    return app

```

See anything different when you go to [http://localhost:5000/](http://localhost:5000/)?

Now, we are ready to add all our old playlist routes back into the application:

> [action] For this next step, feel free to copy over the rest of your playlist routes from the ol' `app.py` file to `app/playlists/routes.py`. Just as before, make sure to add all the relevant imports, and change all the decorators from `@app.route` to `@playlists_bp.route`!

Finally, there's one final thing I should advise you to do - is to *namespace* your templates in the `routes.py` module. What does this mean?

In any controller function where you return using the `render_template()` function, there is no issue, because Flask can find the HTML template you pass in. But, some of routes return a `redirect`, which uses the `url_for()` function. This is basically telling our application to go find the controller for another route; yet, at the same time remember, our `app` doesn't know about any of the routes - just the blueprints that encapsulate them.

So how do we properly use the `url_for()` function now? We need to prepend the name of the controller function, with the name of the `Blueprint` that has that controller function (e.g. `"my_blueprint.controller_func"`). Let's see how this is done:

> [action] Namespace the controller function that we pass into the `url_for()` function in `routes.py`. This change only affects the `playlists_submit`, `playlists_update`, and `playlist_delete` controller functions:

```python
# app/playlists/routes.py
from flask import Blueprint, render_template, request, redirect, url_for
from bson.objectid import ObjectId

from app.util import get_db_collections
...

@playlists_bp.route("/playlists", methods=["POST"])
def playlists_submit():
    """Submit a new playlist."""
    playlist = {
        "title": request.form.get("title"),
        "description": request.form.get("description"),
        "videos": request.form.get("videos").split(),
    }
    print(playlist)
    playlist_id = playlists.insert_one(playlist).inserted_id
    return redirect(
        url_for("playlists.playlists_show", playlist_id=playlist_id)
    )

...

@playlists_bp.route("/playlists/<playlist_id>", methods=["POST"])
def playlists_update(playlist_id):
    """Submit an edited playlist."""
    updated_playlist = {
        "title": request.form.get("title"),
        "description": request.form.get("description"),
        "videos": request.form.get("videos").split(),
    }
    playlists.update_one({"_id": ObjectId(playlist_id)}, {"$set": updated_playlist})
    return redirect(
        url_for("playlists.playlists_show", playlist_id=playlist_id)
    )

@playlists_bp.route("/playlists/<playlist_id>/delete", methods=["POST"])
def playlist_delete(playlist_id):
    """Delete one playlist."""
    playlists.delete_one({"_id": ObjectId(playlist_id)})
    return redirect(url_for("playlists.playlists_index"))

```

Excelsior! Double-check that your app still works manually. Then, you are free to continue.


# Now Commit

```bash
$ git add .
$ git commit -m 'add blueprint for playlists'
$ git push
```

# Create a Blueprint for Comments
This section will be a piece of cake - we'll mainly just repeat what we did for the playlist-related routes, for our comments-related ones.

> [action] Go ahead and make a new `comments` package in the `app` folder, with an `__init__.py` and `routes.py`.

> [action] In `comments/routes.py`, instantiate your `Blueprint` for the comments resource. It should like something similar to the following:

```python
# app/comments/routes.py
from flask import Blueprint


comments_bp = Blueprint("comments", __name__)

```

> [action] As before, register the new `Blueprint` object in the Flask application:

```python
# app/__init__.py
from flask import Flask
from app.comments.routes import comments_bp
from app.playlists.routes import playlists_bp


def create_app():
    """Init the app, and all the same routes as we had before."""
    app = Flask(__name__)
    app.register_blueprint(playlists_bp)
    app.register_blueprint(comments_bp)
    return app
```

> [action] Onward - let's now connect to our database, and add the comments-related routes to our blueprint. Remember to namespace the routes that return a `redirect`, as well:

```python
# app/comments/routes.py
from flask import Blueprint, request, redirect, url_for
from bson.objectid import ObjectId

from app.util import get_db_collections

playlists, comments = get_db_collections()

comments_bp = Blueprint("comments", __name__)


@comments_bp.route("/playlists/comments", methods=["POST"])
def comments_new():
    """Submit a new comment."""
    comment = {
        "title": request.form.get("title"),
        "content": request.form.get("content"),
        "playlist_id": ObjectId(request.form.get("playlist._id")),
    }
    comment_id = comments.insert_one(comment).inserted_id
    return redirect(
        url_for("playlists.playlists_show", playlist_id=request.form.get("playlist._id"))
    )


@comments_bp.route("/playlists/comments/<comment_id>", methods=["POST"])
def comments_delete(comment_id):
    """Action to delete a comment."""
    if request.form.get("_method") == "DELETE":
        comment = comments.find_one({"_id": ObjectId(comment_id)})
        playlist_id = comment.get("playlist_id")
        comments.delete_one({"_id": ObjectId(comment_id)})
        return redirect(
            url_for("playlists.playlists_show", playlist_id=playlist_id)
        )

```

Try out the refactored routes for the comments-related blueprint, as an extra check. By this point, your folder structure should look something like the following:

<img src="https://i.postimg.cc/52n11jSs/end-of-step4.png" height=475 alt="project structure after step 4">

# Now Commit

```bash
$ git add .
$ git commit -m 'add blueprint for comments'
$ git push
```

# Remove `app.py`
You will no longer need it for the rest of this tutorial.
# Refactor the Tests
Alrighty then! Our project is now more modular, but refactoring does not stop there - let's also modify the `tests.py`, so we don't need to keep manually testing out all our routes.

We're only going to make 1 small change, to how we import the `app` variable. As opposed to importing it from `app.py` (which no longer exists), we'll use the `create_app()` function. Let's do the following:

> [action] In `tests.py`, change the import statement `from app import app` to say `from app import create_app`

> [action] Next, change the `setUp()` method in the `PlaylistsTests` test suite to use `create_app()`:

```python
# tests.py
from unittest import TestCase, main as unittest_main, mock
from app import create_app
...

class PlaylistsTests(TestCase):
    """Flask tests."""

    def setUp(self):
        """Stuff to do before every test."""

        # Get the FLask test client
        app = create_app()
        self.client = app.test_client()

# rest of file
```

Run your tests again my friend - do you still get a line of green dots?

# Redeploy to Heroku
> [action] Since we no longer have the `app.py`, you will also need to refactor your `Procfile` to the following. 
> This is so that your deployment server on Heroku can still call `app.run()`:

```
web: gunicorn run:app
```


# Now Commit

```bash
$ git add .
$ git commit -m 'refactor tests + Procfile'
$ git push
```

Congrats! You've completed the tutorial :)


# Resources
Please see the following links or ask an instructor if you run into blockers in this section:

1. [Zain's Example Repo](https://github.com/UPstartDeveloper/Playlister-Tutorial) for a finished project.
2. [Flask documentation](https://exploreflask.com/en/latest/blueprints.html) on blueprints.
3. [Another video](https://youtu.be/Wfx4YBzg16s) that goes through using blueprints in a Flask project.