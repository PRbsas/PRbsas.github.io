---
layout: post
title:      "Rails API and JavaScript"
date:       2018-03-10 00:52:19 +0000
permalink:  rails_api_and_javascript
---





## Rails API 
#### Using Active Model Serializers to render customized JSON 

First you’ll need to add the gem to your Gemfile: `gem 'active_model_serializers'` and run `bundle install` in your terminal.  

The gem provides a generator, so we can just setup a serializer for our model running: `rails g serializer community`. This will create `app/serializers/community_serializer.rb`.  

The attributes specified will be returned in JSON. We can add the associations to get those attributes as well. To render specific attributes from associated models we have to explicitly give them a serializer.

```
class CommunitySerializer < ActiveModel::Serializer
  attributes :id, :title, :short_description
  belongs_to :poster, serializer: UserCommunitySerializer
  has_many :posts
end  
```

#### Creating API Endpoints

To respond to requests with different formats we can add a `respond_to` block to our controller action:

```
def index
  @communities = Community.by_created_at
  respond_to do |format|
    format.html { @communities }
    format.json { render json: @communities }
  end
end
```


## and JavaScript
#### Consuming APIs with JavaScript

In `app/assets/javascripts/communities.js`

A listener for load events added to the document object will fire the `getCommunitiesIndex()` function:

```
document.addEventListener('turbolinks:load', () => {
  getCommunitiesIndex()
})
```


The event interface's preventDefault() method, prevents the default behaviour for the click event: 

```
const getCommunitiesIndex = () => {
  $('.list_communities').on('click', (e) => {
    e.preventDefault()
    
		//get JSON from Rails API
  })
}
```


The Fetch API provides an interface for fetching resources using Promises. This enables a simpler and cleaner API while avoiding callback hell.

```
fetch(`/communities.json`, {credentials: 'same-origin'})
    .then((res) => res.json())
    .then(communities => {
    $('.wrapper').html('')
    $('.wrapper').append(renderAddACommunityLink())
    renderCommunityIndex(communities)
    })

```



#### Posting form data with fetch()

To do this we can set the `method` and `body` parameters in the `fetch()` options.
We use the `.serialize()` jquery method to create a text string that will be sent in the `body` of the post request:

```
fetch(url, {
  credentials: 'include', //pass cookies, for authentication
  method: 'post',
  headers: {
  'Accept': 'application/json, application/xml, text/plain, text/html, *.*',
  'Content-Type': 'application/x-www-form-urlencoded; charset=utf-8'
  },
  body: form.serialize()
});
```



#### Translating the JSON response into a JavaScript Model Object with a Constructor function

```
function Community (community) {
  this.id = community.id
  this.title = community.title
  this.description = community.short_description
  this.posterId = community.poster.id
  this.poster = community.poster.username || community.poster.name || community.poster.email
}

let newCommunity = new Community(community)
```


#### Template literals in object prototypes to render to the DOM

Template literals allow for multi-line strings and string interpolation, making them really helpful while dealing with markup:

```
Community.prototype.formatIndex = function () {
  let communityHtml = `
  <section id="list">
    <ul>
      <li>
      <a href="/communities/${this.id}" class="show_community" data-id="${this.id}"><h2>${this.title}</h2></a>
      <p class="description">${this.description}</p>
      <p class="time-ago">created by <a href="/users/${this.posterId}" class="user-tag">${this.poster}<a></p>
      </li>
    </ul>
  </section>
  `
  return communityHtml
}
```
