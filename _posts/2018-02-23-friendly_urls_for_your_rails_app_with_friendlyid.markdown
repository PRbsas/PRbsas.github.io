---
layout: post
title:      "Friendly URLs for your Rails App with FriendlyId"
date:       2018-02-23 19:52:16 +0000
permalink:  friendly_urls_for_your_rails_app_with_friendlyid
---

 
[FriendlyId](https://github.com/norman/friendly_id) is an add-on to Ruby's ActiveRecord that allows you to create clean, human-readable URLs and work with strings as if they were numeric ids:

#### without FriendlyId `http://example.com/posts/4323454`
#### with FriendlyId `http://example.com/posts/awesome-post`  

### How ?

The gem uses **slugs** and **finders** to generate human-readable URLs.  
##### Slugs are the part of a URL that identifies a page in human-readable keywords.

##### Finders: Slugs Act Like Numeric IDs, FriendlyId lets you treat text-based identifiers like normal IDs.  

FriendlyId finders will search for your record by friendly id, and fall back to the numeric id if necessary. 

By setting up FriendlyId the following methods will be available to you:  

```
Post.friendly.find('awesome-post') #=> works
Postt.friendly.find(16)            #=> also works
Post.find(16)                      #=> still works
Post.find('awesome-post')          #=> will not work  
```  

You can check out the methods made available through this gem [here, in the Method List tab.](http://norman.github.io/friendly_id/file.Guide.html)

### Setup 

First you'll have to add the [gem](https://rubygems.org/gems/friendly_id/versions/5.2.3) to your Gemfile: `gem 'friendly_id', '~> 5.1'`.  
  

And run `bundle install` in your terminal.  

### Migrations

Then you'll need to generate the friendly configuration file by running `rails generate friendly_id`.  
  
This will create the a file called friendly_id in your initializers directory through a migration. 

Run the migration with `rake db:migrate`. (If you're using Rails 5.1+, before running the migration go to the generated migration file and specify the Rails version: `class CreateFriendlyIdSlugs < ActiveRecord::Migration[5.1]`. 

You'll need to run a migration to add the slug attribute to your model: `rails g migration AddSlugToPosts slug:string`  

Your migration will look something like this: 

```
class AddSlugToPosts < ActiveRecord::Migration[5.1]
  def change  
    add_column :posts, :slug, :string, after: :id  
  end  
end  
```

Run migrations once more: `rake db:migrate`.

### Models

Edit `app/models/post.rb` to extend or include the FriendlyId module and invoke the friendly_id method:

```
class Post < ApplicationRecord
  extend FriendlyId
  friendly_id :title, use: [:finders, :slugged]
end
```

You can set the defaults for the gem in its initializer, so you don't need to keep referencing them in every model, making them more DRY. In `app/config/initializers/friendly_id.rb` add the following: 

``` 
config.use :finders  
config.use :slugged  
```

Your model should now look like this: 

```
class Post < ApplicationRecord
  extend FriendlyId
  friendly_id :title  
end   
```  

### Controllers 

Edit `app/controllers/posts_controller.rb` and replace `Post.find` with `Post.friendly.find`:

```
class PostsController < ApplicationController
  def show
    @post = Post.friendly.find(params[:id])
  end
end  
```  

Now when you create a new post: `Post.create(title: 'awesome-post')`, your URL will be: `http://example.com/posts/awesome-post`.

### Next 

This was a basic setup to get started. FriendlyId offers many more advanced features including: slug history and versioning, i18n, scoped slugs, reserved words, and custom slug generators. You can check them out in the [FriendlyId Guide.](http://norman.github.io/friendly_id/file.Guide.html)
