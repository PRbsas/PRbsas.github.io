---
layout: post
title:      "Behind the scenes, a Sinatra Application "
date:       2017-12-20 22:11:39 +0000
permalink:  behind_the_scenes_a_sinatra_application
---


#### Sinatra
Sinatra is a DSL (Domain Specific Language) implemented in Ruby that's used for writing dynamic web applications. Sinatra is designed to be lightweight and flexible. It is a "Human Interface" to Rack (a powerful middleware between the application and the server).

#### MVC Architecture 
A **M**odel **V**iew **C**ontroller pattern, is used to separate the application's concerns,

* Model: Responsible for the logic of the application. Models interact directly with the database. They store and manipulate data.
* View: This is the only part that the user interacts with directly. Views represent the visualization of the data that the models contain.
* Controller: The go-between for models and views. The controller relays data from the browser to the application, and from the application to the browser.

#### REST
In Sinatra, a route is an HTTP method (get, post, put, delete, patch) paired with a URL-matching pattern. A RESTful route is a route that provides mapping between these methods to controller CRUD actions. 

#### CRUD
**C**reate **R**ead **U**pdate **D**elete. These are the four basic functions of persistent storage.

#### A Sinatra Request 
1. The client (in this case the browser) makes a HTTP request. Sinatra routes it to the correct **Controller** Action.
2. A **Controller** Action asks the models for objects.
3. **Model** asks for the data to the database.
4. The database gives a response.
5. **Model** sends objects to **Controller** Action.
6. A **Controller** Action sends objects to the **View** for rendering.
7. The **View** responds with HTML, CSS and JS that can be read by the browser.
8. A**Controller** Action sends back HTML and HTTP response.

#### ActiveRecord Associations 
Active Record is an Object-Relational Mapper (ORM). It facilitates the creation and use of business objects whose data requires persistent storage to a database, mapping exactly one row in a database to one object in Ruby.

#### Database SQLite 
SQL (Structured Query Language) is a language for managing data in a Database. SQLite is a SQL Database Engine.

#### Sessions and Cookies 
Since HTTP is stateless, to persist data across multiple requests the browser (client) stores the information the server sends in the form of cookies.
Cookies store the data the server sends on each HTTP request in the form of a hash. They are typically used to tell if two requests came from the same browser, for example, for keeping a user logged-in.
Sinatra supports sessions, which are a way for web applications to persist a cookie across multiple HTTP requests. Sessions live on the server. They can be accessed via controllers.

### Paper  
Paper is a web application that allows the user to create Notebooks in which to save **Notes**, **Bookmarks** and **Tasks**.  

The application structure follows the MVC pattern:  
**Models**: User, Notebook, Note, Bookmark, Task.  
**Controllers**: Application Controller, User Controller, Notebook Controller, and Asset Controller (to manage notes, bookmarks and tasks).  
**Views**: Index and Layout, User sign up and login. Notebooks have 4 views, to show all the notebooks, show an individual notebook, create one and edit one. Notes, bookmarks and tasks have views to create each.   
 
The **associations** between the models are as follows, 
 
* A User has_many notebooks and a notebook belongs_to a user.
* A Notebook has_many notes, has_many bookmarks, and has_many tasks. And each one belongs_to a notebook.  

A link for the GitHub repo [here.](https://github.com/PRbsas/paper)
