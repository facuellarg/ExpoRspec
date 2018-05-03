# ExpoRspec #

## Primero creamos el proyecto 
$ `rails new todos-api --api -T`

## Luego modificamos el gemfile para las hemas que usaremos 
*	`group :development, :test do
	  gem 'rspec-rails', '~> 3.5'
	end`
*  `gem 'factory_bot_rails', '~> 4.0'
	  gem 'shoulda-matchers' , '~> 3.1'
	  gem 'faker'
	  gem 'database_cleaner'
	end`

