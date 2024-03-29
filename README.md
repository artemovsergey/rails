# Мануал по Ruby on Rails

## Развертывание приложения на VPS

### Создание публичного ssh-ключа

```ssh-keygen```
```ls ~/.ssh/```
```cat ~/.ssh/id_rsa.pub```

### Действия на сервере VPS

- добавить на хостинг ssh ключ
- сгенерировать сервер
- поменять его имя
- сразу зайти не получиться потому, что 80 порт, который ответственен за http не открыт, но открыт порт 22 для ssh
- ls ~/.ssh/autorized_keys на удаленном сервере = cat ~/.ssh/id_rsa.pub на хосте

### Создание пользователя в Linux

```dpkg-reconfigure locales```
```adduser  iof``` 
```adduser  iof sudo```

### Копирование ключа с удаленного сервера
ssh-copy-id  iof@server_remote_ip

### Установка rbenv

- установка rbenv ( на серверах )

- установка плагина ruby-build

- rbenv ruby 3.0.0

- rbenv global 3.0.0

- which ruby

- настройка отключения скачивания документации 

- установить bundler

### Установка nginx passenger

```sudo service nginx start```

- проверяем сервер и должны увидеть стартовое окно nginx

- надо настроить nginx и отредактировать файл

- nano etc/nginx/nginx.conf

- ищем команды include и раскомментируем

- nano etc/nginx/passenger.conf

```
passenger_root /usr/lib/ruby/vendor_ruby/fusion_passenger/location.ini;
passenger_root /home/[user]/.rbenv/shims/ruby;
```

перезапустить сервер

```sudo service nginx restart```

создадим папку

/etc/nginx/sites_available/myapp

myapp - название сайта приложения 

в файл скопировать

```
server 
{
		listen 80 default_server;
		listen [::]:80 default_server ipv6only=on;
		server_name mydomen.com;
		access_log  /var/log/nginx/deploy.access.log;
		error_log  /var/log/nginx/deploy.error.log;
		passeger_enabled on;
		rails_env production;
		root home/[user]/www/public;
}
```

mkdir -p /www/public/
echo "Hello World" > /www/public/index.html

включим наш сервер и сделаем символьную ссылку

sudo ln -s /etc/nginx/sites_available/myapp  /etc/nginx/sites_enabled/myapp

ls  /etc/nginx/sites_enabled/

удаляем стандартный сервер default 

sudo -rm /etc/nginx/sites_enabled/default 

перезапустим сервис

sudo service nginx restart

перезайдем на сервер и должны увидеть hello world

установка posgresql на ubuntu

nano /etc/postgresql9.5/main/pg_hba.conf

ищем строку

local all all peer

эта настройка отвечает за то, что можно аутентифицироваться в базе с таким же именем как имя в linux


sudo su -postgres

psql


CREATE USER iof WITH PASSWORD '123';

CREATE DATABASE myapp WITH OWNER = iof;

\q

psql myapp

psql --user iof --p myapp


создаем приложение на rails

rails new bookmarks

cd bookmarks

bundle exec rails g  scaffold bookmark title url

bundle exec rake db:migrate

nano config/routes.rb

root to: 'bookmarks#index'

bundle exec rails s

запушим в github

database.yml   secrets.yml не должны быть в репозитории

mv config/database.yml  config/database.yml.example

mv config/secrets.yml  config/secrets.yml.example

nano .gitignore

добавляем 

config/database.yml

config/secrets.yml 

nano Gemfile

group :production
  gem 'pg'
end

добавлем в репозиторий и коммит

push

если у нас приватный репозиторий

генерируем ключ для сервера

ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub

настройки репозитория deploy keys

сервер имеет доступ к этому репозиторию ,чтобы забирать исходный код

удаляем прошлую папку www

git clone  ...   www

заходим в папку

доустанавливаем пакеты

sudo apt-get install nodejs sqlite3 libsqlite3-dev libpq-dev

bunle install --without test development

настройка config/database.yaml

```
produccion:

adapter: postgresql

user:iof

password:123

database:myapp
```

bundle exec rake db:migrate RAILS_ENV  = production

rake secret

копируем

mv config/secrets.yml.example config/secrets.yml  

nano config/secrets.yml  

```
production:
secret_key_base: paste
```

bundle exec rake assets:precompile RAILS_ENV:production

sudo service nginx start

если что-то изменили

git push 

в папке приложения www git pull

touch tmp/restart.txt

если этот файл был изменен сервер перезапускается




## Создание приложения

```rails
rails new app --skip-test --database=postgresql
rails делает при создании bundle install и rails webpacker:install
```
**Замечание**: при работе с WSL надо проверить наличие библиотеки ```sudo apt install libpq-dev```.


### Удаляем test и устанавливаем rspec-rails
	
Далее ```rails g rspec:install```
	Если у нас ruby-3.0.0 то надо при использовании rpec установить ``gem 'rexml'``
	Запуск теста rspec путь к тесту

**Замечание**: gem ```rspec-rails```

### Настройка .rspec

```rails
--require spec_helper
--color
--format doc
```

### Настройка при деплое на Heroku для webpack

