<!-- # Learning Outcomes

By the end of this tutorial, you should be able to...

1. Build a web app using Flask and Jinja2
1. Utilize a NoSQL database (MongoDB) for your web app
1. Integrate RESTful and Resourceful routing in your web app
1. CRUD a single resource
1. Understand how to use Bootstrap for basic styling -->

# How to Plan a Coding Project: User Stories

Software development these days is usually organized into **Agile Sprints** that are usually two weeks long. You'll notice evidence of this if you ever update your apps and read the update text. Sometimes it just says:

```
We ship a new version every two weeks...
```

That's because they are organizing their engineering in this standard agile way.

During each sprint developers pick a few **User Stories** to build, test, and ship to production. A **User Story** is one chunk of functionality. Engineers use the term **User Story** instead of "feature" to emphasize the user's experience in their design, development and testing. This emphasis on the user is called **User Centered Development**. Whenever you build anything on a team or by yourself, always use a **Backlog** of **User Stories** to organize and prioritize what you will build and when.

It is a good habit to build applications one resource at a time, adding all the **Resourceful Routes** for each resource before moving to the next.

So the user stories we can have for this playlist app will be as follows:

1. Users can view all playlists (index)
1. Users can create a playlist (new/create)
1. Users can view one playlist (show)
1. Users can delete a playlist (destroy)
1. Users can edit a playlist (edit/update)

Once we have this single Playlist resource build, we can move onto making an associated Comment resource.

