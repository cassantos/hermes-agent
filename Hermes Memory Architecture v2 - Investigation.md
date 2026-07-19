Hermes Memory Architecture v2 — Investigation

Resumo
------
Documento de referência que consolida a investigação arquitetural realizada sobre o crescimento do contexto no Hermes, a causa raiz identificada, princípios aprovados, arquitetura de referência proposta, estratégia de migração incremental, backlog de evolução (Fases 1–4), decisões tomadas e adiadas, e próximos passos. Serve para retomar o trabalho posteriormente sem perda de contexto.

1. Problema original
--------------------
Conversa longas acumulavam contexto vivo (histórico persistido) que, ao ser reenviado ao provedor, causava:
- custos elevados por tokens;
- latência alta;
- erros de rate limit / TPM;
- perda de foco do modelo.

Observação de caso: sessão investigada mostrou ~143k tokens de contexto (estimado), resultado de outputs de ferramentas grandes e blobs de "codex reasoning" armazenados inline no transcript.

2. Causa raiz identificada
--------------------------
O principal fator é arquitetural: o pipeline atual persiste integralmente os outputs de ferramentas como mensagens ativas (active=1) no transcript. O compressor (ContextCompressor) é corretivo — só age quando o threshold é atingido — e, quando acionado, precisa montar o próprio bloco extenso que deve resumir. Essa dependência circular (o compressor precisa do bloco grande para reduzi-lo) pode levar a requisições de compressão que excedem limites do provedor e provocam falhas.

3. Princípios arquiteturais aprovados
-------------------------------------
1. Eliminação da dependência circular: o mecanismo de recuperação não pode depender do recurso que entrou em colapso.
2. Complexidade previsível: custo/complexidade para iniciar compressão deve ser limitado e estável, independente do tamanho da sessão.
3. Custo proporcional ao trecho novo: o crescimento de custo deve responder sobretudo ao trecho recentemente adicionado, não ao histórico completo.
4. Rastreabilidade completa: preservar artefatos brutos para auditoria sem mantê‑los no contexto vivo.
5. Lazy rehydration: recuperação de artefatos completos apenas sob demanda.
6. Persistência preventiva: a política de persistência é primária; o compressor é corretivo.
7. Compatibilidade: manter compatibilidade com o modelo atual de sessões, ferramentas e memória do Hermes.

4. Arquitetura de referência
----------------------------
Visão em camadas e componentes (alto nível):

A. Tool Output Handler
- Recebe outputs das ferramentas, anexa metadados e marca hints.

B. Persistence Manager (política central)
- Decide summary vs artifact, grava artefatos no Artifact Store e persiste apenas summary+pointer no Conversation Memory.

C. Artifact Store
- Repositório indexado de artefatos brutos (deduplicado, metadados, retenção, criptografia).

D. Conversation Memory
- Transcript leve contendo system prompt, head, tail protegidos e summaries/pointers.

E. Preflight Estimator & Selector
- Usa metadados/FTS (ou embeddings) para decidir quais mensagens ler/enviar; controla triggers de compressão.

F. Compression Orchestrator
- Orquestra pré‑prune, chunking/two‑phase summarization, grava summaries no Conversation Memory; opera como mecanismo corretivo.

G. Background Compaction Worker
- Executa compactions progressivas em segundo plano para amortizar carga.

H. Rehydration Service
- Recupera artifacts on‑demand e reintroduz nos turnos quando necessário; caching curto por sessão.

I. Telemetry & Policy Engine
- Métricas tokens_before/after, gates automáticos, alertas, CQ proxies (continuidade e fidelidade).

Fluxo resumido de interação
1. Tool executa → Tool Output Handler envia output + hint para Persistence Manager.  
2. Persistence Manager grava artifact bruto em Artifact Store e persiste summary+pointer em Conversation Memory (preventivo).  
3. No início do turno, Preflight Estimator decide se compress é necessário; se sim, Compression Orchestrator é acionado (usando seletores e pre‑prune).  
4. Request Builder monta payload com summaries/pointers/cauda protegida e envia ao provedor.  
5. Se o usuário pedir, Rehydration Service recupera artifact completo (lazy) e o injeta numa única interação.

