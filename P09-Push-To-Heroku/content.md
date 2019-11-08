---
title: "Push to Production with Heroku"
slug: push-to-heroku
---

Time to ship some code! Since we've built and styled our Playlister app, let's show it to the world by putting it online. We'll use a service called [Heroku](https://www.heroku.com) that is free, but it does require a **credit card** to be on file.

# Sign Up For and Install Heroku

> [action]
>
> First you need to create a Heroku account at [heroku.com](https://www.heroku.com).
>
> Then you will need to add the Heroku Command Line Interface (CLI) to your bash terminal so we can interact with the Heroku service via the terminal and git. Follow [these instructions](https://devcenter.heroku.com/articles/heroku-cli) to install the Heroku CLI.

# Install Gunicorn

Heroku does not provide a web server, and the Flask web server is not well-suited for production use because it is single-threaded. If we want our server to perform well when multiple users make requests simultaneously, then we should use a multi-threaded web server. We will be using **gunicorn**.

> [action]
> Install gunicorn in your project directory:
>
```bash
(env) $ pip3 install gunicorn
```

We'll then need to update our `requirements.txt` file so that it reflects all of the Python modules that we have installed so far. Heroku will use this file to understand which modules need to be installed when starting our production server.

> [action]
> Update your requirements.txt file:
>
```bash
(env) $ pip freeze > requirements.txt
```

# Adding Your Procfile

Before we push to Heroku, we'll have to create a special file, called `Procfile`, that lets Heroku know how to run your website.

> [action]
>
> Create and open your new `Procfile` using the following commands:
>
```bash
$ touch Procfile
$ atom Procfile
```

Next, we need to insert the command to run our application on Heroku.

> [action]
>
> Paste the following into your new `Procfile` and save:
>
```bash
web: gunicorn app:app
```

Great! When we push to Heroku, it'll know how to run our application. Keep going!


# Pushing to Heroku using Github

Let's start by committing what we have so far so that we have that `Procfile`.

> [action]
>
> Commit the `Procfile`
>
```bash
(env) $ git add .
(env) $ git commit -m 'adding Procfile'
```

Now let's use the `heroku` command to create a heroku app and name it with "Playlister" and then our initials. So someone named Samantha Bee would be "Playlister-sb". This `heroku create` command will add our heroku app as a git remote repository that we will be able to push to using the git command `git push`. We can see our remote repos by using the command `git remote -v`.

> [action]
>
> Create your heroku app, replacing `MY-INITIALS` with your initials
>
```bash
(env) $ heroku create playlister-MY-INITIALS
(env) $ git remote -v
```

Alright, now we can push our code to heroku and open our new website!

> [action]
>
> Push to Heroku!
>
```bash
(env) $ git push heroku master
(env) $ heroku open
```

You are probably seeing an error screen right now! Bit of a let down maybe. But also a good lesson - **Things never work the first time**. Let's debug our issues.

# Scale Your Heroku application

We need to run an additional command that tells Heroku to assign a free worker to our deployment in order to run your website. In Heroku syntax, that means executing the `ps:scale` command, like so:

> [action]
>
> Run the `ps:scale` command:
>
```bash
(env) $ heroku ps:scale web=1
```

**Just like `heroku create`, you only need to execute this command once per project**!

# Heroku Logs

When ever you have an error or problem with Heroku, you can see the logs by running `heroku logs`.

```bash
$ heroku logs
```

If you want to see the logs to continually you can add `--tail` to this command.

```bash
$ heroku logs --tail
```

What error are we seeing in heroku now? What do we need to do?

# Adding a Production Database

It looks like the error is that we cannot connect to our mongodb database. That's because it is looking at the `'mongodb://localhost/Playlister'` URI, but that is on our local computer and heroku, which is remote, doesn't have access to that. So we have to add a mongodb heroku add-on called [mLabs](https://mlab.com/).

> [action]
>
> Add `mLabs`:
>
```bash
$ heroku addons:create mongolab:sandbox
```

Then we have to point to this production mongodb database URI in our `app.py` file. We'll use `os.environ.get` to get the `MONGODB_URI` **environment variable**.

> [action]
>
> Update `app.py` to point to the mongodb URI if it exists:
>
```python
# app.py
import os
...
host = os.environ.get('MONGODB_URI', 'mongodb://localhost:27017/Playlister')
client = MongoClient(host=host)
db = client.get_default_database()
playlists = db.playlists
```

Next, follow the steps again to `git add`, `git commit`, and `git push heroku master` to ensure that our changes are reflected in Heroku.

Now if we try to open our heroku app via the terminal using `heroku open`, what happens?

> [action]
>
> You may need to add `?retryWrites=false` to your MongoDB URL. If so, change the 'host=' line above to:
>
```python
client = MongoClient(host=f'{host}?retryWrites=false')
```

# ... Hanging?

Another error! This is a weird one. First it just hangs for a while, then times out, and then the error says "cannot bind on $PORT". This is because Heroku does not use port 5000. It uses another port available in production at the environment variable `PORT`, just like your mongoDB URI.

> [action]
>
> Let's fix that by setting the port also with the `os.environ.get`. Then we have to point to this production mongodb database URI in our `app.py` file.
>
```python
# app.py
>
if __name__ == '__main__':
  app.run(debug=True, host='0.0.0.0', port=os.environ.get('PORT', 5000))
```

Once again, you will need to commit your `app.py` and run `git push heroku master`.

Now if we try to open our heroku app, what happens?

# WooHoo!

Awwww yeah - you did it! You made your first RESTful and Resourceful app using Flask, Jinja2, and MongoDB!