1. Users can comment on playlists (comments#create)
1. Users can delete comments (comments#destroy)

As we finish and test user stories we'll be committing to github 🐙.

# Getting Started - Python & the Virtual Environment

Now that we have some user stories, let's initialize a Python project so we can start on the first user story in the next chapter of this tutorial. We're gonna jump right in to starting the new playlist and provide more context as new topics and concepts come up.

For reference for the rest of this chapter, you can look at the Flask [User's Guide](https://flask.palletsprojects.com/en/1.1.x/).

Open your computer's terminal and then...

# Install Python

If you don't already have Python installed, install that first.

> Whenever you see the `$` in a command, that means it should be called in your computer's terminal. Remember: Don't include the `$` in your command.

If you haven't installed Python on your computer, go ahead and install the latest version from the [Python official website](https://www.python.org/downloads/). To check if the Python installation is functional, you can type `python3` into a terminal window. You should see something like:

```bash
$ python3
Python 3.5.2 (default, Nov 17 2016, 17:05:23)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> _
```

This brings up the Python interpreter, which is waiting for you to enter a command. To exit the prompt, you can type `exit()` and press Enter, or press Ctrl-D.

# Creating a Virtual Environment

Next, we need to set up a Python **virtual environment**. This is kind of like a sandbox where we can install Python **packages** that will be active only for this particular project. If we install packages without a virtual environment, they are installed **globally** on your machine. Sometimes a new version of a package will be incompatible with the old version, so if you need to install the new version for your new project, it could unintentionally break one of your old projects. A virtual environment keeps us from polluting our other projects with potentially incompatible code.

Make a new directory called 'playlister', then navigate into that directory, and finally create a new virtual environment.

Use the terminal commands below to execute the above instructions:

```bash
$ mkdir playlister
$ cd playlister
$ python3 -m venv env
```

Virtual environment support is included with Python3, so by running `python3 -m venv env`, we are telling Python to execute the 'venv' module (the 'm' is short for 'module'), and put the resulting virtual environment in a directory called 'env'.

When you open the 'playlister' directory in Atom, you will see a directory called 'env' which contains several sub-directories. No need to worry about these for now!

We still need to **activate** our newly created virtual environment so that we can use it to install our Python packages. Go ahead and do so now.

```bash
$ source env/bin/activate
(env) $ _
```

Finally, we can install Flask in our virtual environment so that we can get started with our project.

```bash
(env) $ pip3 install flask
```

> If you ever want to deactivate your virtual environment, just type `deactivate` in your terminal window.

# Freeze Your Dependencies with Pip Freeze

Using a virtual environment also makes it easy for someone reading your code to see what packages your project depends on. To make this information visible in our code, let's use the `pip freeze` command to get a list of all of the dependencies we have so far.

In your project directory, create a `requirements.txt` file by entering the following in your terminal window:

```bash
(env) $ pip3 freeze > requirements.txt
```

If you open up the `requirements.txt` file we generated, you should see something like:

```txt
Click==7.0
Flask==1.1.1
itsdangerous==1.1.0
Jinja2==2.10.1
MarkupSafe==1.1.1
Werkzeug==0.15.5
```

> It's good to re-run the `pip freeze` command whenever you add or upgrade your packages. That way, others reading your code can re-install your dependencies without too much pain! If you ever want to install packages from a `requirements.txt` file, you can do so with:
>
> ```bash
> $ pip3 install -r requirements.txt
> ```

# Adding a Flask server

Now we are going to make a "Hello World" rendered with Flask and the templating language Jinja2, which is by default included with Flask.

Let's write our main file of the whole application, which we will call `app.py`.

Create the main file using the "touch" command.

```bash
(env) $ touch app.py
```

Now your project should have two files and a folder: `app.py`, `requirements.txt`, and `env`.

Now open your project's code base using the Atom text editor by typing:

```bash
$ atom .
```

And let's add some code to `app.py` to show a hello world.

Add the following to `app.py`:

```python
from flask import Flask
>
app = Flask(__name__)
>
@app.route('/')
def index():
    """Return homepage."""
    return 'Hello, world!'
>
if __name__ == '__main__':
    app.run(debug=True)
```

At this point, you can now try running your project!

Go to your terminal and enter:

```bash
(env) $ export FLASK_ENV=development; flask run
```

Navigate your browser to `http://localhost:5000`, and you should see "Hello World". Remember that we still haven't added a template engine yet! We are just sending text back to the browser.

Hello there code!

# Add Templates

For templating, we will be using Jinja2, which is automatically included with Flask.

Extend your **root route** ('/') to render `index.html`.

```python
from flask import Flask, render_template
>
app = Flask(__name__)
>
@app.route('/')
def index():
    """Return homepage."""
    # change the original return statement you wrote to the one below
    return render_template('home.html', msg='Flask is Cool!!')
```

Refresh your browser now and _read the error you get carefully_. This error tells us that our application can't find a `home.html` template yet (because we haven't made it!).

> It is useful to try to predict what error you might get as you are coding, and see if you get the error you expected. Errors can be a great way to check your work as you go and not go too far before checking your work.

Create the `templates` folder and `base.html` and `home.html` files.

```bash
$ mkdir templates
$ touch templates/base.html
$ touch templates/home.html
```

So now we have our **templates** folder set up with a template called `home`. Great!

Now we'll add some boilerplate code to the `base.html` template. Having a base template means that we can put things that go on *every* page in just one file of code. To make our templates for the website pages, we will be **extending** `base.html`.

Add the following into `templates/base.html`:

```html
<!-- templates/base.html -->
<!doctype html>
<html>
<head>
  <meta charset='utf-8'>
  <title>Playlister</title>
</head>
<body>
>
    {% block content %}
        BODY CONTENT GOES HERE
    {% endblock %}
>
</body>
</html>
```

Add the following to `templates/home.html` so that it will render the message from our root route:

```html
<!-- templates/home.html -->
{% extends 'base.html' %}
>
{% block content %}
    <h1>{{ msg }}</h1>
{% endblock %}
```

Now when you visit `localhost:5000` you now should see "Flask is Cool!" inside of an `h1` tag.

# Initialize, Commit, and Push

Let's start by installing git if you don't already have it installed. And then initializing our project folder as a git repository. Then we can check the git status. We should see all our files as uncommitted and unstaged to commit.

Install git if you haven't, and then initialize it:

```bash
$ brew install git
$ git init
$ git status
```

Before we commit our changes, we want to tell Git to ignore the `env` directory, since it's not necessary for anyone else who is reading or running our code. (That's what the `requirements.txt` file is for!) Let's add a **.gitignore** file, which will specify any file types, files, or folders that we do not want to include in our Git repository.

We also want Git to ignore any Python **generated files**, which are contained in the `__pycache__` directory. We should also ignore the `.DS_Store` file, since it's just macOS metadata that is irrelevant to our project.

In your project directory, create a file called '.gitignore' with the following content:

```
env
__pycache__
.DS_Store
```

Now we can stage all the files to commit, then commit them while also adding a commit message, and then double check our status. We should see we have no files to commit because we just committed them all.

Add and commit what we have so far:

```bash
$ git add .
$ git commit -m 'init'
$ git status
```

Now go to GitHub and create a public repository called `Playlister-Tutorial`.

Once you've done that, associate it as a remote for your local git project and then push to it:

```bash
$ git remote add origin GITHUB-REPO-URL
$ git push origin master -u
```

# Next

Click [here](../P01-Index-Playlists/content.md) to move onto the next section about indexing playlists.