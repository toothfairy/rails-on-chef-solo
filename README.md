Коробка для подъёма [Rails](http://rubyonrails.org)-приложений на [Debian Squeeze](http://wiki.debian.org/DebianSqueeze) через [Chef Solo](http://wiki.opscode.com/display/chef/Chef+Solo)

* Установка [hostname](http://community.opscode.com/cookbooks/hostname)
* Установка дефолтной [locale](http://community.opscode.com/cookbooks/locale) `en_US.utf8` 
* Создание [пользователей](https://github.com/fnichol/chef-user), указанные ключи добавляются в `~/.ssh/authorized_keys`, для пользователя генерируется ключ
* Сборка последней стабильной версии [NGINX](http://community.opscode.com/cookbooks/nginx) из сорцов и дефолтная настройка
* Настройка [Rails](http://github.com/macovsky/chef-rails)-приложений: конфиг для NGINX и индивидуальный сервис для управления [Unicorn](http://unicorn.bogomips.org/)
* Установка [MySQL](http://community.opscode.com/cookbooks/mysql) из пакета, установка пароля для `root` и создание [баз](http://github.com/macovsky/chef-rails)
* Установка [ImageMagick](http://community.opscode.com/cookbooks/imagemagick) из пакета

**Установка, под рутом**

Обновим свежую систему, установим dev-пакеты и поставим [rbenv](http://github.com/sstephenson/rbenv) и Ruby 1.9.3:

```bash
# Update, upgrade and install development tools:
apt-get update
apt-get -y upgrade
apt-get -y install build-essential git-core curl libssl-dev \
                   libreadline5 libreadline5-dev \
                   zlib1g zlib1g-dev \
                   libmysqlclient-dev \
                   libcurl4-openssl-dev \
                   libxslt-dev libxml2-dev

# Install rbenv
git clone git://github.com/sstephenson/rbenv.git /usr/local/rbenv

# Add rbenv to the path:
echo '# rbenv setup' > /etc/profile.d/rbenv.sh
echo 'export RBENV_ROOT=/usr/local/rbenv' >> /etc/profile.d/rbenv.sh
echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> /etc/profile.d/rbenv.sh
echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh

chmod +x /etc/profile.d/rbenv.sh
source /etc/profile.d/rbenv.sh

# Install ruby-build:
pushd /tmp
  git clone git://github.com/sstephenson/ruby-build.git
  cd ruby-build
  ./install.sh
popd

# Install Ruby 1.9.3-p194:
rbenv install 1.9.3-p194
rbenv global 1.9.3-p194
rbenv rehash

# Production installing gems skipping ri and rdoc
cat << EOF > /root/.gemrc
---
:sources:
- http://gems.rubyforge.org
- http://gems.github.com
gem: --no-ri --no-rdoc
EOF
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

**Настройка**

Теперь надо сконфигурировать ноду, скопируем файл с примером:

`cp node.json.example node.json`

и откроем `node.json` (на всякий случай напомню, что в `json` не бывает комментов :-1:):

```javascript
{
  // hostname
  "set_fqdn": "sloboda.dk",

  // пользователи
  // https://github.com/fnichol/chef-user
  // нужно создать одноимённые databags, например databags/users/sloboda.json
  // смотрите databags/users/sloboda.json.example
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

  // настройки Rails-приложений
  // смотрите подробно: https://github.com/macovsky/chef-rails
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

**Запуск**

После команды `chef-solo` и краткого ожидания остаётся только задеплоить приложение.

Смотрите особенности настройки приложений в рецепте [chef-rails](https://github.com/macovsky/chef-rails).

:satisfied:

Протестировано на Debian Squeeze 64.