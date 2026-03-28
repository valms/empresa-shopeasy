# Documentação de Pipeline CI/CD - ShopEasy

## Metadados do Documento
- Documento:
- Versão:
- Status: Rascunho | Em revisão | Aprovado
- Responsável (owner):
- Aprovador:
- Última atualização:
- Próxima revisão:
- Público-alvo:
- Classificação da informação: Interna | Restrita | Confidencial

## Premissas, Lacunas e Riscos (preenchimento obrigatório)
- Premissas (o que está sendo assumido para elaborar o documento):
- Lacunas de informação (dados ausentes que impactam o detalhamento):
- Riscos identificados (inclua impacto e mitigação sugerida):


## 1. Visão Geral do Pipeline

- **Produto/sistema atendido:** Plataforma ShopEasy - aplicação web composta por front-end e back-end distribuídos em serviços independentes.

- **Objetivo do pipeline:** Garantir a integração e entrega contínua (CI/CD) das aplicações de forma automatizada, segura e padronizada, desde o envio do código até a disponibilidade nos ambientes produtivos. O pipeline visa reduzir erros manuais, aumentar a rastreabilidade das mudanças e assegurar qualidade por meio de validações automatizadas.

- **Ferramentas envolvidas:**

| Ferramenta | Função no pipeline |
|---|---|
| GitHub Actions | Orquestração e execução de todas as etapas do pipeline |
| Docker | Empacotamento da aplicação em imagem para garantir consistência entre ambientes |
| GitHub Container Registry (GHCR) | Armazenamento e versionamento das imagens geradas |
| SonarQube | Análise estática do código-fonte e validação do quality gate |
| Trivy | Varredura de segurança no repositório e na imagem gerada (vulnerabilidades, segredos expostos e misconfigurações) |
| GitHub Environments | Controle de aprovação manual e registro de auditoria por ambiente |
| Framework de testes (a definir) | Execução de testes unitários e de integração - framework a ser definido conforme stack do projeto |

## 2. Estratégia de Versionamento e Branches

- **Modelo de branches adotado:** GitFlow simplificado. ([referência](https://nvie.com/posts/a-successful-git-branching-model/))

| Branch | Finalidade |
|---|---|
| `main` | Código em produção - estável e auditado |
| `develop` | Integração contínua - base para homologação |
| `feature/*` | Desenvolvimento de novas funcionalidades |
| `release/*` | Preparação e ajustes finais de uma versão |
| `hotfix/*` | Correções urgentes aplicadas diretamente sobre `main` |

- **Convenção de versionamento:** Versionamento Semântico ([SemVer](https://semver.org/lang/pt-BR/)), no formato `MAJOR.MINOR.PATCH`:
  - `MAJOR` - mudança incompatível com versões anteriores
  - `MINOR` - nova funcionalidade compatível com versões anteriores
  - `PATCH` - correção de bug sem impacto em funcionalidades existentes

- **Padrão de commits:** Commits Semânticos ([Conventional Commits](https://www.conventionalcommits.org/pt-br/)), no formato `tipo(escopo): descrição`:

| Tipo | Quando usar |
|---|---|
| `feat` | Adição de nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Alterações em documentação |
| `refactor` | Refatoração sem mudança de comportamento |
| `test` | Adição ou correção de testes |
| `chore` | Tarefas de manutenção (dependências, configurações etc.) |
| `hotfix` | Correção urgente em produção |

- **Política de merge:**
  - Todo código deve passar por Pull Request (PR) com ao menos uma aprovação de revisor técnico.
  - A execução bem-sucedida do pipeline (build e testes) é requisito obrigatório para o merge.
  - Commits diretos nas branches `main` e `develop` não são permitidos.
  - Squash merge para branches de `feature` - mantém o histórico limpo.
  - Merge commit para `release` e `hotfix` - preserva o contexto do agrupamento.

## 3. Gatilhos de Execução
| Gatilho | Evento | Branch alvo | Observações |
|---|---|---|---|
|   |   |   |   |

## 4. Estágios do Pipeline
| Estágio | Objetivo | Critério de sucesso | Evidência gerada |
|---|---|---|---|
|   |   |   |   |

## 5. Gates de Qualidade
- Testes obrigatórios:
- Análises estáticas:
- Critérios de bloqueio:

## 6. Estratégia de Deploy
- Ambientes (dev/hml/prd):
- Tipo de deploy (rolling, blue/green, canary):
- Janela de publicação:

## 7. Aprovação e Governança
- Quem aprova cada ambiente:
- Critérios de aprovação:
- Trilha de auditoria:

## 8. Rollback e Recuperação
- Quando acionar rollback:
- Passos de rollback:
- Tempo alvo de recuperação:

## 9. Segurança no Pipeline
- Gestão de segredos:
- Controle de acesso:
- Validações de segurança:

## 10. Métricas e Observabilidade
- Lead time:
- Taxa de falha de deploy:
- Tempo médio de restauração:
- Onde os logs/indicadores são consultados:

## 11. Riscos e Melhorias
- Riscos atuais:
- Dependências críticas:
- Melhorias recomendadas:


## Anexos e Referências
- Exemplo de pipeline (YAML/arquivo de configuração):
- Evidências de execução (logs/screenshots):
- Política de deploy/rollback relacionada:
- Links de PRs/issues relacionados:

## Checklist de Qualidade (pré-entrega)
- [ ] Estágios e gatilhos do pipeline documentados.
- [ ] Gates de qualidade e critérios de bloqueio definidos.
- [ ] Estratégia de deploy e rollback detalhada.
- [ ] Papéis de aprovação e governança descritos.
- [ ] Premissas, lacunas e riscos preenchidos.
