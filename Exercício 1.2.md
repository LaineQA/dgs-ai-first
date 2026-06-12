Apliquei a rubrica usando o **Anexo A como fonte de verdade** e também conferi os chunks/referências do **Anexo B** para validar cobertura, armadilhas e riscos de alucinação. O template reutilizável em Excel foi gerado aqui:

[Baixar template de avaliação QA — NovaTech](blob:https://outlook.office.com/f5d6db3e-bd3d-439d-873e-a822bc8abdac)

***

## 1. Avaliação manual — feita antes da rubrica

| # | Avaliação manual     | Justificativa resumida                                                                                                                                                                                                                                                       |
| - | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | Parcialmente correta | O prazo geral é de até **7 dias úteis após o recebimento**, mas a resposta omite outras exceções relevantes e o encaminhamento para **Gestão de Riscos — ramal 4500** nos casos não elegíveis ao processo padrão. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md) |
| 2 | Parcialmente correta | Para 600kg em Manaus/Norte, o multiplicador v2 é **1,8** e o fator de peso para 500kg–1.000kg é **1,0**, mas faltou explicar a fórmula completa e a regra de transição entre PROC-042 v1 e v2. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md)                    |
| 3 | Incorreta            | O tier**“Platinum” não existe**; a NovaTech possui apenas **Gold, Silver e Standard**. A resposta alucinou o tier e os valores de SLA. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md)                                                                            |
| 4 | Incorreta            | Cargas perigosas classes 1 a 6 da ANTT **não são elegíveis para devolução pelo processo padrão**; devem ser tratadas pela Gestão de Riscos. A resposta inverteu a regra. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md)                                          |
| 5 | Parcialmente correta | O multiplicador v2 do Sudeste é **1,1**, mas faltou mencionar que a v1 traz **1,0** e que chamados anteriores a **01/12/2023** podem seguir a versão anterior. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md)                                                    |

***

## 2. Rubrica de avaliação — escala 1 a 3

### Dimensão 1 — Precisão factual

| Nota | Critério                                                                                                                                                      |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3    | Todos os fatos centrais estão corretos e consistentes com a fonte de verdade; não há alucinação, inversão de regra ou mistura indevida de versões.            |
| 2    | A resposta acerta o ponto principal, mas omite ressalva relevante, condição de aplicação, cálculo/elemento necessário ou contexto que pode alterar a decisão. |
| 1    | A resposta contém erro factual material, alucina informação, contradiz regra explícita ou leva o usuário a uma ação incorreta.                                |

### Dimensão 2 — Citação e uso da fonte

| Nota | Critério                                                                                                                                              |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3    | Cita fonte/seção correta e usa fonte adequada, priorizando documento normativo/oficial quando existir; a evidência sustenta integralmente a resposta. |
| 2    | Cita fonte relacionada e majoritariamente correta, mas incompleta, genérica ou sem apontar seção/chunk essencial; há evidência parcial.               |
| 1    | Cita fonte errada, inexistente, insuficiente para a afirmação, ou a resposta contradiz a própria fonte citada.                                        |

### Dimensão 3 — Aderência aos guardrails

| Nota | Critério                                                                                                                                           |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3    | Respeita os limites da base, explicita incerteza/condições quando necessário, não inventa dados e orienta escalonamento correto em casos críticos. |
| 2    | Em geral respeita os limites, mas deveria pedir dado adicional, sinalizar transição/contradição documental ou incluir orientação de escalonamento. |
| 1    | Afirma com confiança informação não suportada, ignora exceção crítica, omite restrição de segurança/compliance ou recomenda ação proibida.         |

### Dimensão 4 — Completude e utilidade

| Nota | Critério                                                                                                                                              |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| 3    | Responde diretamente à pergunta e inclui todos os elementos necessários para uso operacional: condições, exceções, próximos passos e dados faltantes. |
| 2    | Responde parcialmente; é útil, mas falta detalhe operacional importante, ressalva, fórmula completa, versão aplicável ou próximo passo.               |
| 1    | Resposta incompleta ou enganosa; não permite decisão segura pelo usuário ou omite informação essencial de alto impacto.                               |

### Regra de decisão sugerida

|   Pontuação total | Classificação                                                            | Decisão QA               |
| ----------------: | ------------------------------------------------------------------------ | ------------------------ |
|             10–12 | Boa o suficiente                                                         | Aprovar                  |
|               7–9 | Parcial                                                                  | Revisar antes de aprovar |
|               4–6 | Incorreta                                                                | Reprovar                 |
| Regra de bloqueio | Se houver alucinação, inversão de regra crítica ou recomendação insegura | Reprovar automaticamente |

***

## 3. Template reutilizável do Cowork / QA

O arquivo Excel contém as seguintes abas:

1. **Avaliação manual** — registro da avaliação feita antes da rubrica.
2. **Rubrica 1-3** — critérios objetivos por dimensão.
3. **Guia de decisão** — faixas de pontuação e regras de bloqueio.
4. **Template QA reutilizável** — planilha em branco para qualquer lote futuro.
5. **Pontuações aplicadas** — aplicação da rubrica às 5 respostas simuladas.

Arquivo: [template\_avaliacao\_qa\_respostas\_novatech.xlsx](blob:https://outlook.office.com/f5d6db3e-bd3d-439d-873e-a822bc8abdac)

***

## 4. Pontuações aplicadas às 5 respostas

| # | Precisão factual | Fonte | Guardrails | Completude | Total / 12 | Classificação        | Decisão  |
| - | ---------------: | ----: | ---------: | ---------: | ---------: | -------------------- | -------- |
| 1 |                2 |     2 |          2 |          2 |          8 | Parcialmente correta | Revisar  |
| 2 |                2 |     2 |          2 |          2 |          8 | Parcialmente correta | Revisar  |
| 3 |                1 |     1 |          1 |          1 |          4 | Incorreta            | Reprovar |
| 4 |                1 |     1 |          1 |          1 |          4 | Incorreta            | Reprovar |
| 5 |                2 |     2 |          2 |          2 |          8 | Parcialmente correta | Revisar  |

### Observações críticas

* A **resposta 3** deve ser reprovada porque inventa um tier inexistente. A documentação SLA-2024 afirma que só existem **Gold, Silver e Standard**, e o Anexo B também marca “Platinum” como armadilha de alucinação. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md), [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-b-chunks-referencia-rag.md)
* A **resposta 4** deve ser reprovada porque contradiz a POL-001: cargas perigosas classes 1 a 6 **não são elegíveis pelo processo padrão** e exigem tratamento pela Gestão de Riscos. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md), [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-b-chunks-referencia-rag.md)
* As respostas 1, 2 e 5 não são totalmente erradas, mas ficam como **parciais** por omitirem condições importantes, como exceções de devolução, fórmula completa de frete, fator de peso e regra de transição entre versões. [\[uniprimebr...epoint.com\]](https://uniprimebr-my.sharepoint.com/personal/db1_irocha_sisprimedobrasil_com_br/Documents/Arquivos%20de%20Microsoft%20Copilot%20Chat/anexo-a-documentacao-simulada-novatech.md)
