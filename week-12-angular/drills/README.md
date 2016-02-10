# Drills

### Day 2: Rails + Angular

## Rails Angular Setup

#### Base Application

1. Create a new Rails application with a Postgres database:

 ```zsh
 ➜  rails new rails_angular_sample -Td postgresql
 ➜  cd rails_angular_sample
 ➜  rake db:create
 ```

2. Remove `turbolinks` from your Rails app (`Gemfile`, `application.js`, and `application.html.erb`).

3. Create a `SiteController` with an `index` action. You'll also need to create `site/index.html.erb` inside `app/views`. Your `site#index` will serve as the "layout" for your Angular app.

#### Server Routes

1. Since `site#index` is the "layout" for your Angular app, you want the server to respond with this view every time a route is requested. This will allow Angular to handle routing on the client-side.

 You can use `get '*path'` to send every server-requested route to `site#index`:

 ```ruby
 #
 # config/routes.rb
 #

 Rails.application.routes.draw do
   root 'site#index'
   get '*path', to: 'site#index'
 end
 ```

#### Requiring Angular

1. As you've seen before, there are many ways to require assets in Rails. If you'd like, you can require the CDNs for Angular and `ngRoute` directly in the application layout:

 ```html
 <!-- app/views/layouts/application.html.erb -->

 <!DOCTYPE html>
 <html>
 <head>
   <%= csrf_meta_tags %>
   <meta name="viewport" content="width=device-width, initial-scale=1">
   <base href="/">

   <%= stylesheet_link_tag :application, media: :all %>

   <!-- angular -->
   <script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.min.js"></script>

   <!-- ngRoute -->
   <script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.8/angular-route.min.js"></script>

   <%= javascript_include_tag :application %>

   <title>RailsAngularSample</title>
 </head>
 <body>
   <%= yield %>
 </body>
 </html>
 ```

2. Another option is to download the files for Angular and `ngRoute` and include them in the asset pipeline.  These would go into `vendor`.

 To download via CURL in the Terminal:

 ```zsh
 ➜  curl https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.min.js > vendor/assets/javascripts/angular.min.js
 ➜  curl
 https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.18/angular-ui-router.min.js > vendor/assets/javascripts/angular-ui-router.min.js
 ```

 And require in `application.js`:

 ```js
 /*
  * app/assets/javascripts/application.js
  */

 //= require angular.min
 //= require angular-ui-router.min
 ```

#### Configuring Your Angular App

1. Create a new JavaScript file `app/assets/javascripts/app.js`. This is where you'll put the initial logic for your Angular app.
  * Once you get more than one controller, you should break this up into files like `wine.controller.js`, `wine.factory.js`, `painters.controller.js`.

2. Whether or not you're using the asset pipeline for Angular and `ngRoute`, make sure to require your newly created `app.js` in `application.js` after angular:

 ```js
 /*
  * app/assets/javascripts/application.js
  */

 //= require angular.min
 //= require angular-ui-router.min
 //= require app
 ```

3. Add the `ng-app` directive in the `<html>` tag in the application layout:

 ```html
 <!-- app/views/layouts/application.html.erb -->

 <!DOCTYPE html>
 <html ng-app="SampleApp">
 <head>
   <%= csrf_meta_tags %>
   <meta name="viewport" content="width=device-width, initial-scale=1">
   <base href="/">

   <%= stylesheet_link_tag :application, media: :all %>
   <%= javascript_include_tag :application %>

   <title>RailsAngularSample</title>
 </head>
 <body>
   <%= yield %>
 </body>
 </html>
 ```

4. Add the `ng-view` directive inside the `<body>` tag in `app/views/site/index.html.erb`:

 ```html
 <!-- app/views/site/index.html.erb -->

 <body>
   <nav class="navbar navbar-default">
     ...
   </nav>
   <div data-ng-view></div>
 </body>
 ```

 **Note:** Since this file serves as the "layout" for your Angular app, it's a good idea to place any shared code here, like a navbar.

5. Configure your Angular app in `app.js`:

 ```js
 /*
  * app/assets/javascripts/app.js
  */

 var app = angular.module('SampleApp', ['ui.router']);
 ```

#### Adding Templates

1. You can use the <a href="https://github.com/pitr/angular-rails-templates" target="_blank">angular-rails-templates</a> gem to add your Angular templates to the Rails asset pipeline.

 Require the gem in your `Gemfile`:

 ```ruby
 #
 # Gemfile
 #

 gem 'angular-rails-templates', '~> 0.1.5'
 ```

 **Note:** Version `0.1.5` is necessary to avoid conflicts with the latest version of `sprockets`.

