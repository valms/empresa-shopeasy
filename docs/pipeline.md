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
