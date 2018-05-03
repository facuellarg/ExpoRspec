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


## en app/controllers/todos_controller.rb pegamos las siguientes acciones 
```
before_action :set_todo, only: [:show, :update, :destroy]

  # GET /todos
  def index
    @todos = Todo.all
    json_response(@todos)
  end

  # POST /todos
  def create
    @todo = Todo.create!(todo_params)
    json_response(@todo, :created)
  end

  # GET /todos/:id
  def show
    json_response(@todo)
  end

  # PUT /todos/:id
  def update
    @todo.update(todo_params)
    head :no_content
  end

  # DELETE /todos/:id
  def destroy
    @todo.destroy
    head :no_content
  end

  private

  def todo_params
    # whitelist params
    params.permit(:title, :created_by)
  end

  def set_todo
    @todo = Todo.find(params[:id])
  end
```

## creamos un metodo para tener la respuesta en json 

$ `touch app/controllers/concerns/response.rb`

## en el archivo creado app/controllers/concerns/response.rb
```
module Response
  def json_response(object, status = :ok)
    render json: object, status: status
  end
end
```

## para manejar las excepciones 

$ `touch app/controllers/concerns/response.rb`

## en app/controllers/concerns/exception_handler.rb

```
module ExceptionHandler
  # provides the more graceful `included` method
  extend ActiveSupport::Concern

  included do
    rescue_from ActiveRecord::RecordNotFound do |e|
      json_response({ message: e.message }, :not_found)
    end

    rescue_from ActiveRecord::RecordInvalid do |e|
      json_response({ message: e.message }, :unprocessable_entity)
    end
  end
end
```
## en app/controllers/application_controller.rb
```
 include Response
 include ExceptionHandler
```

## corremos el servidos
$ `rails s`

## en el archivo spec/requests/items_spec.rb
```
require 'rails_helper'

RSpec.describe 'Items API' do
  # Initialize the test data
  let!(:todo) { create(:todo) }
  let!(:items) { create_list(:item, 20, todo_id: todo.id) }
  let(:todo_id) { todo.id }
  let(:id) { items.first.id }

  # Test suite for GET /todos/:todo_id/items
  describe 'GET /todos/:todo_id/items' do
    before { get "/todos/#{todo_id}/items" }

    context 'when todo exists' do
      it 'returns status code 200' do
        expect(response).to have_http_status(200)
      end

      it 'returns all todo items' do
        expect(json.size).to eq(20)
      end
    end

    context 'when todo does not exist' do
      let(:todo_id) { 0 }

      it 'returns status code 404' do
        expect(response).to have_http_status(404)
      end

      it 'returns a not found message' do
        expect(response.body).to match(/Couldn't find Todo/)
      end
    end
  end

  # Test suite for GET /todos/:todo_id/items/:id
  describe 'GET /todos/:todo_id/items/:id' do
    before { get "/todos/#{todo_id}/items/#{id}" }

    context 'when todo item exists' do
      it 'returns status code 200' do
        expect(response).to have_http_status(200)
      end

      it 'returns the item' do
        expect(json['id']).to eq(id)
      end
    end

    context 'when todo item does not exist' do
      let(:id) { 0 }

      it 'returns status code 404' do
        expect(response).to have_http_status(404)
      end

      it 'returns a not found message' do
        expect(response.body).to match(/Couldn't find Item/)
      end
    end
  end

  # Test suite for PUT /todos/:todo_id/items
  describe 'POST /todos/:todo_id/items' do
    let(:valid_attributes) { { name: 'Visit Narnia', done: false } }

    context 'when request attributes are valid' do
      before { post "/todos/#{todo_id}/items", params: valid_attributes }

      it 'returns status code 201' do
        expect(response).to have_http_status(201)
      end
    end

    context 'when an invalid request' do
      before { post "/todos/#{todo_id}/items", params: {} }

      it 'returns status code 422' do
        expect(response).to have_http_status(422)
      end

      it 'returns a failure message' do
        expect(response.body).to match(/Validation failed: Name can't be blank/)
      end
    end
  end

  # Test suite for PUT /todos/:todo_id/items/:id
  describe 'PUT /todos/:todo_id/items/:id' do
    let(:valid_attributes) { { name: 'Mozart' } }

    before { put "/todos/#{todo_id}/items/#{id}", params: valid_attributes }

    context 'when item exists' do
      it 'returns status code 204' do
        expect(response).to have_http_status(204)
      end

      it 'updates the item' do
        updated_item = Item.find(id)
        expect(updated_item.name).to match(/Mozart/)
      end
    end

    context 'when the item does not exist' do
      let(:id) { 0 }

      it 'returns status code 404' do
        expect(response).to have_http_status(404)
      end

      it 'returns a not found message' do
        expect(response.body).to match(/Couldn't find Item/)
      end
    end
  end

  # Test suite for DELETE /todos/:id
  describe 'DELETE /todos/:id' do
    before { delete "/todos/#{todo_id}/items/#{id}" }

    it 'returns status code 204' do
      expect(response).to have_http_status(204)
    end
  end
end
``
## en el controlador app/controllers/items_controller.rb

```
before_action :set_todo
  before_action :set_todo_item, only: [:show, :update, :destroy]

  # GET /todos/:todo_id/items
  def index
    json_response(@todo.items)
  end

  # GET /todos/:todo_id/items/:id
  def show
    json_response(@item)
  end

  # POST /todos/:todo_id/items
  def create
    @todo.items.create!(item_params)
    json_response(@todo, :created)
  end

  # PUT /todos/:todo_id/items/:id
  def update
    @item.update(item_params)
    head :no_content
  end

  # DELETE /todos/:todo_id/items/:id
  def destroy
    @item.destroy
    head :no_content
  end

  private

  def item_params
    params.permit(:name, :done)
  end

  def set_todo
    @todo = Todo.find(params[:todo_id])
  end

  def set_todo_item
    @item = @todo.items.find_by!(id: params[:id]) if @todo
  end

  ```

  