5. Estratégia de migração aprovada (Strangler Fig incremental)
------------------------------------------------------------
Objetivo: evoluir sem reescrita, com baixo risco e rollback fácil. Fases principais (resumo):

Fase 0 — Preparação e backups
- Objetivo: telemetry, backups, profile canary, runbooks de rollback.  
- Critério: backups verificados, telemetria ativa.

Fase 1 — Pre‑prune heurístico (implementar primeiro)
- Objetivo: mitigação imediata — truncar/blindar blobs óbvios antes de qualquer chamada de compressão.  
- Critério: /compress manual em canary não excede limites de TPM; redução de tokens por compress ≥ 25% (e sem regressão funcional grave).

Fase 2 — Background progressive compaction
- Objetivo: amortizar compaction no tempo; reduzir picos e preparar summaries prévios.  
- Critério: compactions background estabilizadas e redução de tokens médios.

Fase 3 — Pilot Artifactize (uma ferramenta)
- Objetivo: aplicar two‑tier persistence (Artifact Store + pointer) para uma ferramenta piloto (ex.: read_file).  
- Critério: piloto grava artifact + pointer; rehydrate funcional; tokens med. caem.

Fase 4 — Expandir artifactize para ferramentas críticas
- Objetivo: estender para search_files, skill_view, terminal.  
- Critério: cobertura das ferramentas alvo, redução de tokens em workloads, rehydrate OK.

Notas: fases subsequentes (DB‑selector, chunking, index/embeddings, etc.) planeadas mas fora do escopo imediato.

6. Backlog de evolução (Fases 1 a 4)
----------------------------------
- Fase 1: Pre‑prune heurístico — detectores de blobs grandes, truncation seguro, placeholder; logs e métricas.  
- Fase 2: Background compaction worker (scheduler + incremental summary store).  
- Fase 3: Persistence Manager mínimo + Artifact Store (pilot para read_file).  
- Fase 4: Expandir Artifactization para search_files, skill_view, terminal; retention policies; rehydrate API; access control.  
(Para cada item do backlog detalhar tarefas, critérios de aceite, testes e rollback quando iniciar implementação.)

7. Decisões tomadas
-------------------
- A arquitetura de referência acima foi aprovada.  
- Estratégia de migração incremental (Strangler Fig) aprovada.  
- Implementar Fase 1 (Pre‑prune heurístico) como prioridade operacional.
- Compressor é considerado corretivo; persistência preventiva (Artifact Store + Persistence Manager) é o objetivo de longo prazo.

8. Decisões adiadas
-------------------
- Implementação do Artifactize‑on‑write (two‑tier persistence) foi adiada para fases futuras (3–4).  
- Decisão sobre qual solução de indexação semântica (embeddings) usar foi adiada para análise posterior.  
- Valores numéricos finais para knobs (threshold, protect_last_n, target_ratio) serão definidos apenas após Experimentos operacionais controlados.

9. Próximos passos imediatos (curto prazo)
-----------------------------------------
1. Executar Experimento 0 (validar compress manual em canary) — já aprovado como ação inicial.  
2. Implementar Fase 1 (Pre‑prune heurístico) no pipeline canary (apenas mitigation), validar com antes/depois.  
3. Coletar métricas essenciais e avaliar se o compressor consegue operar sem exceder TPM.  
4. Se Experimento 0 + Fase 1 indicarem que o compressor reduz context quando acionado, iniciar Experimento 1 (threshold battery) em canary.  
5. Manter este documento como registro de decisão e referência para fases posteriores.

Anexos / referências
--------------------
- Relatórios de investigação (logs e queries de state.db) gerados durante a investigação.
- Localização: ~/.hermes/hermes-agent/ (código) e ~/.hermes/state.db (DB).  

Fim do documento.
