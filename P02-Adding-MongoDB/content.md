---
title: "Adding a MongoDB Database to Your App"
slug: adding-mongodb
---

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

> [action]
>
> Let's install mongodb using homebrew!
>
```bash
$ brew update
$ brew install mongodb
```
>
> You also need to create a folder to save your databases on your computer.
>
```bash
$ sudo mkdir -p /data/db
```

<!-- -->

> [info]
>
> If you have problems with permissions with this folder you can "change owner" of the directory using this command:
>
```bash
sudo chown -R $USER /data/db
```

<!-- -->

> [action]
>
> Now you need to start the MongoDB daemon in order to use MongoDB:
>
```bash
$ mongod # SHORT FOR "MONGO DAEMON"
```

Once you turn on mongod you can close the terminal tab you used to do that.

The command `mongod` should start MongoDB and now it will be accessible from your web server. Let's configure Flask to write to MongoDB using the Python library `PyMongo`.

# PyMongo - A MongoDB Library

To read from MongoDB in our server code, we'll be using PyMongo. Let's install it now.

> [action]
>
> First, install the PyMongo library using pip:
>
```bash
(env) $ pip install pymongo
```

Remember to run `pip freeze > requirements.txt` to update your list of installed packages!

Now initialize MongoDB in `app.py` and connect to our database that we'll name after our app.

> [action]
>
> Add the following to `app.py`:
>
```python
from pymongo import MongoClient
>
client = MongoClient()
db = client.Playlister
playlists = db.playlists
>
...
>
```

Voila, you are connected to your database! But wait, you haven't written or read from it yet! To do that, we'll have to make a database query.

# Our First MongoDB Query with a Mongoose Model

Let's return to our root path that displays our `playlists_index` template.

First let's comment out our `playlists` variable that we hard coded. We're gonna use the database now instead with the model `Playlist` we instantiated.

> [action]
>
> Comment out the mock data and update the root route in `app.js` to the following:
>
```python
# app.py
>
# playlists = [
#   { 'title': 'Great Playlist' },
#   { 'title': 'Next Playlist' }
# ]
>
@app.route('/')
def playlists_index():
    """Show all playlists."""
    return render_template('playlists_index.html', playlists=playlists.find())
```

The `find()` method returns an iterable of all playlists in our database.

Refresh your browser at `localhost:5000`. What do you see?

> [solution]
> You shouldn't see anything! :D - that's because there are no playlists in your database right now.

In the next lesson we will save a new playlist to our database. Onward!

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Added MongoDB and Playlist model'
$ git push
```
