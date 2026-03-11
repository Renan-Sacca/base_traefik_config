# Base Traefik Config

Este é um projeto de configuração base para o **Traefik v3.0**, utilizando Docker Compose. Ele funciona como um proxy reverso e gerenciador de certificados SSL (Let's Encrypt) automáticos, servindo aplicações containerizadas através de uma rede externa comum e fornecendo redirecionamento global de HTTP para HTTPS de forma transparente.

## Características

- Imagem **Traefik v3.0** configurada e pronta para uso.
- Provedor do Docker ativado (apenas containers expostos explicitamente serão pegos pelo proxy).
- Pontos de entrada para **HTTP (porta 80)** e **HTTPS (porta 443)**.
- **Redirecionamento global** de tráfego HTTP para HTTPS habilitado por padrão.
- Desafio **TLS do Let's Encrypt** (`tlschallenge`) para resolução/geração automática de certificados SSL.
- Persistência dos certificados de Let's Encrypt através de um volume local `letsencrypt/`.
- Rede externa chamada `proxy` para facilmente rotear o tráfego a outros containers/serviços conectados na mesma.
- Logs com nível `INFO` detalhados para monitoramento contínuo.

## Pré-requisitos

1.  **Docker** e **Docker Compose** instalados (versão `3.9` ou superior recomendada).
2.  Portas **80** e **443** livres no host.
3.  Você precisa criar a rede externa `proxy` (se ainda não existir):
    ```bash
    docker network create proxy
    ```

## Executando o Proxy

1. Clone o repositório em sua máquina.
2. Certifique-se de que a rede `proxy` existe (ver passo acima).
3. Levante o serviço com o Docker Compose utilizando o modo *detached*:

    ```bash
    docker compose up -d
    ```

## Configurando Containers Clientes

Para conectar outros containers a este proxy, você precisará adicionar as labels do Traefik neles.

Exemplo de configuração para outro `docker-compose.yml`:
```yaml
services:
  minha-aplicacao:
    image: minha-imagem:latest
    networks:
      - proxy
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.minha-aplicacao.rule=Host(`meusite.com`)"
      - "traefik.http.routers.minha-aplicacao.entrypoints=websecure"
      - "traefik.http.routers.minha-aplicacao.tls.certresolver=letsencrypt"

networks:
  proxy:
    external: true
```

*Nota: O provedor `exposedbydefault` está igual a `false`. Sendo assim, lembre-se sempre de colocar `traefik.enable=true` nos containers alvos para que o Traefik reconheça as configurações e inicie o roteamento.*
