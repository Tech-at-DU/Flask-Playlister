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

<img src="https://i.postimg.cc/qMHpxP7x/woohoo-final-folder-strucute.png" height=450 alt="initial project structure">

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

<img src="https://i.postimg.cc/ryxksf90/end-of-step1-init-app-pakage.png" height=300 alt="initial project structure">


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

Great work so far my friend. But, you may notice that if you go to [http://localhost:5000/](http://localhost:5000/), you still don't see your playlists. What gives?

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

# Create a Blueprint for Comments
still don't forget to namespace

# Refactor the Tests


# Redeploy to Heroku

# Resources
my repo
video

Congrats! You've completed the tutorial :)