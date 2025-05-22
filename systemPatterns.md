# System Patterns for TimesTrader

## General Trading Loop Overview

The following outlines the general execution loop of the TimesTrader system:

```
while True:
    # 1. Wait for a new 5-minute candle
    #    - Data is sourced from a data API (e.g., Polygon, TwelveData, NinjaTrader).
    candle = get_new_candle()

    # 2. Update the time series
    #    - The new candle is appended to the historical series.
    series.append(candle)

    # 3. Extract features using TimesNet (inference only)
    #    - The TimesNet model processes the last N candles to generate the current state.
    state_t = timesnet_model.infer(series[-N:])

    # 4. Each PPO agent makes a decision based on the current features
    #    - Agents for Buy, Sell, Stop Loss (SL), Trailing Stop (TS), Break Even (BE) decide their actions.
    actions = {}
    for agent_name, agent in agents.items():
        actions[agent_name] = agent.decide(state_t)

    # 5. XGBoost Validator receives context and PPO agent actions for validation
    #    - It assembles an input from the current state and PPO actions.
    #    - Predicts the probability of success for the combined proposed action.
    xgb_input = assemble_input_for_xgboost(state_t, actions)
    success_prob = xgboost_model.predict_proba(xgb_input)

    # 6. If the success probability is high, send the order to the broker
    #    - A predefined threshold (e.g., 0.8) is used to gate execution.
    #    - If approved, the order is sent to NinjaTrader 8 (or other configured broker).
    if success_prob >= threshold:
        order_info = execute_order(actions, broker="NinjaTrader8")
        log_trade(order_info)
        trade_executed = True
    else:
        log_rejected_decision(state_t, actions, success_prob)
        trade_executed = False

    # 7. If a trade was executed, wait for its closure and compute the reward
    #    - The system monitors the executed trade until it closes (e.g., hits SL, TP, or is manually closed).
    #    - The result (profit/loss) is used to compute a reward signal.
    if trade_executed:
        trade_result = wait_for_trade_close(order_info)
        reward = compute_reward(trade_result)

        # 8. Collect the new state (state_t+1) after the trade closes
        #    - A new candle is fetched and appended to the series.
        #    - TimesNet infers the next state.
        next_candle = get_new_candle()
        series.append(next_candle)
        state_t1 = timesnet_model.infer(series[-N:])

        # 9. Store the experience in the Replay Buffer for future learning
        #    - The transition (state, action, reward, next_state, done) is stored.
        buffer.add((state_t, actions, reward, state_t1, True))  # done = True indicates the end of an episode/trade

    # 10. Periodically, retrain PPO agents using experiences from the buffer
    #     - If the buffer has enough samples and it's time to retrain (e.g., based on a schedule or buffer size).
    #     - Each agent is trained on a batch of relevant experiences.
    if buffer.ready() and time_to_retrain():
        for agent_name, agent in agents.items():
            agent.train(buffer.sample_batch(agent_name))
```
