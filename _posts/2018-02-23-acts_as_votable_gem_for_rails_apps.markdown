---
layout: post
title:      "Acts As Votable gem for Rails Apps"
date:       2018-02-23 20:29:51 +0000
permalink:  acts_as_votable_gem_for_rails_apps
---


[Acts As Votable](https://github.com/ryanto/acts_as_votable/) is a Ruby Gem written for Rails ActiveRecord. This gem adds methods to your model that allow it to vote or be voted on while providing an easy syntax to read and write. This makes it easy to enable user interaction with the content of your application.

## Getting started
 
### Setup

First you'll need to add the [gem](https://rubygems.org/gems/acts_as_votable/versions/0.11.1) to your Gemfile: `gem 'acts_as_votable', '~> 0.10.0'`.   

And run `bundle install` in your terminal. 

### Migrations 

Acts As Votable uses a votes table to store all the voting information, so you'll need to generate a migration and migrate the database as well:

```
rails generate acts_as_votable:migration
rake db:migrate
```

The migration generated:

```
class ActsAsVotableMigration < ActiveRecord::Migration[4.2]
  def self.up
    create_table :votes do |t|

      t.references :votable, :polymorphic => true
      t.references :voter, :polymorphic => true

      t.boolean :vote_flag
      t.string :vote_scope
      t.integer :vote_weight

      t.timestamps
    end

    add_index :votes, [:voter_id, :voter_type, :vote_scope]
    add_index :votes, [:votable_id, :votable_type, :vote_scope]
  end

  def self.down
    drop_table :votes
  end
end
```
 
### Models

To make a model votable you will need to add to it the `acts_as_votable` helper:
 
```
class Post < ApplicationRecord
  acts_as_votable
end
```

You can have models `act_as_voters`, this will provide them with more helper methods.  

```
class Post < ApplicationRecord
  acts_as_voter
end
```

### Routes

In `app/config/routes.rb` you'll need to add a nested block to the votable model routes, with upvote and downvote actions: 

```
resources :posts do
  member do
    put 'like', to: 'posts#upvote'
    put 'dislike', to: 'posts#downvote'
  end
end
```

This will generate two new routes for you:

``` 
Prefix        Verb   URI Pattern      			    Controller#Action
like_post     PUT    /posts/:id/like(.:format)      posts#upvote
dislike_post  PUT    /posts/:id/dislike(.:format)   posts#downvote
```

### Controllers

In the controller we will need to define the `#upvote` and `#downvote` actions:

```
def upvote
	@post = Post.find(params[:id])
    @post.upvote_by current_user
    redirect_back fallback_location: root_path
  end
```

```
def downvote
	@post = Post.find(params[:id])
    @post.downvote_by current_user
    redirect_back fallback_location: root_path
  end
```

The `current_user` helper method is provided by [Devise](https://github.com/plataformatec/devise) to check for the current signed-in user. (Devise is a flexible authentication solution for Rails applications).  

The user votes and gets redirected to the current page with `redirect_back fallback_location: root_path`.

### Views

Lastly, making this functionality visible in the browser:

```
<%= link_to like_post_path(@post), method: :put do %>
   <div class="engagement">
     <%= image_tag("up") %>
     <p><%= @post.get_upvotes.size %> </p>
   </div>
<% end %>
```

```
<%= link_to dislike_post_path(@post), method: :put do %>
   <div class="engagement">
     <%= image_tag("down") %>
     <p><%= @post.get_downvotes.size %> </p>
   </div>
<% end %>
```
  
### Next 

This gem provides lots of helper methods, for example: you can count the upvotes and downvotes a model has.
 
```
@post.votes_for.size # => 10

@post.get_likes.size # => 7
@post.get_upvotes.size # => 7

@post.get_dislikes.size # => 3
@post.get_downvotes.size # => 3
```


You can find more useful methods in the [Acts As Votable GitHub Repo](https://github.com/ryanto/acts_as_votable/).

