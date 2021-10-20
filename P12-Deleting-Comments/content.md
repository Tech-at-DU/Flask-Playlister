# Deleting Comments

Finally, since we are creating comments, we should also be able to delete them.

Remember that currently we do not have authentication so we'll just be letting any user create and delete comments. As we developed this project for a go live, we would probably add authentication and only allow people to delete their own comments!

**For this chapter, you will be writing all the code on your own. Refer back to similar code you did for playlists to guide you. You can do this!**

# Deleting Comments

You need to define a route. Here is the list of the current routes with a new one added for the comment form at the bottom.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |
| /playlists/:id     | GET       | show    |
| /playlists/:id/edit| GET       | edit    |
| /playlists/:id     | PUT/PATCH | update  |
| /playlists/:id     | DELETE    | destroy |
| /playlists/comments | POST      | create  |
| /playlists/comments/:id | DELETE      | destroy  |

We'll use the `<comment_id>` url parameter to look up the comment we want to destroy.

# What the User Sees: Making a Delete Link

As always, we start with what the users sees and does. So let's make a link to delete a comment.

We can't set an `<a>` tag's method (it is always GET by default) so we are going to use a form to submit a DELETE request to our delete action path.

Let's add a link to the comments partial at: `templates/partials/comment.html` so that we can delete a comment.

# Adding the Destroy Route

Now we need a delete action route. After deleting the comment, it should redirect back to the parent playlist (`playlists_show`).

Add a delete route for comments in `app.py`:

Ok, so now try deleting a comment.

# What Happened

So now we've completed all our user stories:

1. Users can view all playlists (index)
1. Users can create a playlist (new/create)
1. Users can view one playlist (show)
1. Users can delete a playlist (destroy)
1. Users can edit a playlist (edit/update)
1. Users can comment on playlists (comments#create)
1. Users can delete comments (comments#destroy)

You used **Resource Based Development** and **Resourceful Routing** to build a playlist app!

Congrats! You got your Playlister app all set up! The following chapter has some extra challenges if you're looking to improve your app even more.

Reflect on what you've learned so far. How does it relate to the content in the rest of the course? What will you take with you?

# Feedback and Review - 2 minutes

**We promise this won't take longer than 2 minutes!**

Please take a moment to rate your understanding of learning outcomes from this tutorial, and how we can improve it via our [tutorial feedback form](https://forms.gle/kXt4Y8w2LpWSN6eU6)

# Next

Click [here](../P12-Bells-And-Whistles/content.md) to move onto the next section about adding the bells and whistles.
