# Operação Ufology — ufoTracker

[![CI](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/gradle-ci.yml?branch=main&label=CI)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/gradle-ci.yml)
[![Tests](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/tests.yml?label=Tests)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/tests.yml)
[![Run Monitor](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/run-monitor.yml?branch=main&label=Run%20Monitor)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/run-monitor.yml)
[![Hello CI/CD](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/hello.yml?label=Hello%20CI%2FCD)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/hello.yml)
[![Env Demo](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/env-demo.yml?label=Env%20Demo)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/env-demo.yml)
[![Secret Demo](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/secret-demo.yml?label=Secret%20Demo)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/secret-demo.yml)

Sistema de rastreamento de avistamentos UFO da **Operação Ufology** (AREA 51).
Stack: **Java 21 + Spring Boot 3.5.4 + Maven + PostgreSQL + Redis**,
empacotado em imagem Docker multi-stage e operado em cluster **Kubernetes**
no namespace `ufology`.

---

## 1. Git no Ciclo de DevOps e Entrega Contínua

O Git é o **sistema nervoso central** de qualquer pipeline de CI/CD — é a
partir do estado do repositório e dos seus eventos (`push`, `pull_request`,
`tag`, `release`) que se desencadeiam todos os passos automatizados de
build, teste, empacotamento e deploy.

Num ciclo de DevOps, o Git cumpre três papéis inseparáveis:

1. **Fonte única da verdade** — o código, a infraestrutura como código
   (`k8s/`, `Dockerfile`) e a própria definição dos pipelines
   (`.github/workflows/`) residem no mesmo repositório. Qualquer alteração
   produtiva passa obrigatoriamente por um commit, o que garante
   reprodutibilidade e auditoria completa.
2. **Gatilho de automação** — cada evento Git aciona um conjunto específico
   de workflows: `push` em `main` dispara o CI, a abertura de um
   `pull_request` dispara validações antes do merge, e a publicação de uma
   `tag` semântica dispara o pipeline de release.
3. **Trilha de auditoria** — o histórico de commits, pull requests, tags e
   releases serve de registro imutável de quem entregou o quê e quando,
   requisito típico de processos de compliance (ISO 27001, SOC 2).

---

## 2. Branches e Tags

### 2.1 Branches

Branches isolam **contextos de trabalho** e permitem paralelizar o
desenvolvimento sem arriscar a estabilidade do código em produção. Este
projeto adota um fluxo minimalista derivado do *GitHub Flow*:

| Branch      | Propósito                                     | Gatilhos de CI                    |
|-------------|-----------------------------------------------|-----------------------------------|
| `main`      | Linha estável; recebe código validado por PR  | `push` → CI + Run Monitor        |
| `ci/setup`  | Desenvolvimento dos workflows e infra de CI   | `push` → Hello CI, Env/Secret Demo |
| `feature/*` | Novas funcionalidades de aplicação            | `pull_request` → Tests            |

A separação entre branches permite configurar **regras de proteção** em
`main` (exigir PR, exigir CI verde, exigir aprovação humana) sem bloquear o
trabalho do dia a dia em outras branches.

### 2.2 Tags e Releases

Enquanto branches são **móveis**, tags marcam **pontos imutáveis** do
histórico — tipicamente cortes de release versionados com SemVer
(`v1.0.0`, `v1.1.0`, `v2.0.0-rc1`). O valor estratégico está em três
dimensões:

- **Rastreabilidade**: uma tag permite reconstruir exatamente o código que
  foi promovido para produção em determinada data.
- **Separação CI/CD**: workflows com `on: push: tags:` disparam apenas no
  corte da release, separando o pipeline contínuo (CI, por `push`) do
  pipeline pontual de entrega (CD, por `tag`).
- **Releases GitHub**: a interface de *Releases* do GitHub anexa a tag a
  notas de versão, binários e artefatos de deploy, servindo de ponto de
  entrega formal para stakeholders.

### 2.3 Runners: GitHub-hosted vs Self-hosted

| Característica         | GitHub-hosted                             | Self-hosted                                                              |
|------------------------|-------------------------------------------|--------------------------------------------------------------------------|
| Provisionamento        | Automático pela plataforma                | Responsabilidade do time                                                 |
| Manutenção             | Zero                                      | Requer atualizações e monitoramento                                      |
| Custo                  | Incluso nas cotas do plano                | Infraestrutura e operação próprias                                       |
| Acesso a rede interna  | Não                                       | Sim                                                                      |
| Caso de uso típico     | Build, testes, lint em projetos públicos  | Deploy em cluster Kubernetes privado, acesso a VPN, ambientes air-gapped |

