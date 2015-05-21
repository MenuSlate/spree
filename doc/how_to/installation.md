# Installation
## Alternatives
### Building a Sandbox Application
Spree includes a helpful Rake task for setting up a sample application:
* Clone the Git repo
```shell
git clone git://github.com/spree/spree.git
cd spree
```
* Install the gem dependencies
```shell
bundle install
```
* Create a sandbox application for testing purposes (and automatically perform all necessary database setup)
```shell
bundle exec rake sandbox
```
* Start the server
```shell
cd sandbox && rails server
```

This creates a barebones rails application configured with the Spree gem. It runs the migrations  for you and sets up the sample data. The resulting `sandbox` folder is already ignored by `.gitignore` and it is deleted and rebuilt from scratch each time the Rake task runs.

### Use as a Gem with Existing Application
* To use a stable branch of Spree, add this line to your Gemfile
```
gem 'spree', github: 'spree/spree', branch: '3-0-stable'
```
* To use the bleeding edge version of Spree:
```
gem 'spree', github: 'spree/spree'
```
* Install these gems using this command:
```shell
bundle install --auto-accept
```
* Use the install generator to set up Spree:
```shell
rails g spree:install --sample=false --seed=false
```
* You can avoid running DB migrations and or generating seed and sample data by passing these flags:
```shell
rails g spree:install --migrate=false --sample=false --seed=false
```
* To perform those on your own:
```shell
bundle exec rake railties:install:migrations
bundle exec rake db:migrate
bundle exec rake db:seed
bundle exec rake spree_sample:load
```

#### Browse Store
[http://localhost:3000](http://localhost:3000)

#### Browse Admin Interface
[http://localhost:3000/admin](http://localhost:3000/admin)

Username `spree@example.com` and password `spree123`

> If you elected not to use the `--auto-accept` option when you added Spree to your Rails app, and  did not install the seed data, the admin user will not yet exist in your database. You can run   a simple rake task to create a new admin user.

> ```bash
> rake spree_auth:admin:create
> ```

## Performance
Rails in development mode continuously reloads objects on each request. The asset pipeline made  things worse. To speed up performance in development mode when NOT working on front end issues:
* In your `config/development.rb` add:
```
config.assets.debug = false
```
* Precompile your assets:
```
RAILS_ENV=development bundle exec rake assets:precompile
```
* To remove precompiled assets (especially before committing to Git):
```
RAILS_ENV=development bundle exec rake assets:clean
```

## Upgrade
Before updating, you will want to ensure the installed spree gem is up-to-date by modifying `Gemfile` to match the new spree version and run `bundle update`.

Thanks to Rails 3.1 Mountable Engine, the update process is "non-destructive" than in previous versions of Spree. The core files are encapsulated separately from sandbox, thus upgrading to newer files will not override nor replace sandbox's customized files.

This makes it easier to see when and how some file has changed � which is often useful if you need to update a customized version.
