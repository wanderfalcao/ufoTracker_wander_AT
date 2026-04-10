# Operação Ufology — ufoTracker

[![Hello CI/CD](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/hello.yml?label=Hello%20CI%2FCD)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/hello.yml)
[![Tests](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/tests.yml?label=Tests)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/tests.yml)
[![Maven CI](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/gradle-ci.yml?label=Maven%20CI)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/gradle-ci.yml)
[![Env Demo](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/env-demo.yml?label=Env%20Demo)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/env-demo.yml)
[![Secret Demo](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/secret-demo.yml?label=Secret%20Demo)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/secret-demo.yml)
[![Run Monitor](https://img.shields.io/github/actions/workflow/status/wanderfalcao/ufoTracker_wander_AT/run-monitor.yml?label=Run%20Monitor)](https://github.com/wanderfalcao/ufoTracker_wander_AT/actions/workflows/run-monitor.yml)

Sistema de rastreamento de avistamentos UFO. Stack: Java 21, Spring Boot 3.5.4, PostgreSQL, Redis.

---

## Git na Entrega Contínua

O Git é o elo central do processo de CI/CD. Cada `push` ou abertura de `pull_request` aciona automaticamente os workflows configurados no GitHub Actions, garantindo que alterações sejam validadas antes de chegarem ao ambiente de produção. O histórico de commits serve também como trilha de auditoria de quem entregou o quê e quando.

### Branches no CI/CD

Branches isolam contextos de trabalho. No modelo adotado:
- `ci/setup` — branch de desenvolvimento do pipeline, gatilho dos workflows de CI.
- `main` — branch estável, recebe apenas código validado via PR.

Workflows com `on: push: branches: [main]` só disparam em merges aprovados, separando validação contínua de integração de entrega.

### Tags no CI/CD

Tags marcam pontos imutáveis no histórico, geralmente cortes de release (`v1.0.0`). Workflows com `on: push: tags:` disparam exclusivamente quando uma tag é publicada, permitindo separar o pipeline de CI (contínuo, por push) do pipeline de release (pontual, com promoção para produção e geração de artefatos versionados).

### Runners: GitHub-hosted vs Self-hosted

| Característica        | GitHub-hosted                            | Self-hosted                                                              |
|-----------------------|------------------------------------------|--------------------------------------------------------------------------|
| Provisionamento       | Automático pela plataforma               | Responsabilidade do time                                                 |
| Manutencao            | Zero                                     | Requer atualizacoes e monitoramento                                      |
| Custo                 | Incluso nas cotas do plano               | Infraestrutura e operacao proprias                                       |
| Acesso a rede interna | Nao                                      | Sim                                                                      |
| Caso de uso tipico    | Build, testes, lint em projetos publicos | Deploy em cluster Kubernetes privado, acesso a VPN, ambientes air-gapped |

Runners self-hosted são necessários quando o job de deploy precisa alcançar infraestrutura interna sem exposição pública — como o cluster Kubernetes desta operação em um ambiente corporativo real.

---

## Estrutura do Repositório

```
.
├── .github/workflows/
│   ├── hello.yml
│   ├── tests.yml
│   ├── gradle-ci.yml
│   ├── env-demo.yml
│   └── secret-demo.yml
├── k8s/
│   ├── namespace.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── redis-deployment.yaml
│   ├── redis-service.yaml
│   ├── app-configmap.yaml
│   ├── app-secret.yaml
│   ├── app-deployment.yaml
│   └── app-service.yaml
├── src/
├── Dockerfile
└── pom.xml
```

---

## Aplicar toda a infraestrutura no Kubernetes

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/
kubectl get all -n ufology
```
