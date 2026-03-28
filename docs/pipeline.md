# Documentação de Pipeline CI/CD - ShopEasy

## Metadados do Documento
- **Documento:** Documentação de Pipeline CI/CD - ShopEasy
- **Versão:** 1.0
- **Status:** Em revisão
- **Responsável (owner):** Time de Engenharia / DevOps
- **Aprovador:** Líder Técnico / Gerente de Engenharia
- **Última atualização:** 2026-03-28
- **Próxima revisão:** 2026-06-28
- **Público-alvo:** Time de engenharia, time de QA, gestores de entrega
- **Classificação da informação:** Interna

## Premissas, Lacunas e Riscos (preenchimento obrigatório)

- **Premissas:**
  - O repositório está hospedado no GitHub e utiliza GitHub Actions como plataforma de CI/CD.
  - A imagem da aplicação é publicada no GitHub Container Registry (GHCR).
  - O SonarQube está disponível como serviço externo com credenciais configuradas.
  - Os ambientes de homologação e produção estão configurados no GitHub com as respectivas regras de proteção.
  - A branch `develop` corresponde ao ambiente de homologação e `main` ao ambiente de produção.
  - Os comandos de deploy e testes no pipeline são representativos - devem ser substituídos pelos comandos reais conforme a infraestrutura do projeto.

- **Lacunas de informação:**
  - Framework de testes não definido - impacta a configuração dos relatórios de cobertura.
  - Infraestrutura de destino do deploy (Kubernetes, ECS, VMs etc.) não especificada - impede detalhar o mecanismo de publicação e rollback automatizado.
  - Política de freeze de publicação (ex.: datas de alto volume como Black Friday) não formalizada.

- **Riscos identificados:**

| # | Risco | Impacto | Mitigação |
|---|---|---|---|
| R1 | Credenciais expostas em repositório público | Alto | Manter repositório privado; usar GitHub Secrets com escopo por ambiente |
| R2 | Varredura de segurança bloqueando deploy por vulnerabilidade sem correção disponível | Médio | Manter lista de exceções documentadas com data de revisão e justificativa |
| R3 | Tag `latest` da imagem sobrescrita a cada publicação | Alto | Adotar versionamento semântico para produção; restringir `latest` a desenvolvimento |
| R4 | Ausência de verificação de funcionamento real após o deploy | Alto | Substituir os placeholders por verificação real via endpoint `/health` |
| R5 | Deploy em produção sem aprovação manual configurada | Alto | Configurar aprovadores obrigatórios no ambiente de produção antes do primeiro uso |


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

- **Política de merge:**
  - Correções urgentes em produção (branches `hotfix/*`) utilizam o tipo `fix` nos commits, conforme a especificação Conventional Commits.
  - Todo código deve passar por Pull Request (PR) com ao menos uma aprovação de revisor técnico.
  - A execução bem-sucedida do pipeline (build e testes) é requisito obrigatório para o merge.
  - Commits diretos nas branches `main` e `develop` não são permitidos.
  - Squash merge para branches de `feature` - mantém o histórico limpo.
  - Merge commit para `release` e `hotfix` - preserva o contexto do agrupamento.


## 3. Gatilhos de Execução

| Gatilho | Evento | Branch alvo | Observações |
|---|---|---|---|
| Push | Envio de código | `main`, `develop` | Executa o pipeline completo - análise, segurança, testes e deploy no ambiente correspondente à branch. |
| Pull Request | Abertura ou atualização de PR | `main`, `develop` | Executa apenas as etapas de validação (análise estática, segurança e testes). Não realiza deploy nem publica imagem. |

## 4. Estágios do Pipeline

O pipeline é composto por sete etapas executadas em sequência parcialmente paralela. As etapas de análise de código e varredura de segurança do repositório ocorrem ao mesmo tempo e, somente após ambas serem aprovadas, a imagem da aplicação é gerada. A partir da imagem gerada, os testes e a varredura de segurança da imagem também ocorrem em paralelo. O deploy em homologação ou produção só é acionado após todas as etapas anteriores serem concluídas com sucesso.

