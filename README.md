# ExpoRspec #

## Primero creamos el proyecto 
$ `rails new todos-api --api -T`

## Luego modificamos el gemfile para las hemas que usaremos 
*	`group :development, :test do  
	  gem 'rspec-rails', '~> 3.5'  
	end`
*  `group :test do`\
      `gem 'factory_bot_rails', '~> 4.0'`\
      `gem 'shoulda-matchers', '~> 3.1'`\
      `gem 'faker'`\
      `gem 'database_cleaner'`\
`end`

## Instalamos las gemas el Gemfile
$	`bundle install`
## Installamos RSpec
$ `rails generate rspec:install`
## Crear una carpeta que nos ayudara mas adelante
$ `mkdir spec/factories`
## agregamos la siguiente configuracion en spec/rails_helper.rb
`require 'database_cleaner'`

`Shoulda::Matchers.configure do |config|`\
  `config.integrate do |with|`\
    `with.test_framework :rspec`\
    `with.library :rails`\
  `end`\
`end`

## En el mismo archivo dentro de la funcion `RSpec.configuration do |config|`

`config.include FactoryBot::Syntax::Methods`
  `config.before(:suite) do`\
    `DatabaseCleaner.clean_with(:truncation)`\
    `DatabaseCleaner.strategy = :transaction`\
  `end`\
  `config.around(:each) do |example|`\
    `DatabaseCleaner.cleaning do`\
      `example.run`\
    `end`\
  `end`

## Creamos los modelos y corremos las migraciones generadas
$`rails g model Todo title:string created_by:string`\
$`rails g model Item name:string done:boolean todo:references`\
$`rails db:migrate`

## agregamos la siguiente configuracion en  spec/models/todo_spec.rb

`it { should have_many(:items).dependent(:destroy) }`\
  `it { should validate_presence_of(:title) }`\
  `it { should validate_presence_of(:created_by) }`

## de forma similar para  spec/models/item_spec.rb

`it { should belong_to(:todo) }`\
`it { should validate_presence_of(:name) }`

## ejecutamos el siguiente comando para hacer la prueba 

$`bundle exec rspec`


## en el modelo de todo app/models/todo.rb

`has_many :items, dependent: :destroy`\
`validates_presence_of :title, :created_by`
## ahora en el modelo item app/models/item.rb
`belongs_to :todo`\

 `validates_presence_of :name
  
## ejecutamos el siguiente comando para hacer la prueba 

$`bundle exec rspec`

## ahora probaremos con controladores
$ `rails g controller Todos`\
$ `rails g controller Items`

## crearemos pruebas para peticiones, por orden se creara la siguiente carpeta y los siguientes archivo
$ `mkdir spec/requests && touch spec/requests/{todos_spec.rb,items_spec.rb}`
$ `touch spec/factories/{todos.rb,items.rb}`

## en el archivo spec/factories/todos.rb 
```
FactoryBot.define do
  factory :todo do
    title { Faker::Lorem.word }
    created_by { Faker::Number.number(10) }
  end
end
```
