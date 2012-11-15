Коробка для подъёма [Rails](http://rubyonrails.org)-приложений на [Debian Squeeze](http://wiki.debian.org/DebianSqueeze) через [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo):

* Установка [hostname](http://community.opscode.com/cookbooks/hostname)
* Установка дефолтной [locale](http://community.opscode.com/cookbooks/locale)
* Создание [пользователей](https://github.com/fnichol/chef-user) приложений
* Установка [NGINX](http://community.opscode.com/cookbooks/nginx) из сорцов и дефолтная настройка
* Настройка [Rails](http://github.com/macovsky/chef-rails)-приложения: конфиг для NGINX и индивидуальный `/etc/init.d` скрипт для управления [Unicorn](http://unicorn.bogomips.org/)
* Установка [mysql](http://community.opscode.com/cookbooks/mysql) из пакета, установка пароля для `root` и создание [баз](http://github.com/macovsky/chef-rails)
* Установка [imagemagick](http://community.opscode.com/cookbooks/imagemagick) из пакета

**Установка, под рутом**

Первый делом поставим [rbenv](http://github.com/sstephenson/rbenv) и Ruby 1.9.3 с помощью [замечательного скрипта](https://gist.github.com/4076121):

```bash
sh -c "`curl -L http://bit.ly/rbenv-ruby-193-on-debian-squeeze`"
```

Потом уже [chef](http://www.opscode.com/chef/) + [ruby-shadow](https://github.com/apalmblad/ruby-shadow), [librarian](https://github.com/applicationsonline/librarian) для менеджмента cookbooks и [bundler](http://gembundler.com) на будущее:

```bash
gem i chef ruby-shadow librarian bundler
rbenv rehash
```

Скачиваем коробку и устанавливаем cookbooks из `Cheffile`:

```bash
git clone git://github.com/macovsky/rails-on-chef-solo.git /etc/chef
cd /etc/chef
librarian-chef install
```

Теперь надо сконфигурировать ноду, скопируем файл с примером:

`cp node.json.example node.json`

и откроем (на всякий случай напомню, что в `json` не бывает комментов):

```javascript
{
  // hostname
  "set_fqdn": "sloboda.dk",

  // каких пользователей надо создать
  // https://github.com/fnichol/chef-user
  // нужно создать одноимённые databags, например databags/users/sloboda.json
  "users": [
    "sloboda"
  ],

  // http://community.opscode.com/cookbooks/nginx
  "nginx": {
    "version"                       : "1.2.4",
    "worker_processes"              : 2,
    "worker_connections"            : 1024,
    "user"                          : "www-data",
    "server_names_hash_bucket_size" : 64,
    "keepalive_timeout"             : 30,
    "default_site_enabled"          : false,

    "modules": [
      "http_stub_status_module",
      "http_ssl_module",
      "http_gzip_static_module"
    ],

    "gzip_types": [
      "text/plain",
      "text/html",
      "text/css",
      "text/xml",
      "text/javascript",
      "application/json",
      "application/x-javascript",
      "application/xml",
      "application/xml+rss"
    ]
  },

  "apps": [
    {
      "user": "sloboda",
      "root": "/home/sloboda/sloboda",
      "server_name": "sloboda.dk",
      "rewrite_from": "www.sloboda.dk",
      "service": "unicorn-sloboda"
    }
  ],

  "mysql": {
    "server_root_password": "sacredroot",

	// какие базы данных создать
    // http://github.com/macovsky/chef-mysqldatabases
    "databases": [
      {
        "database": "sloboda",
        "username": "sloboda",
        "password": "sacredpassword"
      }
    ]
  },

  // общий список запускаемых рецептов
  "recipes": [
    "hostname",
    "locale",
    "user::data_bag",
    "nginx::source",
    "mysql::server",
    "database::mysql",
    "mysqldatabases",
    "rails",
    "imagemagick"
  ]
}
```
