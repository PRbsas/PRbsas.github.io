---
layout: post
title:      "Devise and OmniAuth Authentication with Rails"
date:       2018-02-23 23:28:05 +0000
permalink:  devise_and_omniauth_authentication_with_rails
---


## Devise

[Devise](https://github.com/plataformatec/devise) is a flexible authentication solution for Rails. Being a Rails engine, it's like a miniature application that provides functionality to yours. It provides its own routes, controllers and views. It is composed of 10 modules, which can be applied to your model to extend functionality. One of these modules provides Omniauth support to authenticate with other providers. This makes it a great solution for app authentication, as you can provide your users with different methods to authenticate.

#### Setup

You'll need to add a gem to your Gemfile: `gem 'devise'` and run `bundle install`. 

Run the generator: `rails generate devise:install`.

This generator will install an initializer which describes all of Devise's configuration options in `app/config/initializers.devise.rb`, and at this point some instructions to make some manual setup will appear in the terminal. 

We'll have to define default url options in `config/environments/development.rb`.
 
```
#Devise: define default url options 
#(In production, :host should be set to the actual host of the application.)

config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```

Add flash messages in `app/views/layouts/application.html.erb`.

```
<p class="notice"><%= notice %></p>
<p class="alert"><%= alert %></p>
```

Copy Devise views (for customization) to the app by running `rails g devise:views`

Make sure you have defined a root_url or any other route in `app/config/routes.rb`.

After this initial setup we can generate the Devise User by running: `rails generate devise User`. This will create a User model if it doesn't exist and add `devise_for :users` routes to the `app/config/routes.rb` file.

Run `rake db:migrate`. And restart your server.

#### Models 

The User model will have several modules added to it by default:
```
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
end         
``` 

#### Routes

In your `app/config/routes.rb`, `devise_for :users` will have added all the necesary routes:

```
              new_user_session GET      /users/sign_in(.:format)         devise/sessions#new
                  user_session POST     /users/sign_in(.:format)         devise/sessions#create
          destroy_user_session DELETE   /users/sign_out(.:format)        devise/sessions#destroy
             new_user_password GET      /users/password/new(.:format)    devise/passwords#new
            edit_user_password GET      /users/password/edit(.:format)   devise/passwords#edit
                 user_password PATCH    /users/password(.:format)        devise/passwords#update
                               PUT      /users/password(.:format)        devise/passwords#update
                               POST     /users/password(.:format)        devise/passwords#create
      cancel_user_registration GET      /users/cancel(.:format)          devise/registrations#cancel
         new_user_registration GET      /users/sign_up(.:format)         devise/registrations#new
        edit_user_registration GET      /users/edit(.:format)            devise/registrations#edit
             user_registration PATCH    /users(.:format)                 devise/registrations#update
                               PUT      /users(.:format)                 devise/registrations#update
                               DELETE   /users(.:format)                 devise/registrations#destroy
                               POST     /users(.:format)                 devise/registrations#create
```
                               
#### Controllers 

Devise helpers are now available to us! Next, we need to set up the application controller with user authentication, by adding `before_action :authenticate_user!` after `protect_from_forgery with: :exception` to avoid any issues.

```
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :authenticate_user!, :except => [:home]
end
```

Two great helpers (and many more) are made available by Devise:  

```
user_signed_in?
#verifies if a user is signed in

current_user
#for the current signed-in user
```

## OmniAuth: Standardized Multi-Provider Authentication 

[Omniauth](https://github.com/omniauth/omniauth) is a flexible authentication system that allows to easily integrate multiple authentication providers (refered as strategies), into an application. This means a user can sign in to your application through FaceBook, Twitter, GittHub, and many others. You can find a list of strategies available [here](https://github.com/omniauth/omniauth/wiki/List-of-Strategies).

### OmniAuth with GitHub

#### Setup 

You'll need to add a two gems to your Gemfile: `gem 'omniauth'` and `gem 'omniauth-github'` and run `bundle install`.

Next up, you'll need to generate a migration to add two columns to your users table and migrate the database. 

```
rails g migration AddOmniauthToUsers provider:string uid:string
rake db:migrate
```

In `config/initializers/devise.rb` add OmniAuth configuration:

`config.omniauth :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET']`

You will need to head to your **GitHub account Developer settings** and click on **New OAuth App** to register your application and have a `GITHUB_KEY` (Client ID) and `GITHUB_SECRET` (Client Secret) assigned. You will be asked for the Application name, Homepage URL, a description and an **Authorization callback URL**. You can use this one: `http://localhost:3000/users/auth/github/callback`.

To protect our key and secret we'll add the `'dotenv-rails'` gem to our Gemfile and run `bundle install`.
Next, add `.env` to the `.gitignore` file and create a `.env` file inside the root of the application:

```
GITHUB_KEY= your GITHUB_KEY(Client ID) 
GITHUB_SECRET= your GITHUB_SECRET(Client Secret)
```

#### Routes

In `app/config/routes.rb` add the following code to handle callbacks.

`devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }`

You'll have two more routes:
```
user_github_omniauth_authorize  GET|POST   /users/auth/github(.:format)           users/omniauth_callbacks#passthru
user_github_omniauth_callback   GET|POST   /users/auth/github/callback(.:format)  users/omniauth_callbacks#github
 ```

#### Controllers

OmniAuth will always return a hash of information after authenticating with an external provider under the key `omniauth.auth`. You can read more about the Auth Hash Schema [here](https://github.com/omniauth/omniauth/wiki/Auth-Hash-Schema).

Add a file `app/controllers/users/omniauth_callbacks_controller.rb` to handle the callbacks.


```
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def github
    @user = User.from_omniauth(auth_hash)
    if @user.persisted?
      sign_in_and_redirect @user
      set_flash_message(:notice, :success, kind: 'GitHub') if is_navigational_format?
    else
      session['devise.github_data'] = request.env['omniauth.auth']
      redirect_to new_user_registration_url
    end
  end

  def failure
    redirect_to root_path
  end

  protected

  def auth_hash
    request.env['omniauth.auth']
  end
end
```

Update your ApplicationController with the following private method: 

```
class ApplicationController < ActionController::Base

  protected
    def after_sign_in_path_for(resource)
      request.env['omniauth.origin'] || stored_location_for(resource) || root_path
    end
end
```

#### Models

In the User model we will need to enable OmniAuth as well, by adding a Devise module and writting a class method to generate users from OmniAuth. Here you can choose what information you want to add to your users.

```
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable,
         :omniauthable, :omniauth_providers => [:github],
         :authentication_keys => {email: false, login: true}


  def self.from_omniauth(auth)
      where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.email = auth.info.email
        user.password = Devise.friendly_token[0,20]
        user.name = auth.info.name
        user.username = auth.info.nickname
        user.bio = auth.extra.raw_info.bio
        user.github_profile_url = auth.info.urls.GitHub
        user.avatar_url = auth.info.image
    end
  end
end 
```
  

##### That's it! You now have an application that offers different ways of authentication to it's users, well done!