Webpacker подключает новую задачу webpacker: compile в assets: precompile, которая запускается всякий раз, когда вы запускаете assets: precompile. Если вы не используете Sprockets webpacker: compile автоматически становится псевдонимом assets: precompile. Не забудьте установить для переменной среды NODE_ENV значение production во время развертывания или при выполнении задачи rake.

Чтобы ваше приложение Webpacker могло работать на Heroku, вам необходимо предварительно выполнить небольшую настройку.

```rails
heroku login
heroku apps:create name_app
heroku git:remote -a name_aa
heroku addons:create heroku-postgresql:hobby-dev  # создается база и имя базы 
heroku buildpacks:clear
heroku buildpacks:add heroku/nodejs --index 1
heroku buildpacks:add heroku/ruby --index 2
git push heroku main
```

### Для работы с базой данных Posgres на Heroku

heroku logs
heroku run rak    e db:migrate
heroku run rake db:schema:load

### Скаффолдинг

Мы сразу создаем модель, контроллер и представление. А также ресурсы и тесты

```raisl 
rails generate scaffold User

rails generate scaffold Cart

rails generate scaffold LineItem   [product:references cart:belongs_to}
- это поля миграции
```

### Подготовить системную переменную ENV

```yml

default: &default 
  adapter: postgresql 
  pool: 25 
  timeout: 5000 
  host: <%= ENV['DATABASE_HOST'] || 'localhost' %> 
  username: <%= ENV['DATABASE_USERNAME'] || 'postgres' %> 
  password: <%= ENV['POSTGRES_PASSWORD'] ||'root' %> 
  port: <%= ENV['DATABASE_PORT'] || 5432 %> 
  encoding: unicode 
development: 
  <<: *default 
  database: <%= ENV['DATABASE_NAME'] || 'app_development' %> 
test: 
  <<: *default 
  database: 'app_test' 
production:   
  <<: *default 
  database: <%= ENV['DATABASE_NAME'] || 'app_production' %>
```

### Gemfile

```ruby
source 'https://rubygems.org'  
git_source(:github) { |repo| "https://github.com/#{repo}.git" }  
ruby '3.0.0'  
gem 'rails', '~> 6.1.3', '>= 6.1.3.2'  
gem 'pg', '~> 1.1'  
  gem 'database_cleaner', '~> 1.7'  
gem 'puma', '~> 5.0'  
gem 'sass-rails', '>= 6'  
gem 'webpacker', '~> 5.0'  
gem 'turbolinks', '~> 5'  
gem 'jbuilder', '~> 2.7'  
gem 'bootsnap', '>= 1.4.4', require: false  
gem 'ancestry'  # for category  
gem "aws-sdk-s3", require: false  
gem 'breadcrumbs_on_rails'  
#gem 'doorkeeper'  
gem 'meta-tags'  
gem 'oj', '~> 3.10'  
gem 'oj_mimic_json', '~> 1.0', '>= 1.0.1'  
gem 'omniauth'  
gem 'omniauth-facebook'  
gem 'pundit'  
gem 'pagy', '< 3.5'  
gem 'devise'  
gem 'rails-observers'  
gem 'russian', '~> 0.6.0' # работает для ru   
group :frontend do  
  gem 'bootstrap-sass'  
  gem 'simple_form' # rails g simple_form:install --bootstrap  
  gem 'jquery-rails'  
  gem 'slim-rails'  
  gem 'coffee-rails'  
  gem 'haml-rails', '~> 2.0'  
  gem 'react-rails'  
  gem 'frontend_notifier' # for flash messages  
end  
group :visualization do  
  gem 'rails-erd'  
  gem 'railroady'  
end  
group :development do  
  gem 'rack-mini-profiler', '~> 2.0'  
  gem 'web-console', '>= 4.1.0'  
  gem 'listen', '~> 3.3'  
  gem 'spring'  
  gem 'spring-watcher-listen', '~> 2.0.0'  
  gem 'rspec-rails'  
    gem 'rexml'  
  gem 'shoulda-matchers'  
    gem 'rails-controller-testing'  
  gem 'rubocop'  
  gem 'rubocop-performance'  
  gem 'rubocop-rails'  
  gem 'faker'  
  gem 'factory_bot_rails'  
  gem 'json_spec', '~> 1.1', '>= 1.1.5'  
  gem 'launchy', '~> 2.4', '>= 2.4.3'  
  gem 'rspec-json_expectations', '~> 2.2'  
  gem 'byebug', platforms: [:mri, :mingw, :x64_mingw]  
end  
group :test do  
  gem 'pry'  
  gem 'pry-byebug'  
  gem 'db-query-matchers', '~> 0.10.0'  
  gem 'capybara', '>= 3.26'  
  gem 'selenium-webdriver'  
end  
gem 'tzinfo-data', platforms: [:mingw, :mswin, :x64_mingw, :jruby]  
group :security do  
  gem 'bcrypt', '~> 3.1.7'  
end

```

### Подключение Bootstrap по прямой ссылке

```html
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">
```

```html
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/js/bootstrap.bundle.min.js" integrity="sha384-gtEjrD/SeCtmISkJkNUaaKMoLD0//ElJ19smozuHV6z3Iehds+3Ulb9Bn9Plx0x4" crossorigin="anonymous"></script>
```

### Bootstrap

Пишем в консоли для Bootstrap 4
```yarn add bootstrap jquery popper.js```