| # | Etapa | Objetivo | Critério de sucesso | Evidência gerada |
|---|---|---|---|---|
| 1 | Análise Estática (SonarQube) | Verificar qualidade e segurança do código-fonte | quality gate aprovado | Relatório no SonarQube |
| 2 | Varredura do Repositório (Trivy) | Detectar vulnerabilidades, segredos expostos e misconfigurações | Nenhum achado crítico ou alto | Relatório retido por 30 dias |
| 3 | Build da Imagem Docker | Gerar e publicar a imagem da aplicação | Imagem publicada com sucesso | Imagem versionada no GHCR |
| 4 | Varredura da Imagem (Trivy) | Detectar vulnerabilidades na imagem gerada | Nenhum achado crítico ou alto | Relatório retido por 30 dias |
| 5 | Testes Automatizados | Validar funcionamento da aplicação | Todos os testes passam | Relatório de cobertura retido por 30 dias |
| 6 | Deploy - Homologação | Publicar a versão em homologação | Sistema responde corretamente após a publicação | Registro no histórico do pipeline |
| 7 | Deploy - Produção | Publicar a versão em produção após aprovação | Aprovação unânime + sistema responde corretamente | Registro no histórico do pipeline |

## 5. Gates de Qualidade

Gates de qualidade são barreiras automáticas que impedem o avanço do pipeline caso o código não atinja os padrões definidos pela ShopEasy.

- **Testes obrigatórios:**
  - **Unitários:** Validam a lógica de cada componente de forma isolada.
  - **Integração:** Validam a comunicação entre componentes e serviços da aplicação.
  - **Verificação pós-publicação:** Confirmação automática de que o sistema responde corretamente após cada deploy, via endpoint `/health`.

- **Análise estática (SonarQube):**
  O pipeline bloqueia automaticamente caso qualquer um dos critérios abaixo não seja atingido:
  - **Manutenibilidade:** Rating mínimo B (índice de débito técnico entre 6% e 10%).
  - **Bugs:** Rating A - zero bugs críticos identificados.
  - **Cobertura de testes:** Superior a 80% do código.
  - **Duplicação de código:** Inferior a 3%.

- **Segurança (Trivy):**
  O Trivy realiza duas varreduras - no repositório (etapa 2) e na imagem gerada (etapa 4). Mais detalhes na seção 9 deste documento.
  - **Critério de bloqueio:** O pipeline é interrompido caso sejam encontradas vulnerabilidades de severidade **Alta** ou **Crítica**.


## 6. Estratégia de Deploy

O modelo de entrega da ShopEasy foca em segregação de ambientes e janelas de publicação controladas.

- **Ambientes:**

| Ambiente | Sigla | Finalidade |
|---|---|---|
| Desenvolvimento | DEV | Ambiente local para construção e validação inicial de novas funcionalidades |
| Homologação | HML | Ambiente de testes para validação pelo time de QA antes da produção |
| Produção | PRD | Ambiente final acessado pelos usuários da plataforma |

- **Tipo de deploy:**
  - **Estratégia:** Recreate - a cada publicação, uma nova versão da aplicação é gerada e implantada, substituindo a anterior.
  - **Rastreabilidade:** Cada versão publicada é identificada unicamente e vinculada ao código que a originou. Versões anteriores são preservadas no registro de imagens para viabilizar o rollback manual em caso de incidentes.

- **Janela de publicação:**
  - **Dias:** Terças-feiras e Quintas-feiras.
  - **Horário:** 03:00 (horário de Brasília/São Paulo).

## 7. Aprovação e Governança

- **Quem aprova cada ambiente:**

| Ambiente | Tipo de aprovação | Aprovadores obrigatórios |
|---|---|---|
| Homologação | Automática | - |
| Produção | Manual - unanimidade obrigatória | Líder Técnico, Líder de QA, Product Owner, Líder de Segurança/DevSecOps |

