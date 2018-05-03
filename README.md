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
$ `mkdir spec/requests && touch spec/requests/{todos_spec.rb,items_spec.rb}`\
$`$ mkdir spec/support && touch spec/support/request_spec_helper.rb`\
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

## en el archivo spec/factories/items.rb
```
FactoryBot.define do
  factory :item do
    name { Faker::StarWars.character }
    done false
    todo_id nil
  end
end
```
## en el archivo spec/requests/todos_spec.rb agregaremos las pruebas para las peticiones 
```
require 'rails_helper'

RSpec.describe 'Todos API', type: :request do
  
  let!(:todos) { create_list(:todo, 10) }
  let(:todo_id) { todos.first.id }

  describe 'GET /todos' do

    before { get '/todos' }

    it 'returns todos' do
    
      expect(json).not_to be_empty
      expect(json.size).to eq(10)
    end

    it 'returns status code 200' do
      expect(response).to have_http_status(200)
    end
  end

 
  describe 'GET /todos/:id' do
    before { get "/todos/#{todo_id}" }

    context 'when the record exists' do
      it 'returns the todo' do
        expect(json).not_to be_empty
        expect(json['id']).to eq(todo_id)
      end

      it 'returns status code 200' do
        expect(response).to have_http_status(200)
      end
    end

    context 'when the record does not exist' do
      let(:todo_id) { 100 }

      it 'returns status code 404' do
        expect(response).to have_http_status(404)
      end

      it 'returns a not found message' do
        expect(response.body).to match(/Couldn't find Todo/)
      end
    end
  end


  describe 'POST /todos' do
    # valid payload
    let(:valid_attributes) { { title: 'Learn Elm', created_by: '1' } }

    context 'when the request is valid' do
      before { post '/todos', params: valid_attributes }

      it 'creates a todo' do
        expect(json['title']).to eq('Learn Elm')
      end

      it 'returns status code 201' do
        expect(response).to have_http_status(201)
      end
    end

    context 'when the request is invalid' do
      before { post '/todos', params: { title: 'Foobar' } }

      it 'returns status code 422' do
        expect(response).to have_http_status(422)
      end

      it 'returns a validation failure message' do
        expect(response.body)
          .to match(/Validation failed: Created by can't be blank/)
      end
    end
  end


  describe 'PUT /todos/:id' do
    let(:valid_attributes) { { title: 'Shopping' } }

    context 'when the record exists' do
      before { put "/todos/#{todo_id}", params: valid_attributes }

      it 'updates the record' do
        expect(response.body).to be_empty
      end

      it 'returns status code 204' do
        expect(response).to have_http_status(204)
      end
    end
  end

  describe 'DELETE /todos/:id' do
    before { delete "/todos/#{todo_id}" }

    it 'returns status code 204' do
      expect(response).to have_http_status(204)
    end
  end
end
```

## en el archivo spec/support/request_spec_helper
```
module RequestSpecHelper
 
  def json
    JSON.parse(response.body)
  end
end
```

## en el archivo spec/rails_helper.rb descomentamos la siguiente linea
`Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }`
### y dentro de la funcion `RSpec.configuration do |config|` agregamos:
`config.include RequestSpecHelper, type: :request`

##corremos el test 
$`bundle exec rspec`

## agregamos las rutas en config/routes.rb
```
 resources :todos do
    resources :items
  end
```

## Corremos 
$`rails routes`