Пишем в консоли для Bootstrap 5

```rails
yarn remove bootstrap jquery popper.js
yarn add bootstrap@next jquery @popperjs/core
```

В файле webpack/enviroments.js

```js
const { environment } = require('@rails/webpacker')
const webpack = require('webpack')
environment.plugins.prepend('Provide',
  new webpack.ProvidePlugin({

    $: 'jquery/src/jquery',
    jQuery: 'jquery/src/jquery',
    Popper: ['popper.js', 'default']
    
  })
  )
module.exports = environment
```


Создаем в папке javascript папку stylesheets. В ней создаем файл application.scss. В файле импортируем модуль scss bootstrap:
@import "~bootstrap/scss/bootstrap";

app/javascript/packs/application.js
import 'bootstrap' 
import '../stylesheets/application'


Для отображения валидаций в моделях хорошо подходит gem simple_form
Также он хорошо работает с bootstrap

### Для задачи прикрепления или выбора файла, например изображения в форме можно указать

```html
<div class="field form-group">
    <label for="exampleFormControlFile1">Image</label>
    <input type="file" name = "product[image_url]" id = "product_image_url" class="form-control-file" required autofocus>
  </div>
```

### Команды Rails для работы с текстом

```rails
cicle("class1","class2")  - чередование значений
truncate  - ограничение текста n символами
strip_tags -  текст без тегов
sanitize(text) - понимает html теги в тексте
number_to_currency(price)  - добавляет к цене название валюты
```

### Для AJAX запроса

POST запрос отправлялся с опцией remote: true
Ответ приходил в виде json

```rails 
format.json { render :show, status: :created, location: @line_item }
```

Файл create.js.erb. Например
```rails
$('#cart').html("<%= escape_javascript render('include/cart',local: @cart.line_items) %>");
```

### Электронная почта

Отправка сообщений электронной почты состоит в Rails из трех основных частей: конфигурации отправки электронных сообщений, определения момента отправки сообщения и указания того, что нужно сообщить.

```config/enviroments/development.rb```

Конфигурирование для Gmail для development

```rails
config.action_mailer.delivery_method = :smtp  
  # Альтернативой :smtp служат аргументы :sendmail и :test 
   
  config.action_mailer.smtp_settings = { 
    address: "smtp.gmail.com", 
    port: 587, 
    domain: "domain.of.sender.net", 
    authentication: "plain", 
    user_name: "artik3314@gmail.com", 
    password: "00003314", 
    enable_starttls_auto: true 
    }
```

Конфигурирование для Gmail из Heroku приложения для production. При этом
надо получить код приложения в настройках безопасности Google (нужна двухфакторная аутентификация) включить доступ к небезопасным приложениям. В настройках Heroku включить master-key из приложения или через переменные приложения

``` export MASTER_KEY=value ```

``` heroku config:add MASTER_KEY=value ```

```ruby

config.require_master_key = true

  config.action_mailer.delivery_method = :smtp
  host = 'artik3314.herokuapp.com' #replace with your own url
  config.action_mailer.default_url_options = { host: host }
  
  config.action_mailer.smtp_settings = {
    address: "smtp.gmail.com",
    port: 587,
    #domain: "domain.of.sender.net",
    authentication: "plain",
    user_name: "artik3314@gmail.com",   #  or ENV['GMAIL_SMTP_USER']
    password: 'faiysghilemvbpvj', # code apps from Google Gmail API
    enable_starttls_auto: true
    }

```

```rails generate mailer OrderNotifier received```


```ruby

class OrderNotifierMailer < ApplicationMailer
  def received(order)
    
    @order = order
    
    mail to: order.email, subject: 'Подтверждение заказа'
  end
  
end

```
В настройках Google при отправлении почты на Gmail разрешить подключения для небезопасных подключений. Через некоторое время Google отключает эту настройку

Событие отправки электронной почты

```OrderNotifierMailer.received(@order).deliver```


### Поиск

Добавляем в коллекцию маршрутов

```ruby
  resources :users do
    get 'search', on: :collection
  end
```

В форме ставим настройки на action

```html
    <form class="d-flex"  action = <%= search_users_path %> %>
      <input name="search" class="form-control me-2" type="search" placeholder="Search" aria-label="Search">
      <button class="btn btn-outline-success" type="submit">Search</button>
    </form>
```

Метод в контроллере на регулярное выражение

```ruby
  def search
    @users = User.where("name LIKE '%#{params[:search]}%'  ")
  end
```


## Теория

Rails относится к среде Модель—Представление—Контроллер (Model—View—Controller). Rails получает входящие запросы от браузера, декодирует запрос для поиска контроллера и вызывает в контроллере метод, который называется действием. Затем контроллер вызывает соответствующее представление, отображающее результаты на экране пользователя.

Связывая все три компонента вместе, жизненный цикл нашего Rails-приложения выглядит следующим образом: 1. Пользователь нажимает на кнопку возле формы или переходит по ссылке или вводит url вручную — браузер отправляет соответствующий запрос 2. В зависимости от запроса, он попадает в тот или иной контроллер, который решает что с ним делать 3. Скорее всего, метод контроллера, который обрабатывает запрос, найдет ту или иную модель и, возможно, обновит ее свойства 4. Затем контроллер сделает редирект на другой url (в этом случае цикл повторится сначала) или примет решение отрендерить тот или иной шаблон из views 