Runners self-hosted tornam-se necessários quando o job de deploy precisa
alcançar infraestrutura interna sem exposição pública — como seria o caso
do cluster Kubernetes desta operação num ambiente corporativo real.

---

## 3. Estrutura do Repositório

```
.
├── .github/workflows/
│   ├── gradle-ci.yml        # Build + testes (CI principal)
│   ├── run-monitor.yml      # Monitor + deploy (environment: production)
│   ├── tests.yml            # Testes em PRs
│   ├── hello.yml            # Pipeline introdutório
│   ├── env-demo.yml         # Demonstração de vars
│   └── secret-demo.yml      # Demonstração de secrets
├── k8s/
│   ├── namespace.yaml       # namespace ufology
│   ├── app-configmap.yaml   # ConfigMap app-config (DB_NAME)
│   ├── app-secret.yaml      # Secret db-secret (DB_PASSWORD)
│   ├── app-deployment.yaml  # 2 réplicas ufotracker
│   ├── app-service.yaml     # NodePort 8080
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── redis-deployment.yaml
│   └── redis-service.yaml
├── src/
├── .mvn/                    # Maven Wrapper config
├── Dockerfile               # multi-stage (Maven → JRE Alpine)
├── docker-compose.yml
├── mvnw
└── pom.xml
```

---

## 4. Build e Execução Local

### 4.1 Build com Maven

```bash
./mvnw package -DskipTests      # Build sem testes (rápido)
./mvnw test                     # Só testes
./mvnw package                  # Build completo com testes
```

### 4.2 Build e push da imagem Docker

```bash
docker build -t wanderfalcao/ufotracker:latest .
docker push wanderfalcao/ufotracker:latest
```

### 4.3 Subir a stack no Kubernetes

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/
kubectl get all,configmap,secret -n ufology
```

---

## 5. Evidências do AT

### 5.1 Imagem publicada no Docker Hub

- **Repositório**: https://hub.docker.com/r/wanderfalcao/ufotracker
- **Tag em uso**: `wanderfalcao/ufotracker:latest`
- **Referenciada em**: `k8s/app-deployment.yaml` (campo `image:`)

### 5.2 Comandos usados na entrega

```bash
# 1. Build local da imagem (multi-stage Maven → Alpine JRE)
docker build -t wanderfalcao/ufotracker:latest .

# 2. Push para o Docker Hub
docker login
docker push wanderfalcao/ufotracker:latest

# 3. Aplicação da stack no cluster
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/

