# Post-Mortem: Falha de Inicialização do Immich (Crash Loop)

Este documento detalha o incidente de incompatibilidade de extensão vetorial no PostgreSQL durante o provisionamento da stack do Immich via Portainer, e as etapas executadas para estabilizar o serviço.

## 1. Cenário e Sintomas do Incidente

Após o deploy inicial da stack do Immich via Docker Compose, a interface web da aplicação não ficou acessível (retornando `ERR_CONNECTION_REFUSED` na porta `2283`). A inspeção no Portainer demonstrou que o contêiner `immich_server` estava reiniciando continuamente (Crash Loop).

## 2. Análise de Causa Raiz (Root Cause Analysis)

A análise da saída padrão de erros (stderr) do contêiner `immich_server` revelou o seguinte stack trace:

```text
microservices worker error: Error: No vector extension found. Available extensions: vchord, vector, stack: Error: No vector extension found.
at getVectorExtension (/usr/src/app/server/dist/repositories/database.repository.js:51:15)
microservices worker exited with code 1
```

**Diagnóstico:** O repositório oficial do Immich na versão `v3.1.0` descontinuou o suporte à extensão de banco de dados `pgvecto-rs`. O servidor web abortou a inicialização porque a imagem do PostgreSQL provisionada (`tensorchord/pgvecto-rs:pg14-v0.2.0`) não atendia ao novo requisito de dependência vetorial oficial (`pgvector`).

## 3. Resolução e Mitigação

A correção exigiu a refatoração do arquivo declarativo (YAML) da infraestrutura e o descarte do volume de dados contaminado com a estrutura antiga. As seguintes ações foram tomadas:

1. **Substituição da Imagem:** A imagem do microsserviço de banco de dados foi atualizada para a versão com suporte nativo exigida pelo Immich (`pgvector/pgvector:pg14`).
2. **Limpeza de Argumentos:** Remoção dos parâmetros obsoletos (`command: ["postgres", "-c", "shared_preload_libraries=vectors.so"...]`) que injetavam a extensão antiga.
3. **Prevenção de Corrupção de Dados:** O ponto de montagem (Bind Mount) foi alterado de `/mnt/immich_data/postgres` para um diretório virgem `/mnt/immich_data/pg_data`, garantindo que o novo banco de dados iniciasse com o *schema* correto, sem conflito de I/O com os artefatos anteriores.

**Bloco YAML Corrigido:**

```yaml
database:
  container_name: immich_postgres
  image: docker.io/pgvector/pgvector:pg14
  environment:
    - POSTGRES_PASSWORD=immich_db_pass_2026
    - POSTGRES_USER=postgres
    - POSTGRES_DB=immich
    - POSTGRES_INITDB_ARGS='--data-checksums'
  volumes:
    - /mnt/immich_data/pg_data:/var/lib/postgresql/data
  restart: always
```

## 4. Validação

Após o *Update* da *stack* no Portainer, os logs do `immich_server` normalizaram, as portas foram expostas com sucesso e a tela de *Getting Started* ficou disponível na rede local.