В Rails-приложении входящий запрос сначала посылается маршрутизатору, который решает, в какое место приложения должен быть отправлен запрос и как должен быть произведен синтаксический разбор этого запроса. В конечном итоге на данном этапе где-то в коде контроллера идентифицируется конкретный метод (называемый в Rails действием). Действие может искать запрошенные данные, может взаимодействовать с моделью и может вызвать другое действие. В результате выполнения действие подготавливает информацию для представления, которое создает изображение для пользователя.


Объект является комбинацией состояния (например, количество и идентификатор товара) и методов, использующих это состояние

Во многих языках понятие nil (или null) означает «объект отсутствует». Но в Ruby все по-другому: nil является таким же объектом, как и все остальные, и предназначен для представления отсутствия.

Инструкции if встречаются в Ruby-приложениях довольно часто, но тех, кто начинает осваивать язык Ruby, зачастую удивляет редкое использование конструкций циклов. Вместо них часто используются блоки и итераторы


## Модели

Создать модель rails g model  [name_model]


## Импорт данных

Rails предоставляет нам возможность импортировать исходные данные.

```ruby
rake db:seed 
```

### Создание миграций

Начнем с создание миграций (подразумевается как создание, изменение,  редактирование базы данных. Отображение объектной модели на реляционную модель

 ```ruby
 rails g migration  #name
 ```
 Путь  db/migrate  =>

 ```ruby
 def change
         create_table :items do |t|
         t.float :price
         t.string :name
         t.boolean :real
         t.float :weigt
         t.timestamps
        end
 end
 ```

### Применим миграцию к базе данных

```ruby
rake db:migrate
```

Чтобы протестировать можно воспользоваться ```rails console``` и ввести ```rails c```

 Чтобы задавать атрибуты моделей вовсе необязательно использовать сеттеры. Можно передать хэш с атрибутами прямо в метод .new, например вот так:
Можно использовать

```ruby
Item.create(name: "Porsche", description: "A really fast car", price: 3000000) 
```

 Создать объект модели, например, если модель называется ```item```, то  ```i = Item.new.``` Для сохранения ```i.save```

 Можно настроить валидацию на момент создания объекта  или на момент сохранения в базу данных.

### Params

Есть специальный hash, который называется ```params[ ]```. Он содержит в себе данные запроса пользователя.

### RESTfull

Чтобы сделать из контроллера RESTfull контроллера надо добавить специальные методы: ```create```,```show```,```update```,```destroy```. А также ```index``` и ```edit```. Чтобы это все заработало  -  надо в роутере прописать строку  ```resources  : #name_model```


###  Тест контроллера без View

Протестировать работу контроллеров без использования template можно используя команду render plain :  "text"


### Для подключения файлов js через assets piperline 

Можно использовать строчку, чтобы работать через ```webpacker```  и установить webpack

```<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload' %>```

```rails webpacker:install```

Отправленные данные из формы хранятся в params. Это POST запрос, а значит он будет обрабатываться методом ```create```

Для форм использовать можно ```helpers for_form```

Чтобы не сбрасывались значения в форме, надо изменить метод ```new```, т.е. прописать ```@item = Item.new```.

Если существует erb , то ```render  #controller/#action```   можно не писать явно

Далее чтобы создать связи ( тут они называются ассоциации ) можно воспользоваться методами ```has_one``` и  ```belongs_to```

Паршиалы - это для того, чтобы можно было весь дублированный код вынести
Layout  то же самое, только это для  html.erb  c помощью ключевого слова yield

При  поиске в моделях удобно использовать метод ```where``` для запросов в Active Record. Можно применять цепочки ```where```

Для того, чтобы применить  ```flash[:errors]``` надо также их выводить в template

Можно использовать шаблоны   ```template.html.haml```

Для подключения bootstrap надо в файл application.css добавить ```@import  "bootstrap"```;  предварительно переименовав в ```application.css.scss```

Есть специальный gem simple_form для генерации форм

Установка гемов в Windows без документации  
```echo install: --no-document && echo update: --no-document) >> c:\ProgramData\gemrc ```

### Английский

Решайте сами, нужен ли вам преподаватель, чтобы выучить английский. Главную ошибку, которую допускают люди, нанимая преподавателя для обучения чему бы то ни было — подсознательное предположение, что преподаватель загрузит все знания в ваш мозг просто потому, что вы присутствуете на занятии. Этого не произойдет и, конечно же, основную работу вам придется как всегда проделывать самому. Максимум, что на мой взгляд может дать преподаватель — это фидбек и исправление ошибок.

---

Ruby on Rails пропагандирует использование REST: к одному URL в приложении должно быть привязано только одно действие.

---

При развертывании приложения на Heroku надо помнить о том, что в приложении по умолчанию используется на стадии разработки ```sqlite3```, а на сервере используется ```postgres```. Поэтому важно правильно настроить Gemfile.

---

Если при обращении к контроллеру (class) он пустой и его методы пусты, то это шаг пропускается и отображается только представление view (одноименное)

---

### Изображения app/assets/images

В конвейере ресурсов предпочтительным местом для этих ресурсов теперь является app/assets каталог. Файлы в этом каталоге обслуживаются промежуточным программным обеспечением Sprockets.

---

Чтобы подключать разные таблицы стилей для разных контроллеров надо в файл ```assets/stylesheets/application.css.scss``` добавить 

```//= require_self```
```//= require_tree```

Далее  в views/layouts/application.html.erb  дописать   ```<body class="<%= controller.controller_name %>">```

---

Фактически всем веб-приложениям в настоящее время требуется какая-либо система входа и аутентификации. Неудивительно, что большинство веб-фреймворков реализует множество вариантов подобных систем, и Rails не исключение. В качестве примеров систем аутентификации и авторизации можно привести: Clearance1, Authlogic2, Devise3 и CanCan4

---

Обратите внимание, что, в отличие от соглашения о выборе множественного числа для имен контроллеров, для именования моделей используется единственное число: контроллер Users, но модель User

---
Rails поставляется с тремя настроенными окружениями: test (тестовое), development (разработки) и production (промышленное). Окружением по умолчанию для консоли Rails является окружение разработки: ```$ rails console Loading development environment >> Rails.env => "development" >> Rails.env.development? => true >> Rails.env.test? => false```

---

При оформлении тестовых значений имеет значение оформление в  file.yml

---

Чтобы добавить столбец, мне просто нужно было выполнить следующие действия:

rails generate migration add_fieldname_to_tablename fieldname:string
Alternative
rails generate migration addFieldnameToTablename
Как только миграция будет сгенерирована, отредактируйте перенос и определите все атрибуты, которые вы хотите добавить в этот столбец.
Примечание. Названия таблиц в Rails всегда множественны (чтобы соответствовать соглашениям DB). Пример использования одного из шагов, упомянутых ранее
rails generate migration addEmailToUsers
rake db:migrate
Вы можете изменить схему в db/schema.rb, добавить столбцы в SQL-запрос.
Запустите эту команду: rake db:schema:load
Имейте в виду, что при запуске rake db:schema:load автоматически стираются все данные в ваших таблицах.

---

Вы можете отменить последнюю миграцию на ```rake db:rollback STEP=1```
или отмените эту конкретную миграцию rake ```db:migrate:down VERSION=<YYYYMMDDHHMMSS>```
и отредактируйте файл, затем запустите ```rake db:mirgate``` снова.

---

Ассоциация ```has_one/belongs_to``` добавила в наши модели по одному важному методу, названия которых соответствуют названиям моделей. Во всех объектах класса User появился метод #cart, а во всех объектах класса Cart появился метод #user.

---

Метод find ищет по id
Например,  ```Item.find(1)``` или может принимать массив из значений:   ```Item.find([1,2,3])```

---

Поиск в модели

```ruby
Item.where(price : 400)
Item.where("price > 400")
Item.where("price > ?",400)
```
Примечание: запрос будет выполнен только тогда, когда на нем будет применен метод массива
---


Чтобы сделать сортировку по определенному полю можно на результат запроса типа Active Record :: Relation применить метод .order( " поле ASC/DESC " )

---

```Rails > 5```
При создание экшена ```create```

```ruby
 private

 def item_params
     params.require(:item).permit(:name,:price,:discription,:weigt)
 end
 ```

 ---

 С точки зрения Rails, все публичные методы внутри контроллера — это экшены, которые должны быть доступны посетителю сайта через то или иное сочетание url + http-глагола. Приватные же методы экшенами не являются, но могут быть вызваны изнутри любого экшена в контроллере 

 Объекты модели можно вызывать сразу в шаблоне и не использовать controller. Это показывает то, что view относительно независимо от контроллера

Для flash сообщений можно использовать гем frontend_notifier


По умолчанию, в соответствии с дефолтными настройками Rails, сжатие файлов происходит только в продакшен среде. На вашей локальной машине файлы сжаты не будут и, если вы решите заглянуть в исходный код страницы и щелкните по ссылке на один из css или javascript-файлов, вы увидите несжатую версию. 

Для подключения бутстрапа
В Gem файле  gem 'bootstrap-saas'

Так как тесты используют базу данных из среды test, то нам важно не забыть применить к ней все те же самые миграции, которые вы прогнали для development базы данных. Иначе тестируемые вами модели столкнутся с тем, что для них нет соответствующих таблиц и полей в тестовой базе данных. Для того чтобы применить миграции к вашей тестовой базе данных, есть специальный rake-таск: rake db:test:clone который просто применит текущую схему в файле db/schema.rb к вашей тестовой базе данных. Я взял за привычку выполнять эту команду одновременно с rake db:migrate каждый раз, когда я добавляю в проект новые миграции. Делается это так: rake db:migrate && rake db:test:clone 


Если мы дописали новый гем в Gemfile, то надо прогнать команду bundle install, если обновили версию гема, который уже есть в Gemfile, то надо понимать, что этот гем может потребовать изменения все гемов. В этом случае надо сделать команду bundle update.

Если в шаблоне мы комментируем код, то все равно выполняется ruby код

В Linux создает файл конфигурации, который не качает документацию

```echo 'gem: --no-document' >> ~/.gemrc```

---

```params is missing or the value is empty```

Ошибка происходит, потому что наш хэш params не включает корневой ключ: сообщение. Мы можем диагностировать эту проблему, используя params.inspect в интерактивной консоли или распечатывая вывод определенных команд.
Решение:

http://localhost:3000/items/index?items[name]=test1&items[price]=107

Вывод  p params покажет следующее:

<ActionController::Parameters {"items"=><ActionController::Parameters {"name"=>"test1", "price"=>"107"} permitted: false>, "controller"=>"items", "action"=>"index"} permitted: false>
---

### Связи между моделями/ таблицами

has_one    belongs_to   1:1
has_many  belongs_to  1:М
has_one_and_belongs_to  1:М   ( после создания моделей и прописывания связей между моделями, надо создать соединяющую таблицу, создаем миграцию)

```ruby
rails g migration create_carts_items
```

Замечание. Название таблицы должно быть во множественном числе в алфавитном порядке

Потом заполнить миграцию следующим образом в методе change ( опция id: false говорит нам о том, что в таблице не создастся поля id )

```ruby
create_table :carts_items, id: false do |t| 
      t.integer :cart_id 
      t.integer :item_id 
    end
```

```has_many :through```


Получается, при такой связи двух моделей/ таблиц  мы создаем промежуточную модель в приложении, а также новую таблицу в базе с ссылками на таблицы, которые надо связать ( это полноценна таблица с id и другими возможными полями, а не просто связующая таблица )


```:through ```

говорит о том, что мы указываем модель "через" которую будет осуществляться связь M : M
это связь нужна только тогда, когда нужны дополнительные данные в связывающей таблице

```ruby

class Item < ApplicationRecord 
    has_many :position 
    has_many :carts, through: :position 
end

class Cart < ApplicationRecord 
    has_many :position 
    has_many :items, through: :position 
end

class Position < ApplicationRecord 
    belongs_to :item 
    belongs_to :cart   
end
```

Для перехода на определенную версию ruby, если ее нет в rvm

```rvm list```
```rvm install ruby <x.x.x>```
```rvm use ruby <x.x.x>```

---

Чтобы пользоваться RubyMine надо установить настройки wsl

which rvm 

/usr/local/rvm/bin/rvm 

и прописать в настройках RubyMine

Также прописать путь к  терминалу wsl

---

Использование rails partial ( паршиалы ) для вынесения дублированного кода в отдельные одноименные вью файлы с нижним подчеркиванием. Например, если в form.html.erb есть повторяющийся код, то его можно вынести в _form.html.etb, а в form.html.erb  в ставить 

```ruby
render partial: form
```

---

```rails g simple_form:install --bootstrap```

По умолчанию simple_form называет поля по именам атрибутов в модели ( в базе данных ). Поэтому, чтобы перевести названия полей надо использовать следующий код
Отдельно можно настраивать locale для simple_form.ru.yaml

---

Чтобы перевести приложение на другие языки можно воспользоваться gem i18n

Cтруктура концерн (concern), делает доступными методы для всех наших контроллеров.

Объект params играет в приложениях Rails довольно важную роль. Он содержит все аргументы, переданные в запросе браузера.

```rails generate migration add_quantity_to_line_items quantity:integer```

Исходя из имени миграции, Rails может сообщить, что вы добавляете один или несколько столбцов к таблице line_items, и может взять имена и типы данных для каждого столбца из последнего аргумента. Rails ищет соответствие двум шаблонам: add_XXX_to_TABLE и remove_XXX_from_TABLE, где значение XXX игнорируется. Значение имеет список имен столбцов и типов данных, появляющийся после имени миграции.

Rails предоставляет удобную rake-задачу, позволяющую проверить состояние ваших миграций.


```rake db:migrate:status```

---

А почему мы не можем хранить информацию об ошибках в любой существующей переменной экземпляра? Не забывайте, что после того, как сообщение о неправильном идентификаторе отослано приложением браузеру, тот посылает в ответ приложению новый запрос. За то время, пока он будет идти, приложение не будет стоять на месте, и все переменные экземпляра, существовавшие во время предыдущих запросов, канут в Лету. А флэш-данные хранятся в сессии для того, чтобы быть доступными между запросами.

---

Очистить журналы лого

```rake log:clear LOGS=test```

---

Поначалу кажется, что будет довольно сложно узнать, когда при необходимости создать ссылку или перенаправление по заданному маршруту нужно использовать метод product_path, а когда метод product_url. Но на самом деле все довольно просто. При использовании метода product_url вы получите полную начинку с протоколом и доменным именем, наподобие http://example.com/products/1. Его следует использовать, если осуществляется перенаправление redirect_to, поскольку спецификация HTTP при осуществлении перенаправлений с кодом 302 и им подобных требует указывать URL-адрес полностью. Полный URL-адрес нужен также при перенаправлении с одного домена на другой, например product_url(domain: "example2.com", product: product).
Во всех остальных случаях можно с успехом использовать product_path. Этот метод будет генерировать только часть пути /products/1, а для ссылок или указания форм вроде link_to "My lovely product", product_path(product) больше ничего и не нужно. Путаница возникает из-за того, что снисходительность браузеров зачастую делает эти два метода взаимозаменяемыми. Перенаправление redirect_to можно производить с product_path, и такой вариант, скорее всего, сработает, но с точки зрения спецификации он будет считаться неправильным. Точно так же можно при ссылке link_to использовать product_url, но тогда ваш HTML будет засорен ненужными символами, что также нельзя признать удачным вариантом.

По умолчанию form_with создает форму, отправляемую с помощью Ajax для избежания полной перезагрузки страницы. Для упрощения этого руководства, мы отключили эту особенность, используя local: true в вышеприведенном коде.

Создание внешнего ключа

```ruby
class CreateComments < ActiveRecord::Migration[6.0]
  def change
    create_table :comments do |t|
      t.string :commenter
      t.text :body
      t.references :article, null: false, foreign_key: true

      t.timestamps
    end

  end
end

```

```ruby
 resources :articles do
    resources :comments
```

Active Record
По умолчанию название класса модели в единственном числе, а название таблицы базы данных в множественном числе

Можно думать о приложении как о real-time стратегии, где модели представляют из себя различные юниты, появляющиеся и исчезающие на карте сражений.

Чтобы заработали связи в моделях нужно сделать
1 Определить отношения has_many / belongs_to в моделях
2 В миграции или в схеме базы данных определить внешний ключ через атрибут t.integer user_id или t.references  :user

order1 = user.orders.create

То есть  user.orders -  это заказы пользователя. Создать заказ пользователя. 
Метод user.orders.create работает также как и Order.create и сохраняется к user
У массива user.orders есть метод create 

has_many :orders, dependent: :destroy     удаление связанных объектов вместе с моделью

action все public методы

Строго говоря, params — это метод, доступный внутри любого метода в наших контроллерах и возвращающий хэш с отправленными в контроллер данными. Для удобства, мы далее будем говорить "хэш params" или просто "params".

Стоит обратить особое внимание на две вещи. Первая: переменная должна быть именно инстансной (начинаться с символа @), чтобы быть доступной в шаблоне.

Сделать доступным метод контроллера в шаблонах, то есть сделать его хелпером, можно

helper_method :name_method


Иногда вам может понадобиться, чтобы все экшены в контроллере рендерили шаблоны с другим лэйаутом. В этом случае не очень удобно указывать метод #render явно внутри каждого экшена. Предположим, что у нас есть контроллер AdminsController, доступный только администраторам. В этом случае, чтобы все экшены рендерили шаблоны в лэйауте "admin", достаточно вызвать метод класса .layout: class AdminsController < ApplicationController layout "admin" ... end


Таким образом, правило использования flash будет звучать так: если вы делаете редирект — используйте обычный flash, если вы рендерите какой-то шаблон и вам нужно вывести сообщение об ошибке — используйте flash.now.


Assets ( обычно произносится как "ассеты") в Rails приложении — это файлы, которые тем или иным образом были вставлены в веб-страницу, которую рендерит Rails приложение и которые должны быть загружены браузером, чтобы страница выглядела именно так, как задумал автор. Если говорить конкретно, то ассеты это изображения, css файлы и javascript-файлы. Все эти файлы в Rails-приложении должны находиться в папке app/assets/ в соответствующих поддриректориях images/ (для изображений), stylesheets/ (для cssфайлов) и javascripts/ (для javascript-файлов). Asset Pipeline — это специальный механизм Rails, который умеет автоматически делать следующие вещи с ассетами: 1) сжимать их (уменьшать размер), чтобы браузеру приходилось качать меньший объем данных 2) соединять файлы вместе, чтобы браузер делал меньше запросов к серверу 3) применять к файлам т.н. препроцессинг (например преобразовывать scss в css и coffeescript в javascript) 4) создавать уникальные "отпечатки пальцев" каждого ассета, чтобы браузер мог перезагрузить ассет в случае, если он изменился (вместо того, чтобы использовать закэшированную версию). Рассмотрим каждый из этих пунктов подробнее.

