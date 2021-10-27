# Adding a MongoDB Database to Your App

## MongoDB - A NoSQL Database

**JSON** or **JavaScript Object Notation** is a way to organize data to transmit it across the internet. It looks the same as a Python dictionary. It is made up of pairs of **keys** and **values** separated by colons (`:`) and surrounded by curly braces (`{}`). For example:

```python
# JSON OBJECT
{
    'title': 'Great Playlist'
}
```

A MongoDB database allows you to save JavaScript Objects or JSON just as they are as **key value pairs**. In a MongoDB each object is called a single **document**, hence why this sort of database is called a **document-based database**. These documents are collected into groups called **collections**. We will save our playlist documents in a collection called "playlists".

MongoDB gives each document a unique identification number with the key **_id** (with an underscore). We can use that **_id** attribute to retrieve the whole document later.

Working with MongoDB is like dropping clothes off at the drycleaners. They put a number on each piece of clothing and give you a ticket for each one. Later we can get that clothing back by matching our ticket to the right number.

If we created a new playlist like this to a MongoDB database:

```python
# PYTHON DICTIONARY
{
    'title': 'A New Playlist'
}
```

Then it will save something like this:

```python
# MONGODB OBJECT
{
    '_id': '507f1f77bcf86cd799439011',
    'title': 'A New Playlist'
}
```

# Installing MongoDB

Our web server is going to save documents in MongoDB, but MongoDB itself has to run on our computer's operating system - it isn't a module we install with Python's package manager.

Let's install MongoDB using homebrew!

```bash
$ brew update
$ brew tap mongodb/brew
$ brew install mongodb-community@5.0
```

To run MongoDB, execute the following command:

```bash
$ brew services start mongodb-community@5.0
```

Verify that it is running with:

```bash
$ brew services list
```

You should see the service `mongodb-community` listed as `started`.

> If you want to stop MongoDB, you can do so with:
>
> ```bash
> $ brew services stop mongodb-community@5.0
> ```

In order to begin using MongoDb, we need to connect `mongosh` to the running instance with:

```bash
$ mongosh
```

> macOS may prevent `mongod` and/or `mongosh` from running after installation. If you receive a security error when starting `mongod` and/or `mongosh` indicating that the developer could not be identified or verified, do the following to grant `mongod` and/or `mongosh` access to run:
>
> * Open System Preferences
> * Select the Security and Privacy pane.
> * Under the General tab, click the button to the right of the message about `mongod` and/or `mongosh`, labelled either **Open Anyway** or **Allow Anyway** depending on your version of macOS.

> MongoDB updates frequently; if you are having trouble during the installation process, try referring to the official docs [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/#std-label-osx-prereq) for the most up to date procedure.

Once done, you can close the terminal tab you used to do that.

The command `mongosh` should enable MongoDB to be accessible from your web server. Let's configure Flask to write to MongoDB using the Python library `PyMongo`.

# PyMongo - A MongoDB Library

To read from MongoDB in our server code, we'll be using PyMongo. Let's install it now.

First, install the PyMongo library using pip:

```bash
(env) $ pip3 install pymongo
```

Remember to run `pip freeze > requirements.txt` to update your list of installed packages!

Now initialize MongoDB in `app.py` and connect to our database that we'll name after our app.

Add the following to the beginning of `app.py`:

```python
from pymongo import MongoClient

client = MongoClient()
db = client.Playlister
playlists = db.playlists

...
```

> Whenever you see a `...` in a code snippet, that means that the rest of your code should be contained below! Be sure **not** to type the `...`.

Voila, you are connected to your database! But wait, you haven't written or read from it yet! To do that, we'll have to make a database query.

# Our First MongoDB Query with a Mongoose Model

Let's return to our root path that displays our `playlists_index` template.

First let's comment out our `playlists` variable that we hard coded. We're gonna use the database now instead with the model `Playlist` we instantiated.

Comment out the mock data and update the root route in `app.py` to the following:

```python
# app.py

# playlists = [
#   { 'title': 'Great Playlist' },
#   { 'title': 'Next Playlist' }
# ]

@app.route('/')
def playlists_index():
    """Show all playlists."""
    # Update this line
    return render_template('playlists_index.html', playlists=playlists.find())
```

The `find()` method returns an [iterable](https://stackoverflow.com/questions/9884132/what-exactly-are-iterator-iterable-and-iteration) of all playlists in our database.

Refresh your browser at `localhost:5000`. What do you see?

<details>
<summary>Solution</summary>
<br>
You shouldn't see anything! :D - that's because there are no playlists in your database right now.
</details>

In the next lesson we will save a new playlist to our database. Onward!

# Now Commit

```bash
$ git add .
$ git commit -m 'Added MongoDB and Playlist model'
$ git push
```

# Next

Click [here](../P03-Creating-A-Playlist/content.md) to move onto the next section about creating a playlist.
