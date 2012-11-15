Коробка для подъёма [Rails](http://rubyonrails.org)-приложений на [Debian Squeeze](http://wiki.debian.org/DebianSqueeze) через [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo):

* Установка [hostname](http://community.opscode.com/cookbooks/hostname)
* Установка дефолтной [locale](http://community.opscode.com/cookbooks/locale)
* Создание [пользователей](https://github.com/fnichol/chef-user)
* Установка [NGINX](http://community.opscode.com/cookbooks/nginx) из сорцов и дефолтная настройка
* Настройка [Rails](http://github.com/macovsky/chef-rails)-приложения: конфиг для NGINX и индивидуальный `/etc/init.d` скрипт для управления [Unicorn](http://unicorn.bogomips.org/)
* Установка [mysql](http://community.opscode.com/cookbooks/mysql) из пакета и создание [баз](http://github.com/macovsky/chef-rails)
* Установка [imagemagick](http://community.opscode.com/cookbooks/imagemagick) из пакета