Уменьшение размера ассетов Этот пункт относится только к css и javascript-файлам. В продакшен режиме на сервере ваша задача — загрузить страницу пользователю максимально быстро. И не только потому, что пользователь может уйти с вашего сайта не дождавшись загрузки страницы, а еще и потому, что поисковые системы при ранжировании страниц учитывают скорость их загрузки. Одним из способов сократить время загрузки страницы — уменьшить размер файлов, которые отдаются сервером, в частности css и javascript файлов. Механизм Asset Pipeline умеет делать именно это: он берет css файл, удаляет в нем все пробелы, переносы строк и комментарии и, таким образом, сокращает его размер. Примерно то же самое Asset Pipeline делает и с javascript-файлами, правда в этом случае доступны более эффективные механизмы для сжатия (например, длинные имена переменных и функций внутри файла могут заменяться на короткие). По умолчанию, в соответствии с дефолтными настройками Rails, сжатие файлов происходит только в продакшен среде. На вашей локальной машине файлы сжаты не будут и, если вы решите заглянуть в исходный код страницы и щелкните по ссылке на один из css или javascript-файлов, вы увидите несжатую версию. Конкатенация ассетов. Еще один способ ускорить загрузку страницы — уменьшить количество запросов, которые браузер должен сделать на сервер, чтобы загрузить страницу. Когда браузер получает от Rails-сервера ответ в виде html-страницы, он не останавливается на этом. После получения ответа, он должен прочесть весь html и найти места, где упоминаются изображения, css-файлы или javascript-файлы (и, возможно другие, более редкие типы ассетов, такие как шрифты), и для каждого такого упоминания сделать еще один запрос на сервер, чтобы получить собственно файл с упомянутым именем. Таким образом, чтобы сократить количество запросов, которые браузер должен сделать на сервер, мы могли бы объединить несколько ассетов в один и вернуть их браузеру в виде одного файла. Предположим у нас есть два файла: application.css и items.css — оба содержат какие-то css-стили и оба нужны для правильного отображения страниц нашего сайта. Мы могли бы внутри лэйаута app/views/layouts/application.erb добавить оба эти файла в страницу: ... <%= stylesheet_link_tag "application", "items" %>  <%= stylesheet_link_tag "application", "items" %> ... и в этом случае браузер сделает два запроса к серверу — один для файла application.css и второй для файла items.css. Чтобы превратить это в один запрос к серверу, нам необходимо добавить в файл application.css следующую строку: /* *= require items */ ... и затем нам останется удалить упоминание о файле items.css из тэга в лэйауте: <%= stylesheet_link_tag "application" %> Таким образом, файл items.css окажется вложенным в application.css и браузер скачает с сервера только последний файл, выполнив всего-лишь один запрос, вместо двух. Похожим образом можно вложить один javascript-файл в другой. Вот как будет выглядеть код в файле application.js, если мы захотим вложить в него файл items.js: //= require items ... Как видите, отличие состоит только в символах комментария, которые предваряют директиву require: в css комментарии ограничиваются последовательностью /* */, а в javascript-е комментариями считаются строки, начинающиеся на //. Asset Pipеline определяет еще две интересных директивы — require_tree и require_self, почитать подробнее о которых можно в Rails Guides.

