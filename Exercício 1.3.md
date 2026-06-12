# Plano de Testes — Pipeline RAG NovaTech
**Projeto:** Assistente de Atendimento com RAG (Azure AI Search + LLM)
**Versão:** 1.0 | **Data:** 2026-06-12
**Responsável geral:** _a definir_

---

> **⚠️ Premissa fundamental:** Testes de IA não são binários (pass/fail). Os critérios de aceitação usam **escalas de qualidade** (scores, thresholds) e avaliação humana amostral. Automação captura regressões; julgamento humano captura qualidade. Um sistema de RAG pode "passar" num teste de retrieval e ainda assim gerar uma resposta inadequada — por isso cada camada é testada separadamente E o fluxo integrado.

---

## Sumário de Categorias

| Categoria | Testes | Status geral |
|-----------|--------|--------------|
| 1. Ingestão | ING-01 a ING-07 | ⬜ Pendente |
| 2. Retrieval | RET-01 a RET-10 | ⬜ Pendente |
| 3. Geração | GER-01 a GER-07 | ⬜ Pendente |
| 4. Contexto | CTX-01 a CTX-06 | ⬜ Pendente |
| 5. Ponta a ponta (E2E) | E2E-01 a E2E-07 | ⬜ Pendente |
| 6. Regressão | REG-01 a REG-04 | ⬜ Pendente |

**Legenda de status:** ⬜ Pendente | 🔄 Em andamento | ✅ Aprovado | ❌ Reprovado | ⏸ Bloqueado

---

## 1. Testes de Ingestão

**Objetivo:** garantir que cada documento foi corretamente extraído do SharePoint, convertido em texto, dividido em chunks e indexado no Azure AI Search — sem perda, corrupção ou duplicação.

| ID | Teste | Como verificar | Critério de aceitação | Responsável | Status |
|----|-------|---------------|----------------------|-------------|--------|
| ING-01 | Extração de texto fiel à fonte | Comparar texto extraído com documento original (diff) | ≤ 2% de caracteres com ruído; zero perda de parágrafos normativos | _ | ⬜ |
| ING-02 | Chunking respeita fronteiras semânticas | Inspecionar que títulos de seção iniciam novos chunks | Nenhum chunk corta parágrafo normativo no meio | _ | ⬜ |
| ING-03 | Metadados preservados em cada chunk | Verificar campos `doc_id`, `versão`, `seção` no índice | 100% dos chunks com metadados de origem preenchidos | _ | ⬜ |
| ING-04 | Documentos conflitantes indexados separadamente | Confirmar que PROC-042-v1 e PROC-042-v2 existem como chunks distintos com versão marcada | Chunks `doc_id=PROC-042-v1` e `doc_id=PROC-042-v2` distintos no índice | _ | ⬜ |
| ING-05 | Contagem de chunks por documento | Contar chunks no Azure AI Search por fonte e comparar com contagem manual | Δ ≤ 1 chunk por documento | _ | ⬜ |
| ING-06 | Re-ingestão não duplica chunks | Rodar pipeline duas vezes; contar chunks no índice | Contagem idêntica antes e depois da segunda execução | _ | ⬜ |
| ING-07 | Atualização de documento atualiza chunks | Substituir POL-001 por versão com novo parágrafo e re-ingerir | Chunk antigo substituído; novo conteúdo pesquisável em < 15 min | _ | ⬜ |

> **🎯 Armadilha NovaTech:** PROC-042-v1 e v2 coexistem no SharePoint sem hierarquia clara. Verificar que **ambos** são indexados com metadados de versão que permitam filtrar qual usar na geração.

---

## 2. Testes de Retrieval

**Objetivo:** dada uma pergunta, os chunks corretos são recuperados e bem rankeados.

**Método:** para cada par (pergunta → chunks esperados), calcular:
- **Recall@5:** quantos chunks esperados aparecem nos 5 primeiros resultados
- **MRR (Mean Reciprocal Rank):** posição do chunk mais relevante

**Critério mínimo global:** Recall@5 ≥ 0.80 no conjunto completo de testes.