- **Critérios de aprovação:**
  - Todas as etapas do pipeline concluídas com sucesso.
  - Funcionalidade validada pelo time de QA em homologação antes de solicitar aprovação para produção.
  - Todos os aprovadores listados devem confirmar individualmente - nenhuma aprovação parcial libera o deploy.
  - Caso qualquer aprovador rejeite, o deploy é bloqueado e o time de desenvolvimento deve corrigir o problema e iniciar um novo ciclo de aprovação.

- **Trilha de auditoria:**
  - Cada execução do pipeline gera um identificador rastreável vinculado à versão do código que originou a execução.
  - Aprovações manuais ficam registradas no histórico do ambiente no repositório (quem aprovou e quando).
  - Relatórios de segurança (Trivy) e cobertura de testes retidos por 30 dias por execução.
  - Alertas de segurança ficam disponíveis no repositório para consulta e acompanhamento.

## 8. Rollback e Recuperação

- **Quando acionar rollback:**
  - Sistema não responde ou retorna erro logo após a publicação de uma nova versão.
  - Aumento anormal de erros detectado por monitoramento após a publicação.
  - Alerta crítico gerado durante ou após a publicação da nova versão.
  - Aprovadores identificam comportamento inesperado durante a validação da nova versão.

- **Responsável pela execução:** Líder Técnico ou Líder de Segurança/DevSecOps. Na ausência de ambos, qualquer membro sênior do time de engenharia com acesso ao repositório.

- **Passos de rollback:**
  1. Acessar o histórico de execuções do pipeline no repositório e localizar a última execução bem-sucedida em produção.
  2. Identificar a versão estável correspondente a essa execução (cada execução é vinculada a uma versão específica do código).
  3. Acionar um novo deploy a partir dessa versão estável - isso pode ser feito reativando a execução anterior bem-sucedida do pipeline ou iniciando uma nova execução a partir do código daquela versão.
  4. Aguardar a conclusão do processo e confirmar que o sistema voltou a responder corretamente.
  5. Registrar no histórico do pipeline a causa da falha e a ação tomada.
  6. Comunicar todos os aprovadores e o time de desenvolvimento sobre o ocorrido e o status de recuperação.

- **Tempo alvo de recuperação:** Meta de 15 minutos a partir da detecção da falha. O tempo real pode variar conforme a disponibilidade da equipe e a complexidade do problema.

## 9. Segurança no Pipeline

- **Gestão de segredos:**
  - Credenciais sensíveis (tokens de autenticação e URLs de serviços) devem ser configuradas exclusivamente via GitHub Secrets, nunca expostas diretamente no código ou nos arquivos de configuração.
  - O token de acesso ao registro de imagens é gerado automaticamente pelo GitHub Actions, dispensando configuração manual.
  - Recomenda-se manter o repositório privado e restringir o escopo de cada credencial ao ambiente onde ela é necessária.

- **Controle de acesso:**
  - O acesso ao ambiente de produção é controlado pelo mecanismo de ambientes do GitHub Actions.
  - Nenhum deploy em produção pode ocorrer sem aprovação humana prévia - os aprovadores obrigatórios estão definidos na seção 7 deste documento.
  - Commits diretos nas branches `main` e `develop` são bloqueados por regras de proteção de branch.

- **Validações de segurança:**
  O pipeline aplica três camadas de verificação de segurança:
  - **Análise de código:** O SonarQube verifica o código-fonte em busca de bugs, vulnerabilidades e problemas de qualidade antes do build.
  - **Varredura do repositório:** O Trivy inspeciona as dependências do projeto, segredos expostos no código e possíveis misconfigurações em arquivos de infraestrutura.
  - **Varredura da imagem:** Após o build, o Trivy analisa a imagem gerada em busca de vulnerabilidades conhecidas no sistema operacional e nos pacotes instalados. O pipeline é bloqueado caso sejam encontradas falhas de severidade **Alta** ou **Crítica**.

## 10. Métricas e Observabilidade

