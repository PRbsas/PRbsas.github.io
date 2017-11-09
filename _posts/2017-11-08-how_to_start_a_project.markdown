---
layout: post
title:      "How to start a project? "
date:       2017-11-09 00:37:50 +0000
permalink:  how_to_start_a_project
---

## A walkthrough of my CLI GEM

 After considering a couple of ideas, I decided I wanted to build an application that showed posts from The DEV Community [dev.to](https://dev.to/), a website where programmers share ideas and help each other grow. 

The approach I took was the one recommended by Avi in the CLI Gem Walktrough video, I took it one step at a time. 

First, to plan my gem I started imagining the interface, what I wanted the user to see and experience. After welcoming the user I decided the next step should be to display a list of post titles, with their authors and tags. After this I wanted to give the user the option to either read one of the listed posts, list them one more time or exit the program. I Knew I wanted to have those options available at any time.

The next step was to define the project structure. I used Bundler to do this, following some steps from this guide: [Developing a RubyGem using Bundler](https://bundler.io/v1.13/guides/creating_gem). I ran `bundle gem dev_to` in my terminal and that built all of the structure for me in advance. I added a config folder with an `environment.rb` file where I later required all my files and gems.

After this I started working on the entry point of my application, the file run. I added `devto` to the bin folder and added permisions to execute it in the terminal by entering `chmod +x dev_to`. This way the program runs through bash using ruby's interpreter. 

This was the moment to start thinking about the code I wish I had. I added `DevTo::CLI.new.call` to `./bin/devto`. This line of code represented the decision to encapsulate all of the CLI interface logic into one object, the CLI class, my Controller. I started working on it following the guide I had set for myself in the beginning. I used harcoded data to be able to quickly see how the interface was shaping up. 

Once I had a working interface I moved on to making this dummy data real. I defined two more classes, the `Post` class and the `Scraper` class. In the `Post` class I defined all the attributes my Post objects were going to have: title, author, tags, date, a url and content. The `Scraper` class was going to be responsible for getting all this data from the dev.to website and make instances of Posts.

I then made these three classes collaborate with each other. The `Post` class was going to be responsible for keeping track of all the instances of Post created by the `Scraper` class. This allowed me to define a class method for `Post` to find instances by id, and show its content to the user. As I had to call the `Scraper` method to make posts at the beginning of the program to make the instances of Post, I determined it was better to define another method to scrape the content, as I only needed to show the content once the user chose a post to read. I grabbed the user input to find the post's url in the `Scraper` make content method, open it with Nokogiri and open-uri and scrape the content and date of the post.  

While working with Nokogiri to scrape the data, I found I wanted to show the user the amount of comments and likes each post had, so I added those attributes in the `Post` class (I used the attr_accessor for all the attributes of `Post`, to be able to read and write to them). I then scraped this data (The Chrome DevTools were, as always, extremely useful in this process) and added it to the CLI method to list the posts.  

I wanted the user to see emojis next to the comments and likes count, as displayed in the website. So I started researching how to make that possible and found I could use UNICODE emoji codes. I was able to add them to my code like this: `\u{1F4AC}` (The numbers and letters inside are specific for each emoji). Another thing I wanted to try was to let the user open the selected post in the browser to keep reading. I was able to achieve this with this code: `system("open #{post.url}")`.  
 
Once all of this was working I decided to try and publish the gem to [rubygems.org](https://rubygems.org). I followed a guide from the same website to do this, [Publishing your gem](http://guides.rubygems.org/publishing/). Now you can find my gem [here!](https://rubygems.org/gems/dev_to).  

Next steps I have in mind are adding new features, like searching posts by #tag, and write tests. I learned a lot making this gem and loved every step of it, looking forward to making many more.