| ID | Pergunta de teste | Chunks que DEVEM ser recuperados | Chunks que NÃO devem liderar | Risco principal | Responsável | Status |
|----|-------------------|----------------------------------|------------------------------|-----------------|-------------|--------|
| RET-01 | "Qual o prazo para solicitar devolução?" | POL-001-A, POL-001-B | — | — | _ | ⬜ |
| RET-02 | "Posso devolver carga perigosa?" | POL-001-B | FAQ-03 deve ser secundário | FAQ informal pode ganhar de documento normativo em ranking | _ | ⬜ |
| RET-03 | "Qual o SLA do cliente Gold para incidente crítico?" | SLA-2024-C, SLA-2024-B | — | Confundir SLA geral (2h) com crítico (30min) | _ | ⬜ |
| RET-04 | "Frete para 800kg com destino à região Norte?" | PROC-042v2-A, PROC-042v2-B | PROC-042-B (v1 desatualizada) | Pipeline pode retornar ambas as versões — contradição de multiplicadores | _ | ⬜ |
| RET-05 | "Qual o multiplicador regional para o Sudeste?" | PROC-042v2-B | PROC-042-B | Multiplicador 1.0 (v1) vs 1.1 (v2): resposta errada se v1 rankeada primeiro | _ | ⬜ |
| RET-06 | "Existe tier Platinum?" | SLA-2024-A | FAQ-15 | FAQ-15 é secundário; SLA-2024-A é normativo e deve liderar | _ | ⬜ |
| RET-07 | "Frete para 300kg para Salvador?" | **Nenhum chunk relevante** | PROC-042v2-B (parcialmente relevante) | Falso positivo: frete padrão < 500kg não está documentado — pipeline não deve retornar nada com alta confiança | _ | ⬜ |
| RET-08 | "O que fazer com carga danificada em trânsito?" | FAQ-38 | — | Apenas FAQ cobre isso — gap de documentação formal; resposta deve sinalizar fonte informal | _ | ⬜ |
| RET-09 | "Desconto para cliente com 12 fretes especiais/mês?" | PROC-042v2-D | PROC-042-C | Chunk correto existe só na v2; v1 não tem desconto por volume | _ | ⬜ |
| RET-10 | "Prazo de devolução + frete especial + SLA Gold" (multi-domínio) | POL-001-A, PROC-042v2-A, SLA-2024-B | — | Query complexa pode diluir relevância e trazer chunks genéricos | _ | ⬜ |

---

## 3. Testes de Geração

**Objetivo:** dado o conjunto correto de chunks, o LLM gera uma resposta adequada?

> A geração pode falhar **mesmo com chunks corretos**: o modelo pode ignorar parte do contexto, misturar informações de versões diferentes, inventar complementos ou inverter regras negativas.

**Método de avaliação:** LLM-as-judge (segundo modelo avalia resposta do primeiro) para escala + revisão humana semanal em amostra de 5%.

| ID | Cenário | Método de avaliação | Critério de aceitação | Responsável | Status |
|----|---------|---------------------|----------------------|-------------|--------|
| GER-01 | Resposta fiel ao chunk (sem acréscimos inventados) | Verificar se resposta cita informação ausente nos chunks fornecidos | Zero alucinação detectável em amostra de 20 respostas | _ | ⬜ |
| GER-02 | Versão correta usada quando há conflito | Fornecer PROC-042-v1 e v2 juntos; perguntar multiplicador Norte | Deve citar v2 (1.8) ou sinalizar ambiguidade — nunca citar só v1 (1.6) como fato | _ | ⬜ |
| GER-03 | Declaração de lacuna quando não há cobertura | Perguntar frete < 500kg (sem documento na base) | Resposta declara que a informação não está disponível — não inventa valor | _ | ⬜ |
| GER-04 | Sinalização de fonte informal | Perguntar sobre carga danificada (só FAQ cobre) | Resposta cita FAQ e sinaliza ausência de documento formal | _ | ⬜ |
| GER-05 | Não inversão de regra negativa | "Posso devolver carga perigosa?" com chunk POL-001-B | Resposta diz que **não** é elegível — não confunde exceção com regra | _ | ⬜ |
| GER-06 | Sinalização de ambiguidade entre versões | Pergunta sobre PROC-042 sem filtro de versão | LLM sinaliza que existem duas versões — não mistura multiplicadores silenciosamente | _ | ⬜ |
| GER-07 | Tom adequado para atendimento | Avaliar tom em amostra de 10 respostas | Tom profissional e direto; sem jargão interno excessivo | _ | ⬜ |

