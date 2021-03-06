---
title: User Profile
description: Access the user profile information using OmniAuth-Auth0 strategy.
budicon: 448
---

The [login](/quickstart/webapp/rails/01-login) step explains how to log users in using the Lock widget and a gem called [OmniAuth](https://github.com/intridea/omniauth) (and a specific Auth0 strategy for OmniAuth). In this step, you'll learn how to access the user profile data once the user has logged into the application.

<%= include('../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-rubyonrails-sample',
  path: '04-User-Profile',
  requirements: [
    'Ruby 2.3.1',
    'Rails 5.0.0'
  ]
}) %>

## The OmniAuth Auth Hash

Using Auth0's OmniAuth strategy, you only need to redirect users to `/auth/oauth2` (which you did in [Login](/quickstart/webapp/rails/01-login)). From there, OmniAuth will take over and take the user through the necessary steps to authenticate them with the Auth0 strategy.

After receiving a successful callback at /auth/oauth2/callback, OmniAuth provides the available user profile information via the request.env['omniauth.auth'] hash.

**NOTE**: The full contents of the authentication hash retrieved by the Auth0 strategy are detailed [here](https://github.com/auth0/omniauth-auth0#auth-hash).

## Add the Dependencies

Add the following dependencies to your `Gemfile` and run `bundle install`:

${snippet(meta.snippets.dependencies)}

## Retrieve the Auth Hash

In the [login](/quickstart/webapp/rails/01-login) step, you configured the application for Lock to start the OmniAuth strategy, and for OmniAuth to take over and complete the authentication process. You also set a route that matches the callback URL in the application routes.

```ruby
get "/auth/oauth2/callback" => "auth0#callback"
```

The `callback` action in the auth0 controller retrieves the auth hash and stores it in the application's session hash. It then redirects to the dashboard controller `show` action, which renders the dashboard view.

```ruby
session[:userinfo] = request.env['omniauth.auth']

redirect_to '/dashboard'
```

## Display the User Profile Data

The auth hash built by the OmniAuth Auth0 strategy has the user profile data under the `info` key. Each element is named according to [the normalized user profile definition](/user-profile/normalized).

The auth hash also contains the full user profile under the `raw` key. The difference is that the normalized user profile data is pre-processed and guaranteed to be present, while as the full user profile data may vary depending on several factors, such as the user's identity provider and its linked profile, among others.

Add the following lines in the dashboard view in order to retrieve the user profile data:

```ruby
<section class="jumbotron  text-center">
  <h2><img class="jumbo-thumbnail img-circle" src="${ '<%= @user[:info][:image] %>' }"/></h2>
  <h1>Welcome, ${ '<%= @user[:info][:name] %>' }</h1>
</section>
<section class="container">
  <div class="panel panel-default">
    <div class="panel-heading">Normalized User Profile</div>
    <div class="panel-body">
      <pre>${ '<%= JSON.pretty_generate(@user[:info]) %>' }</pre>
    </div>
  </div>
  <div class="panel panel-default">
    <div class="panel-heading">Full User Profile</div>
    <div class="panel-body">
      <pre>${ '<%= JSON.pretty_generate(@user[:extra][:raw_info]) %>' }</pre>
    </div>
  </div>
</section>
```

Additionally, declare and assign the contents to the `user` variable in the dashboard controller:

```ruby
class DashboardController < ApplicationController
  include Secured
  def show
    @user = session[:userinfo]
  end
end
```
