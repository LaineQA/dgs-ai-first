# SKILL: create-integration-test

## Propósito
Esta skill define como gerar testes de integração para endpoints e serviços do NovaTech Assistant. Ela serve para criação de arquivos de teste que exercitam o módulo real sob teste enquanto mockam dependências HTTP externas.

## Quando usar
Use esta skill quando for gerar um teste de integração para:
- um endpoint Azure Function
- um endpoint de query que chama Azure AI Search / Azure OpenAI
- qualquer serviço que deva ser testado como módulo com mocks de HTTP externo

Frase-ativação:
- “crie um teste de integração para o endpoint de query”
- “gere um teste de integração Vitest com MSW e fixtures”
- “escreva um teste de integração para o handler da Azure Function”

## Template
Use esta estrutura em todo teste de integração gerado:

```ts
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { handler } from '@/src/functions/query/handler';
import { buildSearchResultList } from '@/tests/fixtures/factories/search-result.factory';
import { ragQueries } from '@/tests/fixtures/queries';

const server = setupServer(
  http.post('https://*.search.windows.net/*', () =>
    HttpResponse.json({ value: buildSearchResultList(3) })
  ),
  http.post('https://*.openai.azure.com/*', () =>
    HttpResponse.json({ choices: [{ message: { content: 'Resposta simulada' } }] })
  )
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('{{ModuleName}} integration', () => {
  it('should {{expected behavior}} when {{condition}}', async () => {
    // Arrange
    const request = { body: JSON.stringify({ question: '{{pergunta realista}}' }) };
    server.use(/* handler específico do cenário */);

    // Act
    const result = await handler(request as any);

    // Assert
    expect(result.status).toBe(200);
    expect(result.body.source_document).toBeDefined();
    expect(result.body.answer).toContain('{{fragmento esperado}}');
  });
});
```

## Regras
- Sempre use `describe` e `it` com texto descritivo em inglês.
- Sempre inclua seções claras de `Arrange / Act / Assert`.
- Sempre use `msw` para mockar chamadas HTTP externas.
- Sempre use fixtures ou factories compartilhadas em fixtures.
- O teste não deve chamar serviços externos reais.
- A asserção deve ser comportamental e específica.
- Use `onUnhandledRequest: 'error'` para falhar em chamadas HTTP não mockadas.
- Evite hardcode no corpo da resposta quando fixtures já existem.
- Prefira dados realistas do domínio logístico, não placeholders genéricos.

## Exemplo DO

```ts
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { handler } from '@/src/functions/query/handler';
import { buildSearchResultList } from '@/tests/fixtures/factories/search-result.factory';
import { ragQueries } from '@/tests/fixtures/queries';

const server = setupServer(
  http.post('https://*.search.windows.net/*', () =>
    HttpResponse.json({ value: buildSearchResultList(2, { documentId: 'POL-001' }) })
  ),
  http.post('https://*.openai.azure.com/*', () =>
    HttpResponse.json({ choices: [{ message: { content: 'O prazo de devolução é de até 7 dias úteis. Fonte: POL-001-A' } }] })
  )
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('query endpoint integration', () => {
  it('should return a sourced devolução answer when the question is about return deadline', async () => {
    // Arrange
    const request = {
      body: JSON.stringify({ question: ragQueries.devolucao.prazoGeral })
    };

    // Act
    const result = await handler(request as any);

    // Assert
    expect(result.status).toBe(200);
    expect(result.body.source_document).toContain('POL-001-A');
    expect(result.body.answer).toContain('7 dias úteis');
  });
});
```

## Exemplo DON'T

```ts
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

Por que é ruim:
- não usa `describe`
- não separa arrange/act/assert
- usa pergunta genérica
- não checa `source_document`
- não mocka HTTP externo
- asserção vaga

## Anti-padrões
- Teste que usa Azure Search/OpenAI reais.
- Usar apenas `expect(result).toBeDefined()` ou `toBeTruthy()`.
- Hardcode de dados inline quando fixtures existem.
- Teste que verifica muitos comportamentos em um único caso.
- Omitir `afterEach(() => server.resetHandlers())`.
- Depender de ordem de execução entre testes.
- Pergunta genérica como `test` ou `hello`.
- Esquecer a asserção de `source_document`.
- Não configurar `onUnhandledRequest: 'error'`.

## Dependências
Leia estas skills antes de usar `create-integration-test`:
- `typescript-conventions`
- `error-handling`
- `project-structure`
- `testing-patterns`
- `azure-functions-endpoint`
- `azure-ai-search-integration`
```