2. Require `angular-rails-templates` in `application.js`, as well as the path to your Angular templates (which you'll create in the next step):

 ```js
 /*
  * app/assets/javascripts/application.js
  */

 //= require angular.min
 //= require angular-route.min
 //= require angular-rails-templates
 //= require_tree ../templates
 //= require app
 ```

 **Note:** Make sure you require your templates AFTER your Angular files and BEFORE `app.js`.

3. Make a `templates` directory inside `app/assets`, and create a template:

 ```
 ➜  mkdir app/assets/templates
 ➜  touch app/assets/templates/home.html
 ```

 At this point, you should `bundle install` and restart your Rails server if you haven't already.

4. Add the `templates` module to your Angular app's dependencies in `app.js`:

 ```js
 /*
  * app/assets/javascripts/app.js
  */

 var app = angular.module('sampleApp', ['ngRoute', 'templates']);
 ```

#### Configuring Angular Routes

1. Configure your Angular routes in `app.js` to hook everything up:

```js
/*
* app/assets/javascripts/app.js
*/

app.config(['$routeProvider', '$locationProvider',
  function ($routeProvider, $locationProvider) {
    $routeProvider
      .when('/', {
        templateUrl: 'home.html',
        controller: 'HomeCtrl'
      })
      .otherwise({
        redirectTo: '/'
      });

    $locationProvider.html5Mode({
      enabled: true,
      requireBase: false
    });
  }
]);
```
```js
config.$inject = ['$stateProvider', '$urlRouterProvider']; // minification protection
function config($stateProvider, $urlRouterProvider) {
  // for any unmatched URL redirect to /
  $urlRouterProvider.otherwise("/");

   $stateProvider
    .state('home', {
      url: "/",
      templateUrl: 'home.html'
    });

}



2. Configure your controller with some test data, so you can check to see if the route, template, and controller are properly connected:

 ```js
 /*
  * app/assets/javascripts/app.js
  */

 app.controller('HomeCtrl', ['$scope', function ($scope) {
   $scope.homeTest = "Welcome to the homepage!";
 }]);
 ```

#### Miscellaneous Setup

1. Since Rails has extra security around submitting forms, there's some setup involved in the `ApplicationController` to allow your Angular app to submit data to the server:

 ```ruby
 #
 # app/controllers/application_controller.rb
 #

 class ApplicationController < ActionController::Base
   # Prevent CSRF attacks by raising an exception.
   # For APIs, you may want to use :null_session instead.
   protect_from_forgery
   after_filter :set_csrf_cookie_for_ng

   protected
     def verified_request?
       super || valid_authenticity_token?(session, request.headers['X-XSRF-TOKEN'])
     end

   private
     def set_csrf_cookie_for_ng
       cookies['XSRF-TOKEN'] = form_authenticity_token if protect_against_forgery?
     end

 end
 ```

#### CRUD

Now that your Angular app is all set up, it's time to CRUD a resource! You'll need:

1. CRUD routes for your resource:

 ```ruby
 #
 # config/routes.rb
 #

 Rails.application.routes.draw do
   root 'site#index'

   namespace :api, defaults: { format: :json } do
     resources :todos, except: [:new, :edit]
   end

   get '*path', to: 'site#index'
 end
 ```

2. A controller with CRUD actions that renders JSON:

 ```ruby
 #
 # app/controllers/api/todos_controller.rb
 #

 class Api::TodosController < ApplicationController
   def index
     @todos = Todo.all.order("created_at DESC")
     render json: @todos
   end

   def create
     @todo = Todo.new(todo_params)
     if @todo.save
       render json: @todo
     else
       render json: { errors: @todo.errors.full_messages.join(", ") }, status: :unprocessable_entity
     end
   end

   ...

   private
     def todo_params
       params.require(:todo).permit(:title, :description, :done)
     end

 end
 ```

3. The Angular `$http` or `$resource` service to interact with your JSON API endpoints. See the module on <a href="https://github.com/sf-wdi-24/modules/tree/master/week-10-angular/day-03/module-02" target="_blank">http and ng-resource</a> for reference. Instead of calling an external API, you're now querying your own application's server.

## Challenges

1. Follow the steps above to set up a new Rails Angular application. Pause once you get a single Angular route (`/`) successfully connected to its template and controller.

## Stretch Challenges

1. Build a JSON API and CRUD one resource (`todos` are always a good start). Test your API routes on Postman.

2. Use Angular `$http` or `$resource` to query your API endpoints from the client side to implement full CRUD in your single-page app.
