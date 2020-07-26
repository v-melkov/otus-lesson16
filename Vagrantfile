# frozen_string_literal: true

# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV['LC_ALL'] = 'en_US.UTF-8'

MACHINES = {
  lesson16: {
    box_name: 'centos/8',
    ip_addr: '192.168.11.116'
  }
}.freeze

Vagrant.configure('2') do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vbguest.no_install = true
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s

      # box.vm.network "forwarded_port", guest: 8080, host: 8080

      box.vm.network 'private_network', ip: boxconfig[:ip_addr]

      box.vm.provider :virtualbox do |vb|
        vb.customize ['modifyvm', :id, '--memory', '1024']
        #        vb.gui = true
      end

      box.vm.provision 'shell', inline: <<~SHELL

        ###Домашнее задание Авторизация и аутентификация
        #Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников
        #Добавим группу и пользователей:
        groupadd admin
        useradd admin1 && echo "pass"|passwd --stdin admin1 && usermod -a -G admin admin1; \
        useradd notadmin1 && echo "pass"|passwd --stdin notadmin1
        #Добавим правила pam
        echo '*;*;!vagrant;!Wd0000-2400' >> /etc/security/time.conf
        sed -i '4iaccount [success=1 default=ignore] pam_succeed_if.so user ingroup admin' /etc/pam.d/system-auth
        sed -i '5iaccount required pam_time.so' /etc/pam.d/system-auth
        #Задание без звездочек выполнено
        ###
        #Разрешить пользователю рестарт докер-сервиса
        #Установим и запустим докер
        dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
        dnf install -y --nobest docker-ce
        systemctl enable --now docker
        systemctl status docker | head -n3

        #Добавим пользователя docker-user и добавим его в группу docker
        useradd docker-user && echo "pass"|passwd --stdin docker-user && usermod -a -G docker docker-user

        #После добавления в группу docker пользователю разрешена работа с docker-контейнерами (запуск, остановка и т.д.)
        #Рестарт сервиса требует аутентификацию root:
        #Добавим правило, разрешающее рестарт сервиса докер
cat <<'EOF' | tee /etc/polkit-1/rules.d/01-docker.rules
//правило разрешающее перезапуск сервиса докер пользователю docker-user
polkit.addRule(function(action, subject) {
  if (action.id == "org.freedesktop.systemd1.manage-units" && action.lookup("unit") == "docker.service" && action.lookup("verb") == "restart" && subject.user == "docker-user") {
      return polkit.Result.YES;
    }
});
EOF
      SHELL
    end
  end
end
