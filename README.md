 [Link para baixar o Ubuntu Server 20.04]([https://raw.githubusercontent.com/larcc-group/Escola-de-inverno-2022-zabbix/main/nvidia.xml?token=GHSAT0AAAAAABW3RGC2WRPZKS5AEV2VQNXUYXAKQMQ](https://ubuntu.com/download/server).

[Link para baixar o VMWARE](https://www.vmware.com/br/products/workstation-player.html).

# Host Server zabbix
### Atualizando o sistema
sudo apt update
### Instalando o apache
sudo apt install apache2
### Habilitando o apache e fazendo o start
systemctl start apache2
systemctl enable apache2
### visualizando o status do apache
systemctl status apache2
### Instalando o php e seus pacotes
sudo apt install php-cli php-common php-dev php-pear php-gd php-mbstring php-mysql php-xml php-bcmath libapache2-mod-php
### Configurando o timezone do php
cd /etc/php/7.4/
### Editando o arquivo php.ini
nano apache2/php.ini
nano cli/php.ini
### Em date.timezone = Asia/Singapore
### Substituir por  date.timezone = America/Sao_Paulo
Restart do apache
systemctl restart apache2
### Instalando o mariadb
sudo apt install mariadb-server mariadb-client
### Iniciando e fazendo start do banco
systemctl start mariadb
systemctl enable mariadb
### Criando uma senha para o banco
mysql_secure_installation
### Logando no banco de dados
mysql -u root -p
### Criando e configurando o banco
create database zabbix character set utf8 collate utf8_bin;

grant all privileges on zabbix.* to zabbix@'localhost' identified by 'senha do banco inserida anteriormente';

grant all privileges on zabbix.* to zabbix@'%' identified by 'senha do banco inserida anteriormente';

flush privileges;

quit

### Baixando o zabbix
wget -q https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+focal_all.deb
### Instalando o zabbix
sudo dpkg -i zabbix-release_5.0-1+focal_all.deb
### Atualizando o sistema
sudo apt update
### Instalando o mysql
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-agent
### Importando as tabelas do banco
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
### Configurando o zabbix server
nano /etc/zabbix/zabbix_server.conf
### Descomentar as linhas e informar a senha do seu banco
DBHost=localhost

DBPassword=zabbix
### Salvar e sair
### habilitando e fazendo start no xabbix-server
systemctl start zabbix-server

systemctl enable zabbix-server
### Conferir se o zabbix-server está funcionando
systemctl status zabbix-server
### Visualizar os logs do zabbix-server
tail -f /var/log/zabbix/zabbix_server.log

# Host Client zabbix
### Baixando o repositório do zabbix 5.0
wget https://repo.zabbix.com/zabbix/5.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.0-1+focal_all.deb
### Instalando o zabbix-release_5.0-1+focal_all.deb
sudo apt install ./zabbix-release_5.0-1+focal_all.deb
### Atualizando o sistema
sudo apt update
### Configurando o zabbiz agent
sudo nano /etc/zabbix/zabbix_agentd.conf
### No arquivo zabbix_agentd.conf é preciso fazer algumas alterações como por exemplo:
### Conferir o ip do server.
Server= 127.0.0.1
### Em caso de haver algum problema na execução do zabbix, recomenda-se comentar o ServerActive 
#ServerActive=127.0.0.1
### Alterar o nome do seu host “agente”. No exemplo abaixo foi utilizado um exemplo da
### configuração trocando somente o nome dex.
Hostname=dex

# GPU
 Para saber qual a versão da placa de vídeo e o drive que ela suporta é só digitar
**nvidia-smi**, a partir das informações mostradas é só digitar **sudo apt install nvidia-versão_disponível**. Ex: **sudo apt install nvidia-340**.
	Após a instalação do driver é necessário reiniciar o sistema, digitando **sudo reboot**. E logo em seguida é necessário  fazer o restart do zabbix sudo systemctl restart zabbix-agent. Agora vamos fazer o start **sudo systemctl start zabbix-agent** e para visualizar o zabbix está funcionando corretamente é só digitar o **sudo systemctl status zabbix-agent**.
 
Em caso de haver uma GPU é necessário inserir no agente do zabbix zabbix_agentd.conf parâmetros no para que o zabbix web consiga acessar os dados. Podem ser encontrados [Aqui](https://github.com/RichardKav/zabbix-nvidia-smi-integration/).

UserParameter=gpu.temp,nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits -i 0

UserParameter=gpu.memtotal,nvidia-smi --query-gpu=memory.total --format=csv,noheader,nounits -i 0

UserParameter=gpu.used,nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits -i 0

UserParameter=gpu.free,nvidia-smi --query-gpu=memory.free --format=csv,noheader,nounits -i 0

UserParameter=gpu.fanspeed,nvidia-smi --query-gpu=fan.speed --format=csv,noheader,nounits -i 0

UserParameter=gpu.utilisation,nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader,nounits -i 0

UserParameter=gpu.power,nvidia-smi --query-gpu=power.draw --format=csv,noheader,nounits -i 0

### Após a configuração é necessário salvar as configurações utilizando o Ctrl + x e logo vai aparecer uma mensagem que é necessário confirmar digitando y e enter.
## Reiniciando o agente
sudo systemctl restart zabbix-client
### Para visualizar o log do agente
sudo tail -f /var/log/zabbix/zabbix_agentd.log



# Zabbix Web
 Para acessar o zabbix web é necessário digitar o ip do zabbix server/zabbix. Após é direcionado para a página de login do zabbix, então é necessario inserir o usuario que por padrão denomina-se **Admin** e a senha **zabbiz**.

 Ao fazer o login, vá para o menu lateral, clicando em Configuração -> “host” e inserir o **Template OS Linux by Zabbix agent**. Para inserir o segundo template  do monitoramento dos sensores de uma placa de vídeo nvidia é necessário fazer a importação do template, pois esse não é disponibilizado pelo zabbix na parte web. O template utilizado pode ser encontrado [Aqui](https://raw.githubusercontent.com/larcc-group/Escola-de-inverno-2022-zabbix/main/nvidia.xml?token=GHSAT0AAAAAABW3RGC2WRPZKS5AEV2VQNXUYXAKQMQ). o mesmo deve ser copiado com a extensão  e colado em Link new templates no zabbix o web.

### ![host](https://user-images.githubusercontent.com/53538228/178557897-c308ae4c-ea43-45e3-9918-698abfb4536f.png)

