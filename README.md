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
	end

## Instalamos las gemas el Gemfile
$	`bundle install`
## Installamos RSpec
$ `rails generate rspec:install`
## Crear una carpeta que nos ayudara mas adelante
$ `mkdir spec/factories`
## agregamos la siguiente configuracion en spec/rails_helper.rb
`require 'database_cleaner'`

Shoulda::Matchers.configure do |config|\
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
`