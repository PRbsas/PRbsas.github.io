---
layout: post
title:      "React, Redux and Rails API "
date:       2018-05-07 00:07:40 +0000
permalink:  react_redux_and_rails_api
---



`Watched` is a fullstack application to keep track and manage your favorite shows. It is built with a React and Redux frontend and a rails API backend.

## Setup 
#### Rails 5 API 

To generate a new api Rails app run `rails new watched --api` the terminal. This will configure your application to start with a more limited set of middleware than normal, the ApplicationController will inherit from ActionController::API instead of ActionController::Base and it will skip generating views, helpers and assets when you generate a new resource.  
`rails s -p 3001` will launch the rails server in `localhost:3001`.

#### Create React App

At the top level directory of the project run the following in your terminal: `create-react-app client`. This will create a client directory to contain the react application.  
`cd client && yarn start` will launch a development server in `localhost:3000`.

#### Managing multiple processes  

`Foreman` helps you run the various processes needed by your application by declaring them in a Procfile.

To get started install the gem by running: `gem install foreman`, to install the gem, and `touch Procfile` to create a Procfile in your main directory. Here we will declare the processes needed for our application:  

```
web: cd client && yarn start
api: bundle exec rails s -p 3001
```  

The web process will start the client server, and the api process will start the rails api server. In your terminal: `foreman start -p 3000` will lauch the client and the api servers.

#### Creating API Endpoints  

In `config/routes.rb` create routes to `/login` and `/signup`:  

```
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      post '/signup', to: 'users#create'
      post '/login', to: 'sessions#create'
    end
  end
end
```


## Authentication with JSON Web Tokens (JWT) from scratch  

> JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA. [(source)](https://jwt.io/introduction/)


Add `gem 'jwt'` to your Gemfile and run `bundle install`.  
In `config/application.rb` add `config.autoload_paths << Rails.root.join('lib')`.
Make a `/lib` directory and inside it create an auth file: `touch auth.rb`.  
Generate a secret by running `rake secret` in your terminal. Store it as `HMAC_SECRET` securely in a `.env`. 

In `lib/auth.rb` add methods to issue the token and to decode it (we will be using the HMAC algorithm):  

```
require 'jwt'

class Auth
  ALGORITHM = 'HS256'

  def self.issue(payload)
    JWT.encode(payload, hmac_secret, ALGORITHM)
  end

  def self.decode(token)
    JWT.decode(token, hmac_secret, true, { algorithm: ALGORITHM }).first
  end

  def self.hmac_secret
    ENV['HMAC_SECRET']
  end
end
```  


Your sessions controller will use this method to issue a token, after authenticating the user, and render it in json: 

```
class Api::V1::SessionsController < ApplicationController
  skip_before_action :authenticate

  def create
    user = User.find_by(email: user_params[:email])
    if user && user.authenticate(user_params[:password])
      render json: { token: Auth.issue({ user: user.id }) }
    else
      render json: { errors: { message: 'Unable to find user with that email or password' } }, status: 500
    end
  end

  private
  def user_params
    params.require(:user).permit(:email, :password)
  end
end
```  

We can now send a `POST` request with our user and we will get the token in return:  

```
const loginReq = user => (
  fetch(`${base}/login`, {
    method: 'POST',
    body: JSON.stringify(user),
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json'
    }
  }).then((res) => { return res.json() })
)
```

We then store the token in `localStorage`. This property allows you to access a Storage object for the Document's origin and the stored data will be saved across browser sessions.  

```
export function loginUser (user) {
  return function (dispatch) {
    return loginReq(user).then(res => {
      if (res.token !== undefined) {
        window.localStorage.setItem('token', res.token)
        dispatch({ type: LOG_IN_SUCCESS, session: true })
      }
    })
  }
}
```  

## Building User Interfaces with React  

##### Declarative:  
Declarative programming describes the WHAT whereas imperative programming describes the HOW, this allows to focus on what the application should look like while making the code more predictable and easier to debug.

##### Component-Based:  
The use of components allows to separate the user interface into independent and reusable pieces.  

##### Unidirectional Data Flow: 
All the data in the application flows in a single direction, down the tree from the parent to the child.

## Managing State with Redux and Redux Thunk
> Redux is a predictable state container for JavaScript apps. [(redux)](https://github.com/reactjs/redux)
 

> Redux Thunk middleware allows you to write action creators that return a function instead of an action. The thunk can be used to delay the dispatch of an action, or to dispatch only if a certain condition is met. [(redux-thunk)](https://github.com/gaearon/redux-thunk)

### Store  
  
The store is an object that holds the whole state of your application. In `store.js` pass your root reducing function to `createStore` to create it:  

```
import { createStore, applyMiddleware, compose } from 'redux'
import thunk from 'redux-thunk'
import rootReducer from './reducers/index'

export function configureStore () {
  return createStore(
    rootReducer,
    compose(
      applyMiddleware(thunk),
      window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
    )
  )
}

export const store = configureStore()
```

In `index.js` we pass the store to our `App` Component:

```
import React from 'react'
import ReactDOM from 'react-dom'
import { store } from './store'
import { Provider } from 'react-redux'
import App from './App'

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```


### Actions  
  
Dispatch actions (Plain objects describing what happened) to the store, to modify the state. 

```

export const fetchShowInfo = (slug) => {
  return function (dispatch) {
    dispatch({ type: FETCHING_SHOW_INFO })
    return fetchShow(slug).then(res => {
      dispatch({ type: FETCHED_SHOW_INFO, show: res })
    })
  }
}

const fetchShow = slug => (
  fetch(`https://api.trakt.tv/shows/${slug}?extended=full`, {
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json',
      'trakt-api-version': 2,
      'trakt-api-key': process.env.REACT_APP_TRAKT_CLIENT_ID
    }
  }).then((res) => { return res.json() })
)
```

### Reducers  

The store will pass two arguments to the reducer, the current state and the action: 

```
import { FETCHING_SHOWS, FETCHED_SHOWS } from '../actions/search'

export default function searchReducer (state = {
  shows: [],
  isFetching: false
}, action) {
  switch (action.type) {
    case FETCHING_SHOWS:
      return Object.assign({}, state, { isFetching: true })
    case FETCHED_SHOWS:
      return Object.assign({}, state, { shows: action.shows, isFetching: false })
    default:
      return state
  }
}
```  

The `rootReducer` combines the output of multiple reducers:

```
const rootReducer = combineReducers({
  session,
  search,
  show,
  collection
})

export default rootReducer
```  

The Redux store then saves the complete state returned by the root reducer.