# 4. Evidência — listar recursos do namespace
kubectl get all,configmap,secret -n ufology
```

### 5.3 Configuração consumida em runtime

| Chave         | Origem                 | Consumo em `app-deployment.yaml` |
|---------------|------------------------|----------------------------------|
| `DB_NAME`     | ConfigMap `app-config` | `configMapKeyRef`                |
| `DB_PASSWORD` | Secret `db-secret`     | `secretKeyRef`                   |
| Host Postgres | Service DNS interno    | `postgres-service:5432`          |
| Host Redis    | Service DNS interno    | `redis-service:6379`             |

---

## 6. Pipelines (GitHub Actions)

### 6.1 Tabela de workflows

| Workflow          | Trigger                               | Função                                                          |
|-------------------|---------------------------------------|-----------------------------------------------------------------|
| `gradle-ci.yml`   | `push` em `main`, `workflow_dispatch` | Build + upload do JAR, testes com Postgres/Redis como services  |
| `run-monitor.yml` | `workflow_run` após CI                | Monitor + deploy em `production`; usa `ACTIONS_STEP_DEBUG=true` |
| `tests.yml`       | `pull_request`                        | Gate de testes antes do merge                                   |
| `hello.yml`       | manual                                | Pipeline introdutório                                           |
| `env-demo.yml`    | manual                                | Demonstração de `vars`                                          |
| `secret-demo.yml` | manual                                | Demonstração de `secrets`                                       |

### 6.2 Variáveis em três níveis

O GitHub Actions permite declarar variáveis de ambiente em três escopos
distintos, cada um com visibilidade diferente:

| Nível    | Como declarar          | Exemplo neste projeto                                  |
|----------|------------------------|--------------------------------------------------------|
| Workflow | `env:` no topo do YAML | `WORKFLOW_VERSION: "1.0"`, `ACTIONS_STEP_DEBUG: true`  |
| Job      | `env:` dentro do job   | `DEPLOY_ENV: ${{ vars.DEPLOY_ENV }}`                   |
| Step     | `env:` dentro do step  | `API_KEY: ${{ secrets.API_KEY }}`                      |

O escopo mais restrito tem precedência. Variáveis sensíveis (como `API_KEY`)
são declaradas apenas no step que efetivamente as utiliza, reduzindo a
superfície de exposição.

### 6.3 Secrets e o papel do GITHUB_TOKEN

O GitHub injeta automaticamente um token de acesso (`GITHUB_TOKEN`) em cada
execução de workflow. Ele não precisa ser criado manualmente em
*Settings → Secrets* — já existe no contexto `secrets` de todo run.

O `GITHUB_TOKEN` autentica ações do workflow com permissão restrita ao
repositório atual: criar comentários em PRs, publicar releases, fazer
deploy em environments, interagir com a API do GitHub. As permissões podem
ser ajustadas no YAML via `permissions:`.

Neste projeto ele é utilizado no job `deploy` do `run-monitor.yml` para
autenticar operações no environment `production`:

```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Por ser um token de curta duração gerado a cada run, o `GITHUB_TOKEN`
é uma prática mais segura do que armazenar um Personal Access Token (PAT)
como secret de longa duração.

### 6.4 Environment `production` com aprovação manual

O job `deploy` em `run-monitor.yml` está vinculado ao environment
`production`, que exige **aprovação manual** antes de executar:

```yaml
deploy:
  needs: monitor
  environment: production
```

Esse mecanismo funciona como um *gate* humano no pipeline: o GitHub pausa
a execução e notifica os revisores configurados. Apenas após a aprovação o
job continua e o deploy é realizado. Isso garante que nenhuma versão chegue
ao ambiente de produção sem revisão consciente, separando claramente o
pipeline contínuo (CI, automático) do pipeline de entrega (CD, controlado).

### 6.5 Monitorização e Debug

**`ACTIONS_STEP_DEBUG=true`**

Quando definida como variável de ambiente no nível do workflow, essa flag
ativa o modo de debug do runner, que imprime no log informações extras de
cada step (variáveis de ambiente, resolução de expressões, estado interno
do runner). É declarada no topo do `run-monitor.yml`:

```yaml
env:
  ACTIONS_STEP_DEBUG: true
```

**Workflow commands**

Os scripts usam comandos especiais que o runner interpreta para enriquecer
os logs e o painel de Actions:

| Comando         | Efeito                                                 |
|-----------------|--------------------------------------------------------|
| `::error::`     | Marca o step como falho e exibe o texto em vermelho    |
| `::warning::`   | Exibe alerta amarelo sem interromper a execução        |
| `::notice::`    | Exibe informação em azul para diagnóstico              |

Exemplo de uso no `run-monitor.yml`:

```bash
if [ -z "$DEPLOY_ENV" ]; then
  echo "::error::Variavel DEPLOY_ENV nao configurada no repositorio"
  exit 1
fi
echo "::notice::Monitoramento iniciado para o ambiente $DEPLOY_ENV"
```

**Job Summaries**

Todos os jobs gravam um resumo estruturado em `$GITHUB_STEP_SUMMARY`,
visível diretamente na aba *Summary* de cada run no GitHub Actions, sem
precisar abrir os logs completos:

```bash
echo "## Build Summary" >> $GITHUB_STEP_SUMMARY
echo "| Campo | Valor |" >> $GITHUB_STEP_SUMMARY
echo "|---|---|" >> $GITHUB_STEP_SUMMARY
echo "| Status | Build OK |" >> $GITHUB_STEP_SUMMARY
```

**Diagnóstico com sugestões de correção**

Além dos comandos de log, os scripts incluem mensagens de orientação que
guiam o operador na resolução de problemas sem precisar consultar
documentação externa:

```bash
echo "::error::PROD_DOMAIN nao configurado no environment production"
echo "Sugestao: acesse Settings > Environments > production > Variables"
```