---

## 4. Testes de Contexto

**Objetivo:** garantir que o contexto enviado ao LLM está dentro do orçamento de tokens, bem ordenado e resiliente a efeitos estruturais que degradam qualidade.

| ID | Teste | Como verificar | Critério de aceitação | Responsável | Status |
|----|-------|---------------|----------------------|-------------|--------|
| CTX-01 | Orçamento de tokens | Medir tokens totais do prompt (system + chunks + histórico + pergunta) | ≤ 90% do limite do modelo (ex: ≤ 115k tokens para GPT-4o 128k) | _ | ⬜ |
| CTX-02 | Lost in the middle | Testar com 5 chunks onde o mais relevante está na posição 3 (meio) vs posição 1 (topo) | Qualidade da resposta não cai > 15% quando chunk relevante está no meio | _ | ⬜ |
| CTX-03 | Context rot em conversas longas no Teams | Simular 10 turnos de conversa; verificar se turno 10 ainda usa documentos corretos | Respostas corretas nos turnos 8–10 com score ≥ 0.75 | _ | ⬜ |
| CTX-04 | Truncagem segura quando contexto excede limite | Forçar contexto acima do limite e verificar comportamento | Sistema trunca chunks menos relevantes — nunca o system prompt ou a pergunta | _ | ⬜ |
| CTX-05 | Ordenação de chunks por relevância | Verificar que chunk com maior score de similaridade aparece primeiro no contexto | Top-1 chunk (por score do Azure AI Search) aparece na primeira posição do prompt | _ | ⬜ |
| CTX-06 | Deduplicação de chunks redundantes | Verificar se chunks quasi-idênticos são removidos antes de montar contexto | Nenhum par de chunks com similaridade > 0.95 entre si no mesmo prompt | _ | ⬜ |

> **📌 Lost in the middle:** modelos tendem a priorizar informação no início e no final do contexto, ignorando o meio. Colocar o chunk mais relevante sempre no topo mitiga esse efeito — o CTX-02 verifica isso empiricamente e decide se a ordenação atual é adequada.
>
> **📌 Context rot:** em conversas longas no Teams, o histórico cresce e pode consumir tokens que seriam usados pelos chunks relevantes. CTX-03 simula isso explicitamente.

---

## 5. Testes de Ponta a Ponta (E2E)

**Objetivo:** simular o fluxo real — pergunta do atendente → resposta do assistente — e comparar com gabarito pré-aprovado.

**Fórmula de score E2E:**
> Score = 0.4 × (correção factual) + 0.3 × (ausência de alucinação) + 0.2 × (completude) + 0.1 × (tom)
> **Threshold de aprovação: ≥ 0.75**

| ID | Pergunta | Resposta esperada (gabarito resumido) | Pontos de atenção | Responsável | Status |
|----|----------|--------------------------------------|-------------------|-------------|--------|
| E2E-01 | "Cliente quer devolver mercadoria recebida há 5 dias úteis. É possível?" | Sim, dentro do prazo de 7 dias úteis. Processo: Portal do Cliente, CT-e + 3 fotos + motivo. | Não citar exceções de carga perigosa sem contexto | _ | ⬜ |
| E2E-02 | "Carga de 2.000kg para Porto Alegre (Sul). Qual o multiplicador e prazo adicional?" | Multiplicador 1.3 (v2), fator de peso 1.15 (1001-3000kg), prazo +3 dias úteis. Citar versão usada. | Não usar v1 (multiplicador 1.2, fator 1.2, prazo +2 dias) | _ | ⬜ |
| E2E-03 | "Cliente Gold com incidente crítico aberto há 1h sem resposta. O SLA já estourou?" | SLA Gold para incidente crítico: resposta em 30min. Com 1h sem resposta, SLA violado. | Não confundir com SLA geral (2h para chamados gerais) | _ | ⬜ |
| E2E-04 | "Existe desconto automático para frete especial?" | Sim. A partir de 8 fretes/mês: 5% de desconto. Acima de 15: 10%. (PROC-042-v2, seção 4) | Não citar PROC-042-v1 (sem desconto por volume) | _ | ⬜ |
| E2E-05 | "Frete para 200kg para Recife. Quanto custa?" | Não há informação disponível na base para frete padrão (< 500kg). | Não inventar valor; declarar gap explicitamente | _ | ⬜ |
| E2E-06 | "Cliente diz que é Platinum. Qual o SLA dele?" | Não existe tier Platinum. Os tiers são Gold, Silver e Standard. | Não inventar SLAs — isso é alucinação pura | _ | ⬜ |
| E2E-07 | "Carga chegou danificada em trânsito. O que oriento o cliente?" | Registrar em até 48h, com fotos e laudo. Encaminhar para sinistros@novatech.com.br. (Fonte: FAQ informal — sem documento formal) | Citar que fonte é FAQ; não afirmar como política oficial | _ | ⬜ |

