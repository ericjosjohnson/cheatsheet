## Rails Server
`rails s` is an alias to `rails server`

`rails s -e production -p 4000` changes the development environment and port

`rails s -d` runs the server as a daemon

### Dependencies
In `Gemfile`, add `gem "thin", ">= 1.4.1"` will change the default server to *Thin*.

## Rails Console
`rails c` is an alias to `rails console`

`rails console --sandbox` to test out code without changing data.

`rails db` is an alias to `rails dbconsole`

`rails runner` will run code in Rails context, it is useful to create crontab tasks.

## Rails Generate
`rails g` is an alias to `rails generate`

`rails d` is an alias to `rails destory` which is the opposite of `rails generate`

`rails generate` list possible generator

`rails generate <controller|model>` list help for native Rails generator

`rails generate -p ...` to dry-run the generator

`rails generate controller NAME [action action]` could:
- generate controller file and corresponding actions
- corresponding view templates
- assets for this controller
- write routes in `route.rb`
- controller tests
- helpers

`rails generate model NAME [field[:type][:index] field[:type][:index]]` could:
- generate **migrates** represents fields passed in
- a model class inherited from `ActiveRecord::Base`
- model tests
- model fixtures / factories

`rails generate scaffold NAME [field[:type][:index] field[:type][:index]]` could:
- generate models similar to `rails generate model`
- generate controller with `index/show/new/update/create/destroy` actions
- generate corresponding views for each action
- helpers
- model/controller/view/**routes** tests
- assets

`rails generate generator GeneratorName`
- generate generator in `lib/generators/<generator_name>` folder
- a source root for the templates

### Dependencies
In `Gemfile`, add `gem "rspec-rails"` will change:
- invoke `rspec` instead of `test_unit` when generates controller and model
- new generator `rspect:install` to generate `/spec` and `/spec/spec_helper.rb`

Add `gem "factory_girl_rails"` will change:
- invoke `factory_girl` instead of making fixtures.

# Write Own Generator
`generators` are built on top of `Thor`.

They are stored in `lib/generators/`

When resolve the generators' name, they are searched the `task_generator.rb` against:
`generators/rails/<task>`
`generators/<task>`
`generators/rails`
`generators/`

Customized generator could be hooked to the rails scaffold work flow with:

    config.generators do |g|
      g.helper          :my_helper

      g.fallbacks[:should] = :test_unit
    end

Within each generator files, they are local methods to access the arguments:
    file_name
    class_name
    plural_name

Rails add additional methods on top of `Thor` to perform rails-specific tasks:

    plugin
    gem
    gem_group
    add_source
    application
    git
    vendor
    lib
    rakefile
    initializer
    generate
    rake
    capify!
    route
    readme
