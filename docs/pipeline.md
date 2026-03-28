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
Os Quality Gates são as barreiras automáticas que garantem que apenas códigos que atingem os padrões de excelência da **ShopEasy** avancem no pipeline.

* **Testes Obrigatórios:**
    * **Unitários:** Validação de lógica isolada (Job: `tests`).
    * **Integração:** Validação de comunicação entre componentes e serviços (Job: `tests`).
    * **Smoke Tests:** Testes de fumaça pós-deploy via endpoint `/health` (Jobs: `deploy-homolog` e `deploy-prod`).

* **Análises Estáticas (SonarQube):**
    O pipeline utiliza o `sonarqube-scan-action` com o parâmetro `sonar.qualitygate.wait=true`. O pipeline será interrompido se os seguintes índices não forem atingidos:
    * **Manutenibilidade:** Rating B (6% - 10%).
    * **Análise de Bugs:** Rating A (Zero bugs críticos).
    * **Cobertura de Testes:** Superior a **80%**.
    * **Duplicação de Código:** Inferior a **3%**.

* **Segurança (TRIVY):**
    Conforme configurado nos estágios 2 e 4 do pipeline, o **Trivy** realiza varreduras de *filesystem*, segredos expostos e vulnerabilidades na imagem Docker.
    * **Critério de Bloqueio:** O pipeline é interrompido imediatamente (`exit-code: 1`) caso sejam encontradas vulnerabilidades de severidade **HIGH** ou **CRITICAL**.
    É possível obter mais detalhes sobre o Trivy na **ETAPA 9** deste documento.


## 6. Estratégia de Deploy
O modelo de entrega da **ShopEasy** foca em segregação de ambientes e janelas de manutenção controladas.

### 6.1. Ambientes (dev/hml/prd)
O projeto possui 3 ambientes segregados:
* **Desenvolvimento (DEV):** Branch que roda localmente para construção de features.
* **Homologação (HML):** Ambiente de testes para validações do time de **QA**.
* **Produção (PRD):** Ambiente final de entrega de valor.

### 6.2. Tipo de Deploy
* **Estratégia:** **Recreate**. 
* **Funcionamento:** Sempre é gerada uma nova imagem versionada no **GHCR** (utilizando o `${{ github.sha }}`). A imagem antiga é preservada no registro para permitir o processo de **Rollback Manual** em caso de incidentes.

### 6.3. Janela de Publicação
* **Dias:** Terças-feiras e Quintas-feiras.
* **Horário:** 03:00 AM (Horário de Brasília/São Paulo).

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

- **Responsável pela execução:** Líder Técnico ou Líder de DevSecOps. Na ausência de ambos, qualquer membro sênior do time de engenharia com acesso ao repositório.

- **Passos de rollback:**
  1. Acessar o histórico de execuções do pipeline no repositório e localizar a última execução bem-sucedida em produção.
  2. Identificar a versão estável correspondente a essa execução (cada execução é vinculada a uma versão específica do código).
  3. Acionar um novo deploy a partir dessa versão estável - isso pode ser feito reativando a execução anterior bem-sucedida do pipeline ou iniciando uma nova execução a partir do código daquela versão.
  4. Aguardar a conclusão do processo e confirmar que o sistema voltou a responder corretamente.
  5. Registrar no histórico do pipeline a causa da falha e a ação tomada.
  6. Comunicar todos os aprovadores e o time de desenvolvimento sobre o ocorrido e o status de recuperação.

- **Tempo alvo de recuperação:** Meta de 15 minutos a partir da detecção da falha. O tempo real pode variar conforme a disponibilidade da equipe e a complexidade do problema.

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