Препроцессинг. С развитием Rails стали популярны "надстройки" над css и javascript, которые позволяли в более элегантной форме писать код css и javascript-файлов, а затем, с помощью соответствующих библиотек, эти файлы компилировались в настоящие css и javascript файлы, понятные браузерам. Например язык scss позволил использовать при описании стилей страницы вложенные селекторы, которые, естественно, не поддерживаются в обычном css. А язык Coffeescript позволил избавиться от ненавистных многими фигурных скобок, которые являются обязательными в языке Javascript. Описание этих языков выходит за рамки книги, но полезно знать, что в Rails можно без каких-либо дополнительных усилий использовать именно эти языки, а не стандартные css и javascript. Для этого вам достаточно назвать ваш файл таким образом, чтобы у него было двойное расширение в имени файла, например application.css.scss или application.js.coffee. Увидев такой файл, Asset Pipeline, перед тем как позволить браузеру его скачать, скомпилирует его в обычный .css или .js соответственно. Именно это и называется препроцессингом.

rake db:migrate && rake db:test:clone


В Rails есть различные среды выполнения. Также можно создавать свои собственные среды

Есть возможность настраивать методы протоколирования с помощью log файлов или использовать системные файлы с помощью syslog. При запуске в терминале идет такое же протоколирование, как и в файле development.log

