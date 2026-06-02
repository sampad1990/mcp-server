Hey Copilot, add a pre-check node at the start of adjustment_retirement_flow.py in Conflow.  

After run_agent routes here (from NLU match), before fetching account details:  
- Call LLM: "User said: '{user_message}'. Current retirement age in state: {state.retirement_age or 'none'}. Do we need to ask for age? Return JSON: {'skip_ask': true/false, 'extracted_age': 40 or null}"  
- If skip_ask=true → skip the hard-coded "what age?" step, set state.retirement_age = extracted_age, jump to calculation step.  
- If false → run normal flow (ask age).  
- Keep everything else—fetch account, etc.—unchanged.  
- Use existing state for history/context.  






<img width="437" height="269" alt="image" src="https://github.com/user-attachments/assets/301c9de1-d05d-4184-832a-a4915db1838c" />


Option 2​
Use Intent & Orchestration Agent​

Yes​

Guardrails already implemented​

Reuse existing architectural flow​

Single deployable service (ECS)​

ECS service bypass requires an internal code change​

Tightly coupled; requires coordination with JPMC orchestration team​

No backend needed. API already integrated. JPMC makes the code change​

Medium​

Long-term, depending on whether orchestration team opens access​

---------------------------------------

Option 3​
Use Intent, Bypass Orchestration Agent​

No​

Separation of responsibilities​

Guardrails at intent agent layer​

3-tier model: intent  orchestrator  A24 supervisor​

Requires JPMC intent agent code change to allow bypass to our performance agent​

Governance questions across two routing layers​

Goes through intent, bypasses orchestration. Owns supervisor agent maintained​

Medium-High​

Recommended Option​

Best governance, decouple campaign builder from tickets; SLM + RAG for smarter orchestration​


