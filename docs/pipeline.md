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
