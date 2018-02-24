---
layout: post
title:      "Devit on Rails"
date:       2018-02-24 18:02:06 +0000
permalink:  devit_on_rails
---


Devit is a link aggregator site, similar to Reddit or Hacker News, where a user can submit a link and content to a community, as well as comment on and vote on those submissions.


`rails new devit` ! 

### Authentication with Devise and OmniAuth with a GitHub strategy

The [Devise and OmniAuth Authentication with Rails](https://prbsas.github.io/devise_and_omniauth_authentication_with_rails) post I wrote describes how I applied them both to authenticate in `devit`.   

### Models and Associations 

The application has a series of models with `has_many` and `belongs_to` associations, as well as `has_many :trough` ones.

The User model makes use of `source:` to name the association between User and Post as `:contributions`

```
class User < ApplicationRecord
  has_many :communities

  has_many :posts
  has_many :contributions, through: :posts, source: :community

  has_many :comments

  has_many :user_flairs
  has_many :flairs, through: :user_flairs
end 
```


The Community model specifies a custom `class_name` and `foreign_key` for the User class to reference it as poster. A Community `belongs_to` its `poster`. 

```
class Community < ApplicationRecord
  belongs_to :poster, class_name: 'User', foreign_key: 'user_id'

  has_many :posts, dependent: :destroy
  has_many :posters, through: :posts, source: :user
end
```

```
class Post < ApplicationRecord
  belongs_to :user
  belongs_to :community

  has_many :comments, dependent: :destroy

  has_many :tags
  accepts_nested_attributes_for :tags, reject_if: :all_blank
end
```

A Comment `belongs_to` a Post and a User. The comment table is a join table.

```
class Comment < ApplicationRecord
  belongs_to :post
  belongs_to :user
end
```

A Tag (or category) `belongs_to` a Post, and a Post `has_many` tags.

```
class Tag < ApplicationRecord
  belongs_to :post
end
```


```
class Flair < ApplicationRecord
  has_many :user_flairs
  has_many :users, through: :user_flairs
end
```

A UserFlair `belongs_to` a User and a Flair. The `user_flairs` table is a join table that adds an extra attribute, `experience_level`. 

```
class UserFlair < ApplicationRecord
  belongs_to :user
  belongs_to :flair
end
```

### Scope 

Through a scope, the Post model adds a class method for retrieving the most recent posts.

```
class Post < ApplicationRecord
  scope :recent, -> { order('posts.created_at DESC').limit(10) }
end 
```

To make this feature visible a route and controller action were added:

`get 'posts/recent', to: 'posts#recent'`

```
class PostsController < ApplicationController
  def recent
    @recent_posts = Post.recent
    render :recent
  end
end 
```

As well as a view, that uses the `_post` partial to render:

```
<h1>Recent Posts</h1>

<section id="list-posts" class="recent">
  <ul>
    <% @recent_posts.each do |post| %>
      <%= link_to community_post_path(post.community, post) do %>
        <%= render partial: 'post', locals: { post: post } %>
      <% end %>
    <% end %>
  </ul>
</section>
```


### Custom Attribute Writer 

The User model includes a custom attribute writer responsible for finding and creating a flair by name through a nested form in `users/edit`.

```
class User < ApplicationRecord

  def flairs_attributes=(flairs_attributes)
    flairs_attributes.values.each do |flairs_attribute|
      if flairs_attribute['name'] != nil
        flair = Flair.find_or_create_by(name: flairs_attribute['name'])
        self.user_flairs.build(flair: flair, experience_level: flairs_attribute['user_flairs']['experience_level'])
      end
    end
  end
end
```

Params hash for updating a User:

```
{"utf8"=>"✓",
 "_method"=>"patch",
 "authenticity_token"=>"PcWvSoJCNeT9XoHNwGNMhAp2XnEYto+P5f2VFYCkCth2tDafYi/eKGFXw10tUgis1hTxV+CA81sqCkooy1taCw==",
 "user"=>
  {"name"=>"Paula Ramirez Pitzen",
   "username"=>"PRbsas",
   "avatar_url"=>"https://avatars0.githubusercontent.com/u/19231820?v=4",
   "bio"=>"Architect - Designer - Web Developer",
   "github_profile_url"=>"https://github.com/PRbsas",
   "flairs_attributes"=>{"0"=>{"name"=>"Ruby on Rails", "user_flairs"=>{"experience_level"=>"master"}}}},
 "commit"=>"Update User",
 "id"=>"11”}
```
 
 
### Gems 

```
gem 'friendly_id', '~> 5.1.0'
gem 'faker'
gem 'acts_as_votable', '~> 0.11.1'
gem 'dotenv-rails'
```

Devit uses the FriendlyID gem to make URLs clean and human-readable. Read the [Friendly URLs for your Rails App with FriendlyId](https://prbsas.github.io/friendly_urls_for_your_rails_app_with_friendlyid) post where I explain how I implemented it in the app.  

The Faker gem is used to create dummy records to seed the database. In `app/db/seeds.rb` I used it to create Users.

```
10.times do
  User.create(
    email: Faker::Internet.email,
    name: Faker::Name.name,
    username: Faker::Internet.user_name,
    bio: Faker::Matz.quote,
    github_profile_url: Faker::Internet.url("github.com"),
    avatar_url: Faker::Avatar.image,
    password: Faker::Internet.password(8)
  )
end
```

The Acts As Votable gem allows a model to vote and be voted on. See my separate post, [Acts As Votable gem for Rails Apps](https://prbsas.github.io/acts_as_votable_gem_for_rails_apps), which goes through the process of adding this functionality to devit.

Dotenv is a gem to load environment variables from `.env`. In the app it stores GitHub's key and secret to use for authentication. 


### Next 

Some features I want to add are:  
* A User activity page. Where users can see their latest contributions to each community, be it posts, comments or likes/upvotes.  
* Habillity to list all Users with the same Flair, or same Experience Level.  
* A way for users to JOIN Communities and be able to list their communities.