Rails используется для создания web-приложений поэтому сначала запрос обрабатывается веб-сервером. Затем сервер перенаправляет запрос приложению Rails, где он попадает к диспетчеру

Еще один случай - рендеринг подшаблона (partial template) . Вообще говоря, подшаблоны позволяют представить всю совокупность шаблонов в виде небольших файлов, избежав громоздкого кода и выделив модули, допускающие повторное использование.

Контроллер прибегает к рендерингу подшаблонов чаще всего для AJAX - вызовов, когда необходимо динамически обновлять участки уже выведенной страницы

Если браузеру нужно послать результат трансляции какого-нибудь фрагмента шаблона, который слишком мал, чтобы оформлять его в виде отдельной части ( То есть код для представления будет в контроллере, что не очень хорошо с точки зрения MVC )

Помощник при обработки AJAX запросов auto_complete_result

render :inline => "<%=  ruby code    %>"

Рендер текста осуществляется с помощью  ключа :plain
render :plain => "ok"

Также можно возвращать данные в  xml или json: 
render xml: @item.to_xml
render json: @item.to_json

Жизненный цикл приложения Rails разбит на запросы. При поступлении каждого нового запроса все начинается сначала
Рендеринг шаблона, который подразумевается по умолчанию является альтернативным, частичным, просто текстом или чем-нибудь еще - это последний шаг обработки запроса. Переадресация же означает, что обработка текущего запроса завершается и начинается обработка нового запроса

Если задается  redirect_to :name_controller , то подразумевается, что это action index

Фильтры применяются через методы. Это частый способ применения. Можно применять фильтры до, после и до-после. Также возможно делать условия only и except

Систем маршрутизации  конструирует запросы на действия которые можно отправить специальными помощникам link_to, button_to, tag_to, redirect_to, а также по URL понять какое действие следует выполнить

```
get 'example/index' , as: "/asv"
```

Ключевое слово as позволяет задать имя helper для адреса запроса

При создании ресурсов в виде resources  :items  создаются все маршруты, а при resource :item одиночные маршруты

Контроллеры - это скелет
Модель - ум и сердце
Представления - это кожа и одежда

rake db:migrate также выполняет  sql сценарий, если он лежит в папке ./db