- **Lead time** (tempo entre o envio do código e a disponibilidade em produção):
  - Meta: até 30 minutos em condições normais.
  - O pipeline utiliza cache de build para reduzir o tempo de geração da imagem e otimizar o tempo total de entrega.

- **Taxa de falha de deploy:**
  - Meta: inferior a 5% das publicações em produção.
  - Em caso de falha, o pipeline notifica automaticamente o canal #deploys da equipe para ação imediata.

- **Tempo médio de restauração** (tempo entre a detecção de uma falha e a recuperação do ambiente):
  - Meta: até 15 minutos (ver seção 8 deste documento).
  - Cada versão publicada é rastreável, o que permite identificar e reverter rapidamente uma mudança problemática.

- **Onde consultar logs e indicadores:**
  - **Histórico de execuções do pipeline:** Disponível no repositório, com registro de cada etapa executada e seu resultado.
  - **Alertas de segurança:** Disponíveis no repositório para consulta e acompanhamento.
  - **Trilha de auditoria:** Registro de cada deploy e suas aprovações disponível no histórico do ambiente no repositório.
  - **Relatórios de segurança e cobertura:** Retidos por 30 dias por execução, vinculados ao histórico do pipeline.

## 11. Riscos e Melhorias

- **Riscos atuais:**
  Os riscos identificados estão detalhados na tabela da seção **Premissas, Lacunas e Riscos** (R1–R5) no início deste documento, com impacto e mitigação para cada um.

- **Dependências críticas:**

| Dependência | Impacto em caso de indisponibilidade |
|---|---|
| GitHub Actions | Pipeline não executado - nenhuma entrega possível |
| SonarQube | Pipeline bloqueado na primeira etapa - nenhum build realizado |
| GitHub Container Registry (GHCR) | Imagem não publicada - deploys impossibilitados |
| Serviço de testes de integração | Etapa de testes falha - deploy bloqueado |

- **Melhorias recomendadas:**

| Prioridade | Melhoria | Justificativa |
|---|---|---|
| Alta | Configurar aprovadores obrigatórios no ambiente de produção | Evitar deploy acidental sem revisão humana |
| Alta | Substituir placeholders de deploy e testes pelos comandos reais | Pipeline funcional em ambiente real |
| Alta | Implementar verificação de funcionamento real via endpoint `/health` | Detectar falhas silenciosas após a publicação |
| Alta | Adotar versionamento semântico nas tags de imagem | Rastreabilidade clara de qual versão está em cada ambiente |
| Média | Adicionar teste de segurança em tempo de execução (DAST) após homologação | Ampliar cobertura de segurança além da análise estática |
| Média | Gerar inventário de componentes da imagem (SBOM) | Rastreabilidade de dependências para auditoria e conformidade |
| Média | Usar conta de serviço dedicada para publicação no GHCR | Reduzir risco de vazamento de credencial pessoal |
| Baixa | Separar os executores de build e deploy | Reduzir superfície de ataque no ambiente de publicação |
| Baixa | Formalizar lista de exceções de segurança com revisão mensal | Evitar bloqueios por falso-positivo sem abrir mão da rastreabilidade |


## Anexos
- Exemplo de pipeline (YAML/arquivo de configuração):
- Evidências de execução (logs/screenshots):
- Política de deploy/rollback relacionada:
- Links de PRs/issues relacionados:

## Referências
- Repositório de Imagens: [GitHub Container Registry (GHCR)](https://github.com/features/packages)

- Ferramenta de Segurança: [Trivy Documentation](https://aquasecurity.github.io/trivy/)

- Qualidade de Código: [SonarQube Guides](https://docs.sonarqube.org/)

## Checklist de Qualidade (pré-entrega)
- [X] Estágios e gatilhos do pipeline documentados.(Seções 3 e 4)
- [X] Gates de qualidade e critérios de bloqueio definidos. (Seção 5)
- [X] Estratégia de deploy e rollback detalhada. (Seções 6 e 8)
- [X] Papéis de aprovação e governança descritos. (Seção 7)
- [X] Premissas, lacunas e riscos preenchidos. (Seções iniciais e 11)
