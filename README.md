# aws-ec2-nodejs-docker
Este `README.md` fornece uma visão geral e um guia detalhado sobre como configurar e implantar sua aplicação NodeJS no EC2 da AWS usando Docker, Docker Compose e Docker Machine.
o Docker, Docker Compose e Docker Machine para realizar o deploy de uma aplicação NodeJS ao EC2 da Amazon Web Services

Claro! Vamos integrar todo o conteúdo em um único arquivo `README.md`:

---

# NodeJS EC2 Docker Deployment

Este repositório contém um guia passo a passo para fazer o deploy de uma aplicação NodeJS em uma instância EC2 da AWS usando Docker, Docker Compose e Docker Machine.

## Índice

- [Pré-requisitos](#pré-requisitos)
- [Configuração Local](#configuração-local)
  - [Criar a Aplicação NodeJS](#criar-a-aplicação-nodejs)
  - [Criar o Dockerfile](#criar-o-dockerfile)
  - [Criar o Docker Compose File](#criar-o-docker-compose-file)
- [Configuração da Instância EC2 na AWS](#configuração-da-instância-ec2-na-aws)
  - [Criar uma Instância EC2](#criar-uma-instância-ec2)
  - [Conectar à Instância EC2](#conectar-à-instância-ec2)
- [Configuração do Docker e Docker Compose na EC2](#configuração-do-docker-e-docker-compose-na-ec2)
  - [Instalar Docker](#instalar-docker)
  - [Instalar Docker Compose](#instalar-docker-compose)
- [Provisionar e Gerenciar com Docker Machine](#provisionar-e-gerenciar-com-docker-machine)
  - [Instalar Docker Machine](#instalar-docker-machine)
  - [Criar a Máquina com Docker Machine](#criar-a-máquina-com-docker-machine)
  - [Configurar o Ambiente Docker](#configurar-o-ambiente-docker)
- [Deploy da Aplicação](#deploy-da-aplicação)
  - [Construir e Iniciar a Aplicação](#construir-e-iniciar-a-aplicação)
  - [Verificar a Aplicação](#verificar-a-aplicação)
- [Resumo](#resumo)

## Pré-requisitos

- Conta na AWS com permissões para criar instâncias EC2.
- Docker, Docker Compose e Docker Machine instalados em sua máquina local.
- Chave de acesso AWS (`access key`) e chave secreta (`secret key`).
- Conhecimento básico de NodeJS e Docker.

## Configuração Local

### Criar a Aplicação NodeJS

Certifique-se de ter uma aplicação NodeJS pronta. Se não tiver, crie um novo projeto com `npm init` e adicione seu código de aplicação.

### Criar o Dockerfile

Na raiz do projeto, crie um arquivo chamado `Dockerfile`:

```dockerfile
# Usar a imagem base do Node
FROM node:14

# Definir o diretório de trabalho
WORKDIR /usr/src/app

# Copiar os arquivos de configuração do Node
COPY package*.json ./

# Instalar as dependências
RUN npm install

# Copiar o restante do código da aplicação
COPY . .

# Expor a porta onde a aplicação irá rodar
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["npm", "start"]
```

### Criar o Docker Compose File

Na raiz do projeto, crie um arquivo `docker-compose.yml`:

```yaml
version: '3'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
```

## Configuração da Instância EC2 na AWS

### Criar uma Instância EC2

1. Acesse o Console da AWS e vá para o serviço EC2.
2. Clique em "Launch Instance" e selecione uma AMI, como "Amazon Linux 2" ou "Ubuntu".
3. Escolha o tipo de instância (ex.: `t2.micro` para testes).
4. Configure o armazenamento e as opções de segurança, garantindo que as portas 22 (SSH) e 3000 (para a aplicação NodeJS) estejam abertas.

### Conectar à Instância EC2

Use SSH para conectar-se à sua instância:

```bash
ssh -i "caminho/para/sua-chave.pem" ec2-user@ec2-your-public-ip.compute-1.amazonaws.com
```

## Configuração do Docker e Docker Compose na EC2

### Instalar Docker

Na instância EC2, execute os seguintes comandos para instalar o Docker:

Para Amazon Linux:

```bash
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
```

Para Ubuntu:

```bash
sudo apt-get update
sudo apt-get install docker.io -y
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

### Instalar Docker Compose

Na instância EC2, baixe e instale o Docker Compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Verifique a instalação com:

```bash
docker-compose --version
```

## Provisionar e Gerenciar com Docker Machine

### Instalar Docker Machine

Em sua máquina local, instale o Docker Machine:

```bash
base=https://github.com/docker/machine/releases/download/v0.16.2
curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine
sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

### Criar a Máquina com Docker Machine

Use o Docker Machine para criar uma máquina EC2:

```bash
docker-machine create --driver amazonec2 \
--amazonec2-access-key YOUR_ACCESS_KEY \
--amazonec2-secret-key YOUR_SECRET_KEY \
--amazonec2-region YOUR_REGION \
--amazonec2-instance-type t2.micro \
my-ec2-node
```

Substitua `YOUR_ACCESS_KEY`, `YOUR_SECRET_KEY`, `YOUR_REGION` e `my-ec2-node` pelos valores apropriados.

### Configurar o Ambiente Docker

Configure seu shell para usar o Docker na instância criada:

```bash
eval $(docker-machine env my-ec2-node)
```

## Deploy da Aplicação

### Construir e Iniciar a Aplicação

Com o ambiente configurado, execute:

```bash
docker-compose up --build -d
```

### Verificar a Aplicação

Acesse sua aplicação via navegador usando o IP público da sua instância EC2 na porta 3000: `http://<EC2_PUBLIC_IP>:3000`.

## Resumo

- Este projeto demonstra como configurar e fazer o deploy de uma aplicação NodeJS em uma instância EC2 usando Docker.
- Ele utiliza Docker Compose para orquestração de serviços e Docker Machine para provisionamento e gerenciamento da instância na AWS.
