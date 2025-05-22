# Active Tasks

This file tracks active, in-progress tasks.

---

## Task ID: 8 (Parent: Implement PPO Agent with Integrated XGBoost Validator)

### Subtask ID: 8.2 - Integrate validator into PPO signal workflow

**Status:** Concluída após revisão (Manual Update)

**Notes:**

- Revisão de código concluída em DD/MM/AAAA (substituir pela data atual).
- A integração do XGBoostValidator no PPOAgent (`__init__`, `collect_rollouts`, `realtime_inference`) está funcional.
- Ação é neutralizada se o validador invalidar o sinal PPO.
- Experiência no replay buffer é tratada corretamente para o aprendizado do PPO.
- `TradingEnvironment.get_current_raw_data_window()` e `RealtimeDataPipeline.get_raw_data_history()` estão implementados e robustos.
- Testes unitários para `get_raw_data_history` estão passando.
- **Ponto de Atenção Futuro:** A lógica de extração do `ppo_signal_value` de `action_np` no `PPOAgent` pode precisar de refinamento dependendo da definição do espaço de ação e da interface do `validate_signal`.

**Blockers/Issues:**

- Unable to update the status of subtask "8.2" using Taskmaster MCP tool (`mcp_taskmaster-ai_set_task_status`) or CLI (`task-master set-status`). Both methods report "Subtask 2 not found in parent task 8". This issue needs further investigation or manual update of `tasks.json`.

**Next Steps (for this subtask):**

- Considerada concluída.

---

### Subtask ID: 8.5 - Create validation metrics dashboard

**Status:** Implementação Concluída - Pendente Revisão (Manual Update)

**Detalhes da Implementação (Subtask 8.5):**

- Arquivo do dashboard: `src/monitoring/validator_dashboard.py`.
- Utiliza Dash e Plotly para visualização.
- **Parsing de Logs Aprimorado:**
  - `parse_log_line` agora extrai timestamps com precisão de microssegundos do `event_id`.
  - Tenta converter `Reason_Not_Called` de string para dicionário Python usando `ast.literal_eval`.
- **Métricas Exibidas:**
  1.  **Contagem de Status de Validação:** Gráfico de barras mostrando a distribuição de todos os status de validação (`Aceito/Confirmado`, `Rejeitado`, `Rejeitado (PPO Overridden to Hold)`, `Erro Validação`, `Não Chamado`).
  2.  **Distribuição de Confiança:** Histograma mostrando a distribuição dos scores de confiança para sinais que foram efetivamente validados (não rejeitados, sem erro, e onde o validador foi chamado).
  3.  **Tabela de Eventos Recentes:** Exibe os 20 eventos de validação mais recentes com detalhes relevantes.
  4.  **Série Temporal de Sinais PPO vs. Validados:** Gráfico de linha mostrando `PPO_Signal` e `Validated_Signal` ao longo do tempo para eventos onde o validador foi chamado e os sinais são numéricos.
- **Lógica de Status de Validação Melhorada (`get_validation_status`):**
  - Trata explicitamente `Validator_Called=False` (status: `Não Chamado`).
  - Trata explicitamente `Validator_Error=True` ou falhas de parsing de `Validated_Signal`/`Confidence` (status: `Erro Validação`).
- **Atualização Dinâmica:** O dashboard atualiza automaticamente a cada 30 segundos.
- **Arquivo de Log de Exemplo:** Cria um `trading_agent.log` de exemplo se não existir, para facilitar testes locais do dashboard.

**Próximos Passos (para 8.5):**

- Executar o dashboard localmente para verificar a visualização e o parsing dos logs reais gerados pelo `PPOAgent`.
- Realizar ajustes finos no layout ou nas queries de dados conforme necessário após a visualização.

---
