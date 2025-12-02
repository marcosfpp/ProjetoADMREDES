# ProjetoADMREDES

### Marcos Felipe Pessoa Pacheco e Marcus Vinicius

Trabalho final desenvolvido para a disciplina de Administração de Redes de Computadores, no curso de Sistemas de Informação. O objetivo do projeto foi montar um ambiente de rede virtualizado utilizando pfSense e Linux Mint, configurando serviços essenciais como DHCP, DNS, Apache, FTP e NFS. A topologia foi criada no VirtualBox e todas as máquinas foram isoladas em uma rede interna exclusiva para testes.

## 1. Topologia da Rede

O ambiente foi composto por três máquinas virtuais:

1. pfSense – Servidor DHCP e DNS

O pfSense atuou como o servidor principal da rede. Ele possui duas placas de rede:

WAN (Bridge) – recebe IP automaticamente da rede física

LAN (Rede Interna) – configurada manualmente, com o IP:
172.16.0.25/24

Essa interface foi responsável por distribuir IPs via DHCP e responder consultas DNS.

## 2. Linux Mint – Servidor (Apache, FTP e NFS)

O servidor ficou configurado com a placa de rede em Rede Interna, recebendo IP fixo através do DHCP do pfSense.
Como o range do servidor seria de 172.16.0.50 até 172.16.0.200, esses foram os IPs que o DHCP forneceu.

IP final do servidor:
👉 172.16.0.50

Nesta máquina foram instalados e configurados:

Servidor web Apache

Servidor FTP (vsftpd)

Servidor NFS

## 3. Linux Mint – Cliente

A terceira VM também ficou em Rede Interna, recebendo IP automático pelo pfSense.

IP final do cliente:
👉 172.16.0.51

Essa máquina foi usada para testar todos os serviços implantados.

2. Configuração do DHCP no pfSense

Dentro da interface web do pfSense, acessamos:
Services → DHCP Server → LAN

As principais configurações foram:

DHCP ativado na LAN

Rede: 172.16.0.0/24

Range para clientes: 172.16.0.50 – 172.16.0.200

Gateway: 172.16.0.25

DNS: 172.16.0.25

Foi criado também um mapeamento estático para o servidor Linux Mint, garantindo sempre o mesmo IP:

MAC cadastrado → IP reservado 172.16.0.50

Com isso, o servidor recebe sempre o mesmo endereço.

3. Configuração do DNS no pfSense

Ajustes realizados em:
Services → DNS Resolver

Configurações principais:

DNS Resolver ativado

Porta 53 (padrão)

Interfaces escutando: All

Interface de saída: WAN

Foram adicionados dois registros HOST:

servidor.lan → 172.16.0.50

cliente.lan → 172.16.0.51

Assim, basta digitar servidor.lan na rede para resolver o IP do servidor.

## 4. Servidor Apache (Máquina 172.16.0.50)

O Apache foi instalado no servidor Linux Mint com:

sudo apt install apache2 -y


Após a instalação, verificamos se estava ativo:

systemctl status apache2


Para testar, no cliente basta abrir o navegador e acessar:

http://172.16.0.50


A página padrão do Apache confirmou o funcionamento.

## 5. Servidor FTP (vsftpd)
Instalação no servidor
sudo apt update
sudo apt upgrade
sudo apt install vsftpd -y

Correção de erros comuns

Durante a configuração, ocorreram:

1. Erro de login (500 OOPS: refusing to run with writable root)

Resolvido adicionando ao arquivo:

sudo nano /etc/vsftpd.conf

chroot_local_user=YES

2. Erro ao rodar IPv6 e IPv4 simultaneamente

Ajustado com:

listen=YES
listen_ipv6=NO


Reinício do serviço:

sudo systemctl restart vsftpd


Para confirmar:

sudo systemctl status vsftpd

Teste pelo cliente

No cliente (172.16.0.51), conectamos ao servidor FTP:

ftp 172.16.0.50


Após inserir usuário e senha do servidor, foi possível:

ls      (listar arquivos)
put     (enviar arquivo)
get     (baixar arquivo)

## 6. Servidor NFS
No servidor (172.16.0.50)

Instalação:

sudo apt install nfs-kernel-server


Criação da pasta compartilhada:

sudo mkdir -p /servidor/pastacomp
sudo chmod 777 /servidor/pastacomp


Configuração no arquivo:

sudo nano /etc/exports


Inserido:

/servidor/pastacomp 172.16.0.0/24(rw,sync,no_root_squash)


Aplicar:

sudo exportfs -ra
sudo exportfs -v

No cliente (172.16.0.51)

Instalar suporte ao NFS:

sudo apt install nfs-common


Criar ponto de montagem:

sudo mkdir -p /cliente/pastacomp


Montar diretório:

sudo mount 172.16.0.50:/servidor/pastacomp /cliente/pastacomp


## 7. Testes do NFS

Teste simples:

Criar um arquivo no servidor em /servidor/pastacomp

Abrir no cliente /cliente/pastacomp

O arquivo aparece automaticamente

Movimentações funcionam nos dois lados

Isso confirmou o compartilhamento NFS.

Conclusão

O projeto permitiu montar uma rede isolada e funcional utilizando pfSense e Linux Mint, configurando serviços essenciais usados em ambientes reais. Todas as máquinas se comunicaram corretamente, o servidor recebeu IP fixo via DHCP, o DNS respondeu conforme configurado e os serviços Apache, FTP e NFS funcionaram com sucesso entre o servidor e o cliente.
