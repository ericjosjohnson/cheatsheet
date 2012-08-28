# Routings
Two purpose:

- recognize URLs and dispatch them to a controller's action.
- generate paths and URLs in views.

## Resource Routings
`resources :photos` defines seven routes in applications.

      GET /photos index
      GET /photos/new new
      POST /photos create
      GET /photos/:id show
      GET /photos/:id/edit edit
      PUT /photos/:id update
      DELETE /photos/:id destroy

It also generates four path helpers:

    photos_path
    new_photo_path
    edit_photo_path(:id)
    photo_path(:id)

Singular Resources is used for resource which is looked up without an id

`resource :coder` generates six routes, without the listing route:

    GET /coder/new new
    GET /coder show
    POST /coders create
    GET /coder/edit edit
    PUT /coder update
    DELETE /coder destroy

The path helpers:

    coder_path
    new_coder_path
    edit_coder_path

### Customized Resource
Use `:controller` to declare controller to be used.

Use `:as` to declare the name of path generator.

Use `:constraints` to specify the constrains on parameters.

Use `:path_names` to override default pathname.

Use `:only` and `:except` to restrict the routes created.

### Additional Resource
For individual member, use the `member` block:

    resources :photos do
      member do
        get 'preview'
      end
    end

It maps the url `/photos/1/preview` to the `preview` action with GET in `PhotosController`.

For collection member, use the `collection` block:

    resources :photos do
      collection do
        get 'search'
      end
    end

It maps the url `/photos/search` to the `search` action with GET in `PhotosController`.

### Nesting and Namespace
use `namespace` to group resources to mapped controllers.

```ruby
namespace :admin do
  resources :posts
end
```

It maps url prefixed with `/admin` and maps to `Admin::PostsController`.

Use `scope :module => 'name'` to map non-prefixed url to controller nested within a module.

```ruby
scope :module => 'admin' do
  resources :posts
end
```

It maps normal url to `Admin::PostsController`

Use `scope 'admin' do` to map prefixed url to non-modulized controller.

```ruby
scope '/admin' do
  resources :posts
end
```
It maps url prefixed with `/admin` to `PostsController`

You could nest resources

```ruby
resources :magazines do
  resources :ads
end
```

### Path Generator
The `url_for` could be used as a shortcut to generate path, it will guess the proper path generator you should use.

```erb
<%= link_to "Ad details", url_for([@magazine, @ad]) %>
# ommit the url_for implicit call
<%= link_to "Ad details", [@magazine, @ad] %>
```

## Arbitrary URLs
Two special simbols are used to map to the controller and action, `:controller` and `:action`

Use `()` to declare optional paths

    match ':controller(/:action(/:id))'


You could declare routes explicitly by pass match an hash

    match 'photos/:id' => 'photos#show', :defaults => { :format => 'jpg' }

Use `:via` to declare HTTP methods

    match 'photos/show' => 'photos#show', :via => [:get, :post]

    # or with as a shortcut

    get 'photos/show'

Use `:constrains` to set constrains on segment, it could contain:

- Use regular expression one `:id => /\d.+/` to filter parameters
- Constrain based on the methods of Request object

You **cannot** use `:namespace` in this case, instead declare an constrain with regular expression.

    match ':controller(/:action(/:id))', :controller => /admin\/[^\/]+/

A constrains is an object which responds to `matches?`.

Use `*` for wildcard matching.

    match '*a/foo/*b' => 'test#index'

Use `redirect` which also could takes parameters for permanent redirection.

    match "/stories/:name" => redirect {|params| "/posts/#{params[:name].pluralize}" }
    match "/stories" => redirect {|p, req| "/posts/#{req.subdomain}" }

## Tests
`assert_generates` tests from path to controller, action.
`assert_recognizes` tests matches for controller, action and path. 
`assert_routing` to test both ways.
