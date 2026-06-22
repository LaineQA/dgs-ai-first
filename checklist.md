## Checklist de revisão de testes

Use este checklist para revisar qualquer teste de integração em menos de 2 minutos.

1. `describe` / `it`
   - [ ] `describe` é específico ao módulo e `it` descreve o comportamento esperado.
2. Arrange / Act / Assert
   - [ ] O teste tem setup, execução e verificação bem separados.
3. Realismo de domínio
   - [ ] A pergunta é um caso logístico realista, não `test` ou `hello`.
4. Mocking
   - [ ] Chamadas HTTP externas são mockadas com `msw`.
   - [ ] `onUnhandledRequest: 'error'` está habilitado.
5. Asserções
   - [ ] O teste verifica comportamento específico.
   - [ ] Não usa apenas `toBeDefined()` ou `toBeTruthy()`.
   - [ ] Verifica `source_document` em endpoints de query.
6. Reutilização de fixtures
   - [ ] Usa fixtures/factories compartilhadas quando disponíveis.
   - [ ] Não duplica dados hardcoded desnecessariamente.
7. Isolamento
   - [ ] O teste não depende de outros testes ou estado global.
8. Cobertura
   - [ ] O cenário está ligado a um critério de verificação ou regra de negócio.

Se todas as caixas estiverem marcadas, o teste está pronto para revisão de QA.
## Checklist de revisão de testes

Use este checklist para revisar qualquer teste de integração em menos de 2 minutos.

1. `describe` / `it`
   - [ ] `describe` é específico ao módulo e `it` descreve o comportamento esperado.
2. Arrange / Act / Assert
   - [ ] O teste tem setup, execução e verificação bem separados.
3. Realismo de domínio
   - [ ] A pergunta é um caso logístico realista, não `test` ou `hello`.
4. Mocking
   - [ ] Chamadas HTTP externas são mockadas com `msw`.
   - [ ] `onUnhandledRequest: 'error'` está habilitado.
5. Asserções
   - [ ] O teste verifica comportamento específico.
   - [ ] Não usa apenas `toBeDefined()` ou `toBeTruthy()`.
   - [ ] Verifica `source_document` em endpoints de query.
6. Reutilização de fixtures
   - [ ] Usa fixtures/factories compartilhadas quando disponíveis.
   - [ ] Não duplica dados hardcoded desnecessariamente.
7. Isolamento
   - [ ] O teste não depende de outros testes ou estado global.
8. Cobertura
   - [ ] O cenário está ligado a um critério de verificação ou regra de negócio.

Se todas as caixas estiverem marcadas, o teste está pronto para revisão de QA.