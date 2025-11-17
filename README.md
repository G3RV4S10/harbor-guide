# Guia de Instalação: Harbor no Debian 12 (Lab)

Estas são minhas anotações pessoais do processo de instalação do Harbor (Registry Privado), focadas em um ambiente de laboratório.

## 📝 Ambiente e Especificações

* **Versão do Harbor:** `v2.14.0` (Online Installer)
* **Sistema Operacional:** `Debian 12 (Bookworm)`
* **Verificação:** `md5sum`
* **Configuração:** `HTTP` (Porta 80) - Apenas para laboratório.

---

## 1. Pré-requisitos

* Um servidor Debian 12.
* `docker` e `docker-compose` instalados.
* Acesso `sudo`.

---

## 2. Instalação do Harbor

### Passo 2.1: Download do Instalador

Baixamos o instalador *online* e o arquivo de verificação `md5sum`.

```bash
# Baixar o instalador
wget [https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-online-installer-v2.14.0.tgz](https://github.com/goharbor/harbor/releases/download/v2.14.0/harbor-online-installer-v2.14.0.tgz)

# Baixar o arquivo de verificação
wget [https://github.com/goharbor/harbor/releases/download/v2.14.0/md5sum](https://github.com/goharbor/harbor/releases/download/v2.14.0/md5sum)
Passo 2.2: Verificação do Pacote (md5sum)
⚠️ Nota sobre a Documentação A documentação oficial do Harbor ou guias mais antigos podem mencionar a verificação com um arquivo *.asc (OpenPGP/GPG).

Para a versão v2.14.0, esse arquivo .asc não é fornecido. A verificação de integridade oficial é feita usando o md5sum, como mostrado abaixo.

Verificamos se o download não está corrompido:

Bash

md5sum -c md5sum
A saída esperada mostrará SUCESSO para o arquivo que baixamos e FALHA para o arquivo offline, que não baixamos (o que é normal):

md5sum: harbor-offline-installer-v2.14.0.tgz: Arquivo ou diretório inexistente
harbor-offline-installer-v2.14.0.tgz: FALHA na abertura ou na leitura
harbor-online-installer-v2.14.0.tgz: SUCESSO
md5sum: AVISO: 1 arquivo listado não pôde ser lido
Passo 2.3: Extração e Preparação
Extraímos o pacote e copiamos o arquivo de configuração de modelo.

Bash

# Extrair
tar xzvf harbor-online-installer-v2.14.0.tgz

# Entrar no diretório
cd harbor

# Criar o arquivo de configuração a partir do modelo
cp harbor.yml.tmpl harbor.yml
Passo 2.4: Configuração (harbor.yml)
Editamos o arquivo harbor.yml para configurar o ambiente de laboratório (HTTP).

Bash

nano harbor.yml
Alterações mínimas necessárias:

hostname: Alterar para o IP do seu servidor Debian.

YAML

hostname: 192.168.1.100 # <== SEU IP AQUI
harbor_admin_password: Definir uma senha segura para o usuário admin.

YAML

harbor_admin_password: SuaSenhaSuperSeguraAqui
Configurar para HTTP: Comentar (com #) a seção https e descomentar (remover #) a seção http e sua porta.

YAML

# https related config
# https:
#   port: 443
#   certificate: /your/certificate/path
#   private_key: /your/private/key/path

# http related config
http:
  # port for http, default is 80. If set to 0, http is disabled.
  port: 80
Passo 2.5: Executar a Instalação
Com a configuração pronta, executamos o script de instalação.

Bash

sudo ./install.sh
A instalação pode demorar vários minutos. Ao final, acesse o Harbor pelo IP (http://SEU.IP.DO.HARBOR).

3. Pós-Instalação: Configurar Cliente Docker
Para que o Docker possa se comunicar com este registry via HTTP, precisamos configurá-lo como um "insecure-registry" (registry não seguro).

Execute estes passos na máquina cliente (pode ser o próprio servidor Debian ou outra máquina na rede).

Passo 3.1: Criar/Editar daemon.json
Bash

sudo nano /etc/docker/daemon.json
Adicione o seguinte conteúdo, substituindo o IP pelo IP do seu Harbor:

JSON

{
  "insecure-registries" : ["192.168.1.100:80"]
}
Passo 3.2: Reiniciar o Docker
Salve o arquivo e reinicie o serviço do Docker para aplicar a mudança.

Bash

sudo systemctl restart docker
4. Teste de Validação (Push/Pull)
Na máquina cliente, faça o teste completo:

Login:

Bash

# Use o IP/porta do Harbor
docker login 192.168.1.100:80
# Usuário: admin
# Senha: (a que você definiu no harbor.yml)
Criar um Projeto: Na interface web do Harbor, crie um projeto (ex: teste).

Tag e Push:

Bash

# Baixar uma imagem de exemplo
docker pull hello-world

# "Taguear" a imagem para o seu registry
# Formato: [HOST]:[PORTA]/[PROJETO]/[IMAGEM]
docker tag hello-world 192.168.1.100:80/teste/hello-world

# Enviar a imagem para o Harbor
docker push 192.168.1.100:80/teste/hello-world
Se o push funcionar, vá até a interface web, abra seu projeto teste e você verá a imagem hello-world listada.