---

## 6. Testes de Regressão

**Objetivo:** quando o sistema muda (prompt, documentos, modelo, parâmetros de retrieval), garantir que nada quebra silenciosamente.

| ID | Gatilho | Testes que rodam automaticamente | Como executar | Critério de alerta | Responsável | Status |
|----|---------|----------------------------------|--------------|-------------------|-------------|--------|
| REG-01 | Atualização de documento na base | ING-01 a ING-07 + RET-01 a RET-10 + E2E-01 a E2E-07 | CI/CD: push de novo doc dispara suite completa | Qualquer teste ING reprovado OU score E2E < 0.70 | _ | ⬜ |
| REG-02 | Mudança de prompt do sistema | GER-01 a GER-07 + CTX-01 a CTX-06 + E2E-01 a E2E-07 | Comparação de scores antes/após mudança | Alerta se delta de score > 10% em qualquer categoria | _ | ⬜ |
| REG-03 | Troca de modelo LLM | Suite completa (todas as categorias) | Gate obrigatório: novos modelos não vão para produção sem score ≥ 0.75 em E2E + aprovação manual | Score E2E < 0.75 bloqueia deploy | _ | ⬜ |
| REG-04 | Mudança em parâmetros de retrieval (top-K, threshold) | RET-01 a RET-10 + CTX-01 a CTX-06 | Monitoramento contínuo de Recall@K e MRR | Recall@5 < 0.75 gera alerta imediato | _ | ⬜ |

> **📌 Golden outputs:** manter arquivo `golden_outputs.json` com pares (pergunta → resposta aprovada). A cada execução de regressão, comparar saída atual com golden via embedding similarity. Alerta automático se score cair abaixo de 0.85.

---

## Apêndice: Armadilhas documentais da NovaTech

Estas situações foram identificadas nos documentos e são propositais para testar a robustez do sistema:

| # | Armadilha | Documentos envolvidos | Risco se não tratado |
|---|-----------|----------------------|---------------------|
| 1 | Multiplicadores regionais conflitantes | PROC-042-v1 vs PROC-042-v2 | LLM mistura multiplicadores de versões diferentes na mesma resposta |
| 2 | Fator de peso diferente entre versões | PROC-042-v1 (1.2/1.5) vs v2 (1.15/1.4) | Cálculo de frete incorreto |
| 3 | Prazo adicional diferente | v1: +2 dias vs v2: +3 dias | Cliente recebe prazo errado |
| 4 | FAQ como fonte para informação crítica | FAQ-32, FAQ-38 sem documento formal | Sistema responde com alta confiança usando fonte não validada por Compliance |
| 5 | Tier inexistente (Platinum) | SLA-2024-A | Alucinação pura: LLM inventa SLAs para tier que não existe |
| 6 | Inversão de regra negativa | POL-001-B (cargas perigosas NÃO elegíveis) | LLM inverte e diz que "podem ser devolvidas" |
| 7 | Pergunta sem cobertura na base | Frete padrão < 500kg | LLM inventa valor de frete em vez de declarar a lacuna |

---

*Documento gerado em 2026-06-12 | Pipeline RAG NovaTech v1.0*
