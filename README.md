# Autenticação e Keycloak

## Introdução

### OAuth 2.0

Protocolo padrão que define regras para **autorização** (o que posso acessar?). Possui diferentes fluxos de autorização definidos para atender a diferentes necessidades: sistemas web, acesso entre serviços (sem interface para usuário), etc.

Composto de 4 elementros principias (roles):

- Resource owner (proprietário do recurso): usuário ou aplicativo que possui o recurso protegido que precisa ser acessado. Exemplo: um app precisa acessar seus dados do facebook para fazer autenticação, você é o proprietário do recurso (dados do facebook) e o único que pode conceder acesso.
- Authorization server (servidor de autorização): servidor que autentica o proprietário do recurso e concede permissões de acesso ao aplicativo solicitante ao servidor de recurso (através de tokens de acesso). Exemplo: Servidor do Facebook onde está implementado OAuth 2.0.
- Client (Aplicativo solicitante): aplicativo que solicita acesso ao recurso protegido. Deve possuir credenciais de acesso para autenticar-se junto ao servidor de autorização. Exemplo: aplicativo que precisa das informações do Facebook do usuário.
- Resource server: servidor onde o recurso protegido está hospedado. Retorna os recursos apropriados de acordo como o token que recebe. Exemplo: servidor do Facebook onde estão as informações do usuário.

### Open ID Connect

Principal protocolo de **autenticação** (posso acessar algo?) existente no mercado, é uma camada de autenticação baseada em token sobre o protocolo OAuth 2.0. Permite que usuários se autentiquem em diferentes sites usando identidade única: Single Sign-On (SSO).

### Keycloak

https://github.com/keycloak/keycloak

#### Características

- Suporte a múltiplataformas: Java, Node.ks, .NET, etc;
- Integração com diversos provedores de identidades: LDAP, Microsoft AD, Google, Facebook, etc.
- Administração centralizada.
- Proteção de API: disponibiliza tokens de acesso baseados em OAuth 2.0 e OpenID Connect.
- Personalização da tela login.
- Suporte multi-tenancy: através do cadastro de diferentes realms.
- Integração com DevOps.
- Extensibilidade: pode ser facilmente estendido com plugins e APIs personalizadas.
- Alta escalabilidade.
- Open-source.

#### Funcionalidades

- Autenticação de usuários: disponibiliza diferentes métodos de autenticação como login/senha, autenticação de multifator, SSO, etc.
- Gerenciamento de usuários.
- Autorização de usuários através de papeis e permissões.
- Integração com diferentes protocolos de autenticação e autorização: OAuth 2.0, OpenID Connect e SAML 2.0.
- Configuração flexível.
- Gerenciamento de sessão: controle de expiração, renovação e encerramento de sessões dos usuários.
- Autenticação e autorização baeadas em níveis.

##### Principais casos de uso

- Single Sign-On.
- Proteção de API.
- Gerenciamento de usuários.
- Autenticação multifator: SMS, e-mail, aplicativo móvel.
- Integração com diferentes sistemas.
- Gerenciamento de sessão.
- Autorização baseada em papéis.

#### Rodando Keycloak com Docker Compose

A imagem oficial e atualizada do Keycloak está disponível no site [Quary IO](https://quay.io/repository/keycloak/keycloak), que é o repositório de imagens oficial da Red Hat.

```YAML
# docker-compose.yaml

version: '3'

services:
  keycloak:
    image: quay.io/keycloak/keycloak:21.1
    command: start-dev #Configurações para o KC em dev
    ports:
      - 8080:8080
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
```

Subir o KC no docker: `docker compose up`
Acessar o KC via browser: `localhost:8080`

Mias informações sobre o container do KC: https://www.keycloak.org/server/containers

#### Integrando MySQL com Keycloak

Por padrão o KC utiliza o banco de dados **H2** que fica no mesmo container que o serviço, ou seja, uma vez que o container é finalizado tudo o banco é removido.

```YAML
docker-compose.yaml
version: '3'

services:

  keycloak:
    image: quay.io/keycloak/keycloak:21.1
    command: start-dev
    ports:
      - 8080:8080
    environment:
      - KEYCLOAK_ADMIN=admin
      - KEYCLOAK_ADMIN_PASSWORD=admin
      - KC_DB=mysql
      - KC_DB_URL=jdbc:mysql://db:3306/keycloak
      - KC_DB_USERNAME=root
      - KC_DB_PASSWORD=root
    depends_on:
      db:
        condition: service_healthy #Container do KC inicia depois que o container do banco de dados estiver healthy

  db:
    image: mysql:8.0.39-debian
    ports:
      - 3306:3306
    volumes:
      - ./.docker/dbdata:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=keycloak
    security_opt:
      - seccomp:unconfined #Desabilita restrições do MySQL (não fazer em produção)
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      interval: 5s
      timeout: 10s
      retries: 3

# Subir o kc: docker compose up
# Acessar o kc: localhost:8080
```

Como o banco de dados está trabalhando com volume, é recomendado incluir o volume local no arquivo .gitignore
