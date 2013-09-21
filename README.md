Коробка для подъёма [Rails](http://rubyonrails.org)-приложений на [Debian Squeeze](http://wiki.debian.org/DebianSqueeze) через [Chef Solo](http://wiki.opscode.com/display/chef/Chef+Solo)

* Установка [hostname](http://community.opscode.com/cookbooks/lxmx_hostname)
* Установка дефолтной [locale](http://community.opscode.com/cookbooks/locale) `en_US.utf8`
* Создание [пользователей](https://github.com/fnichol/chef-user), указанные ключи добавляются в `~/.ssh/authorized_keys`, для пользователя генерируется ключ
* Сборка последней стабильной версии [NGINX](http://community.opscode.com/cookbooks/nginx) из сорцов и дефолтная настройка
* Настройка [Rails](http://github.com/macovsky/chef-rails)-приложений: конфиг для NGINX и индивидуальный сервис для управления [Unicorn](http://unicorn.bogomips.org/)
* Установка [MySQL](http://community.opscode.com/cookbooks/mysql) из пакета, установка пароля для `root` и создание [баз](http://github.com/macovsky/chef-rails)
* Установка [ImageMagick](http://community.opscode.com/cookbooks/imagemagick) из пакета
* Настройка [OpenSSH](https://github.com/opscode-cookbooks/openssh)

**Установка, под рутом**

Чистую систему обновим, установим dev-пакеты и поставим [rbenv](http://github.com/sstephenson/rbenv) и Ruby 2.0:

```bash
apt-get update
apt-get -y upgrade
apt-get -y install git-core libssl-dev

# Install rbenv
git clone git://github.com/sstephenson/rbenv.git /usr/local/rbenv
git clone git://github.com/sstephenson/ruby-build.git /usr/local/rbenv/plugins/ruby-build

# Add rbenv to the path:
echo '# rbenv setup' > /etc/profile.d/rbenv.sh
echo 'export RBENV_ROOT=/usr/local/rbenv' >> /etc/profile.d/rbenv.sh
echo 'export PATH="$RBENV_ROOT/bin:$PATH"' >> /etc/profile.d/rbenv.sh
echo 'eval "$(rbenv init -)"' >> /etc/profile.d/rbenv.sh

chmod +x /etc/profile.d/rbenv.sh
source /etc/profile.d/rbenv.sh

# Install Ruby:
rbenv install 2.0.0-p247
rbenv global 2.0.0-p247

# Install gems skipping ri and rdoc
cat << EOF > /root/.gemrc
---
:sources:
- http://gems.rubyforge.org
- http://gems.github.com
gem: --no-ri --no-rdoc
EOF
```

Потом поставим [chef](http://www.opscode.com/chef/) + [ruby-shadow](https://github.com/apalmblad/ruby-shadow), [librarian](https://github.com/applicationsonline/librarian-chef) для менеджмента cookbooks и [bundler](http://gembundler.com) на будущее:

```bash
gem i chef ruby-shadow librarian-chef bundler
rbenv rehash
```

Скачиваем коробку и устанавливаем cookbooks из `Cheffile`:

```bash
git clone git://github.com/macovsky/rails-on-chef-solo.git /etc/chef
cd /etc/chef
librarian-chef install
```

**Конфигурация**

Теперь надо сконфигурировать ноду, скопируем файл с примером:

`cp node.json.example node.json`

и откроем `node.json` (на всякий случай напомню, что в `json` не бывает комментов :-1:):

```javascript
{
  // hostname
  "net": {
    "hostname": "sloboda.dk"
  },

  // пользователи
  // https://github.com/fnichol/chef-user
  // нужно создать одноимённые databags, в данном случае databags/users/sloboda.json
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
  // https://github.com/macovsky/chef-rails
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
    "server_root_password": "iloverandompasswordsbutthiswilldo",
    "server_repl_password": "iloverandompasswordsbutthiswilldo",
    "server_debian_password": "iloverandompasswordsbutthiswilldo",

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

  // https://github.com/opscode-cookbooks/openssh
  "openssh": {
    "server": {
      // из соображений безопасности полезно поменять дефолтный 22-й порт
      "port": "22",
      "protocol": "2",
      "use_privilege_separation": "yes",
      "ignore_rhosts": "yes",
      "permit_root_login": "no",
      // поменять на yes, если хотите заходить под паролями
      "password_authentication": "no",
      "permit_empty_passwords": "no"
    }
  },

  "run_list": [
    "recipe[lxmx_hostname]",
    "recipe[locale]",
    "recipe[user::data_bag]",
    "recipe[nginx::source]",
    "recipe[mysql::client]",
    "recipe[mysql::server]",
    "recipe[database::mysql]",
    "recipe[mysqldatabases]",
    "recipe[rails]",
    "recipe[imagemagick]",
    "recipe[openssh]"
  ]
}
```

**Установка**

После команды `chef-solo` и краткого ожидания остаётся задеплоить приложение, запустить `nginx` и добавить сервис для `unicorn` в автозапуск.

Смотрите особенности настройки приложений в рецепте [chef-rails](https://github.com/macovsky/chef-rails).

Протестировано на [Debian Squeeze 64](http://wiki.debian.org/DebianSqueeze) — http://dl.dropbox.com/u/54390273/vagrantboxes/Squeeze64_VirtualBox4.2.4.box.
