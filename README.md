# Validations with `form_for`

Now that we know Rails automatically performs validations defined on models,
let's use this information to easily display validation errors to the user.

# Objectives

After this lesson, you'll be able to...

- use `form_for` to display a form with Validations
- print out full error messages above the form

# The differences between `form_for` and `form_tag`

This step will make heavy usage of `form_for`, the high-powered alternative to
`form_tag`. The biggest difference beteen these two helpers is that `form_for`
creates a form specifically **for** a model object. `form_for` is full of
convenient features.

In the example below, `@post` is the model object that needs a form. `form_for`
automatically performs a route lookup to find the right URL for post.

`form_for` takes a block. It passes an instance of [FormHelper][form_helper] as
a parameter to the block, which is what `f` is below.

[form_helper]: http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html

A basic implementation looks like this:

```erb
<!-- app/views/posts/edit.html.erb //-->

<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

This creates the HTML:

```html
<form class="edit_post" id="edit_post" action="/posts/1" accept-charset="UTF-8" method="post">
  <input name="utf8" type="hidden" value="&#x2713;" />
  <input type="hidden" name="_method" value="patch" />
  <input type="hidden" name="authenticity_token" value="nRPP2OqVKB00/Cr+8EvHfYrb5sAkZRtr8f6dzBaJAI+cMceR0fUatcLWd4zdwYCpojW2J3QLK6uyBKeFAgZvmw==" />
  <input type="text" name="post[title]" id="post_title" value="Existing Post Title"/>
  <textarea name="post[content]" id="post_content">Existing Post Content</textarea>
  <input type="submit" name="commit" value="Update Post" />
</form>
```

Here's what we would need to do with `form_tag` to generate the exact same HTML:

```erb
<!-- app/views/posts/new.html.erb //-->

<%= form_tag post_path(@post), method: "patch", name: "edit_song", id: "edit_song" do %>
  <%= text_field_tag "post[title]", @post.title %>
  <%= text_area "post[content]", @post.content %>
  <%= submit_tag "Update Post" %>
<% end %>
```

`form_tag` doesn't know what action we're going to use it for, because it has no
model object to check. `form_for` knows that an empty, unsaved model object
needs a `new` form and a populated object needs an `edit` form. This means we
get to skip all of these steps:

1. Setting the `name` and `id` of the `<form>` element.
1. Setting the method to `patch` on edits.
1. Setting the text of the `<submit>` element.
1. Specifying the root parameter name (`post[whatever]`) for every field.
1. Choosing the attribute (`@post.whatever`) to fill for every field.

Nifty!

# Using `form_for` to generate empty forms

To wire up an empty form in our `new` view, we need to create a blank object:

```ruby
# app/controllers/posts_controller.rb

  def new
    @post = Post.new
  end
```

Here's our usual vanilla `create` action:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.create(post_params)

    redirect_to post_path(@post)
  end
```

We still have to solve the dual problem of what to do when there's no valid
model object to redirect to, and how to hold on to our error messages while
re-rendering the same form.

# Re-Rendering With Errors

Remember from a few lessons ago how CRUD methods return `false` when validation
fails? We can use that to our advantage here and branch our actions based on the
result:

```ruby
# app/controllers/posts_controller.rb

  def create
    @post = Post.new(post_params)

    if @post.save
      redirect post_path(@post)
    else
      render :new
    end
  end
```

# Full Messages with Prepopulated Fields

Because of `form_for`, Rails will automatically prepopulate the `new` form with
the values the user entered on the previous page.

To get some extra verbosity, we can add the snippet from the previous lesson to
the top of the form:

```erb
<!-- app/views/posts/new.html.erb //-->

<% if @post.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@post.errors.count, "error") %>
      prohibited this post from being saved:
    </h2>

    <ul>
    <% @post.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

# More Freebies: `field_with_errors`

Let's look at another nice feature of `FormHelper`. Here's our `form_for`
code again:

```erb
<!-- app/views/posts/edit.html.erb //-->

<%= form_for @post do |f| %>
  <%= f.text_field :title %>
  <%= f.text_area :content %>
  <%= f.submit %>
<% end %>
```

The `text_field` call generates this tag:

```html
<input type="text" name="post[title]" id="post_title" value="Existing Post Title"/>
```

Not only will `FormHelper` pre-fill an existing `Post` object's data, it will
also wrap the tag in a div with an error class if the field has failed
validation:

```html
<div class="field_with_errors">
  <input type="text" name="post[title]" id="post_title" value="Existing Post Title"/>
</div>
```

This can also result in some unexpected styling changes, because `<div>`s are
block tags (which take up the entire width of their container) while `<input>`s
are inline tags. If your layout suddenly gets messed up when a field has errors,
this is probably why.

# Recap

`form_for` gives us a lot of power!

Our challenge as developers is to keep track of the different layers of magic
that makes this tool so convenient. The old adage is true: we're responsible for
understanding not only *how* to use `form_for`, but also *why* it works.
Otherwise, we'll be completely lost as soon as a sufficiently unusual edge case
appears.

When in doubt, **read the HTML**. Get used to hitting the "View Source" and
"Open Inspector" hotkeys in your browser (`Ctrl-u` and `Ctrl-Shift-i` on Chrome
Windows), and remember that most browsers let you [examine POST data in their
developer network tools][devtools].

[devtools]:
http://superuser.com/questions/395919/where-is-the-post-tab-in-chrome-developer-tools-network

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/validations-with-form_for-rails' title='Validations with form_for'>Validations with form_for</a> on Learn.co and start learning to code for free.</p>
