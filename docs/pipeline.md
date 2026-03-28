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
- Produto/sistema atendido:
- Objetivo do pipeline:
- Ferramentas envolvidas:

## 2. Estratégia de Versionamento e Branches
- Modelo de branches adotado:
- Convenção de versionamento:
- Política de merge:

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
- Gestão de segredos: Os segredos sensíveis, como o SONAR_TOKEN e o SONAR_HOST_URL, devem ser configurados exclusivamente via GitHub Secrets. Como medida de mitigação contra exposição, recomenda-se o uso de repositórios privados e o escopo de segredos por ambiente. O GITHUB_TOKEN é gerado automaticamente pelo GitHub Actions para autenticação no GHCR, dispensando configuração manual.
- Controle de acesso: O acesso e as promoções para o ambiente de produção são controlados pelo recurso de "environment" do GitHub Actions. É obrigatória a configuração de "Required reviewers" (aprovadores manuais) para garantir que nenhum deploy em produção ocorra sem revisão humana prévia.
- Validações de segurança: O pipeline integra múltiplas camadas de proteção:
    - Análise Estática (SAST): O SonarQube bloqueia o pipeline imediatamente se o Quality Gate não for aprovado, cobrindo bugs e vulnerabilidades estáticas.
    - Varredura do Repositório: O job trivy-repo-scan busca segredos expostos, vulnerabilidades em dependências e misconfigurações em arquivos de infraestrutura (IaC).
    - Varredura de Imagem: Após o build, o Trivy inspeciona a imagem Docker em busca de CVEs no sistema operacional, impedindo o deploy caso existam falhas de nível HIGH ou CRITICAL.

## 10. Métricas e Observabilidade
- Lead time: Embora o valor exato dependa da operação, o pipeline utiliza Docker Buildx com cache para otimizar o tempo de build e reduzir o tempo total de entrega (lead time). A adoção de tags imutáveis ({sha}) facilita o rastreio rápido de qual mudança está em qual estágio.
- Taxa de falha de deploy: O pipeline monitora as falhas através de um step de notificação automática em caso de erro no job de produção, direcionando alertas para o canal #deploys da equipe.
- Tempo médio de restauração: O tempo de recuperação é apoiado por uma estratégia de rastreabilidade total, onde cada imagem publicada está vinculada a um commit exato ({sha}), permitindo a identificação e reversão rápida de mudanças problemáticas. O template prevê a definição de passos de rollback e um tempo alvo de recuperação para guiar o time.
- Onde os logs/indicadores são consultados:
    - Logs de Execução: Consultados diretamente na interface do GitHub Actions.
    - Alertas de Segurança: Centralizados no GitHub Security via arquivos SARIF.
    - Audit Trail: O registro de cada deploy e suas aprovações fica armazenado na trilha de auditoria do ambiente no GitHub.
    - Relatórios Técnicos: Artefatos de varredura (como os do Trivy) são retidos por 30 dias para consulta técnica.

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
