Коробка для подъёма [Rails](http://rubyonrails.org)-приложений на [Debian Squeeze](http://wiki.debian.org/DebianSqueeze) через [chef-solo](http://wiki.opscode.com/display/chef/Chef+Solo):

* Установка [hostname](http://community.opscode.com/cookbooks/hostname)
* Установка дефолтной [locale](http://community.opscode.com/cookbooks/locale)
* Создание [пользователей](https://github.com/fnichol/chef-user)
* Установка [nginx](http://community.opscode.com/cookbooks/nginx) и дефолтная настройка
* Настройка [rails](http://github.com/macovsky/chef-rails)-приложения: конфиг для nginx и индивидуальный /etc/init.d скрипт для управления Unicorn
* Установка [mysql](http://community.opscode.com/cookbooks/mysql) из пакета и создание [databases](http://github.com/macovsky/chef-rails)
* Установка [imagemagick][http://community.opscode.com/cookbooks/imagemagick] из пакета
