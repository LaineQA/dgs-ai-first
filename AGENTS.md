# AGENTS.md — NovaTech Assistant

> Constitution do projeto. Todo agente de IA (Copilot, Claude Code) lê este arquivo antes de gerar qualquer artefato.
> As seções abaixo são preenchidas por papéis diferentes nos exercícios do Cenário 2.

## Project Overview
<!-- TODO (Tech Lead — Ex. 2.1) -->

## Tech Stack & Architecture
<!-- TODO (Tech Lead — Ex. 2.1): inclui regras de gerenciamento de contexto da ADR-0002 -->

## Coding Standards (Tech Lead)
<!-- TODO (Tech Lead — Ex. 2.1) -->

## Product Rules & Guardrails (Product Specialist)
<!-- TODO (Product Specialist — Ex. 2.3) -->

## Testing Standards (QA)

### Stack e execução

| Ferramenta | Uso |
|------------|-----|
| **Vitest** | Testes unitários e de integração |
| **MSW** (Mock Service Worker) | Mock de APIs HTTP externas (Azure AI Search, Azure OpenAI, etc.) |
| **GitHub Actions** (`/.github/workflows/ci.yml`) | Execução automática de testes no CI |
| **Coverage mínimo** | **80% de linhas** — configurado em `vitest.config.ts`; PRs que reduzirem a cobertura abaixo desse limite devem ser rejeitados |

### Organização (`/tests/`)

| Pasta | Escopo |
|-------|--------|
| `unit/` | Uma unidade isolada; **nenhuma** chamada externa (tudo mockado) |
| `integration/` | Integração entre módulos internos; APIs externas mockadas via MSW |
| `e2e/` | Fluxo completo — usar com parcimônia (consome tokens e recursos) |
| `fixtures/` | Dados compartilhados: chunks, queries e respostas esperadas para RAG |

Convenção de arquivos: espelhar o caminho de `src/` (ex.: `src/functions/query/handler.ts` → `tests/unit/functions/query/handler.test.ts`).

### Nomenclatura de testes

Usar **`describe`** para agrupar por unidade ou cenário e **`it`** (ou `test`) com **frases descritivas em inglês** que leiam como especificação de comportamento:

```typescript
describe('query handler', () => {
  describe('when question is valid', () => {
    it('returns 200 with answer and source chunks', async () => { /* ... */ });
    it('includes chunk IDs from Azure AI Search in the response', async () => { /* ... */ });
  });

  describe('when question is missing', () => {
    it('returns 400 with validation error details', async () => { /* ... */ });
  });
});
```

Regras de nomenclatura:

- **`describe`**: nome do módulo ou condição (`'prompt-builder'`, `'when retrieval returns no chunks'`).
- **`it`**: verbo no presente + resultado esperado (`'returns empty sources when search score is below threshold'`).
- Evitar nomes genéricos como `'works'`, `'handles input'`, `'query endpoint works'`.
- Um `it` = **um** comportamento verificável. Se o nome precisar de "and", dividir em testes separados.

### O que todo teste DEVE ter

1. **Estrutura Arrange / Act / Assert** — separar visualmente (comentários ou linha em branco) a preparação, a execução e a verificação.
2. **Assertions específicas** — validar valores concretos (status HTTP, campos da resposta, mensagens de erro, IDs de chunks), não apenas existência.
3. **Determinismo** — mesmo input produz o mesmo resultado em qualquer ordem de execução.
4. **Isolamento** — sem dependência de estado global, ordem de outros testes ou relógio real (usar `vi.useFakeTimers()` quando necessário).
5. **Um motivo claro de falha** — se o teste quebrar, a mensagem de assertion deve indicar o que foi esperado vs. obtido.

Exemplo **correto** (contrasta com o anti-padrão abaixo):

```typescript
describe('query handler', () => {
  it('returns 200 with answer and cited chunk IDs for a valid question', async () => {
    // Arrange
    const request = buildQueryRequest({ question: 'Qual o prazo de devolução?' });
    mockSearchReturns(chunksFixtures.pol001DevolucaoPrazo);

    // Act
    const result = await handler(request);

    // Assert
    expect(result.status).toBe(200);
    expect(result.body.answer).toContain('7 dias úteis');
    expect(result.body.sources).toEqual(
      expect.arrayContaining([expect.objectContaining({ id: 'POL-001-A' })])
    );
  });
});
```

### O que todo teste NÃO DEVE ter

| Anti-padrão | Por quê |
|-------------|---------|
| **Acesso a serviços reais** (Azure AI Search, OpenAI, Cosmos, rede) | Flaky, lento, custo de tokens; quebra offline e no CI |
| **Dependência de ordem de execução** | Testes devem passar isoladamente (`vitest run path/to/file.test.ts`) |
| **Assertions vagas** (`toBeDefined()`, `toBeTruthy()`, `not.toBeNull()`) | Não provam comportamento correto — só que algo existe |
| **Testes que só executam código sem verificar efeito** | Smoke tests disfarçados de cobertura |
| **Dados hardcoded espalhados** no corpo do teste | Usar fixtures e factories em `tests/fixtures/` |
| **Timeouts longos ou retries** para mascarar flakiness | Corrigir a causa (mock faltando, race condition) |

**Anti-padrão explícito** — não gerar testes neste estilo:

```typescript
// ❌ REJEITAR: sem arrange claro, assertion vaga, não valida contrato
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

### Padrão de mocking

#### HTTP externo → MSW

Interceptar chamadas a Azure AI Search, Azure OpenAI e demais APIs via handlers MSW registrados no setup global (`tests/setup.ts` ou equivalente):

```typescript
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.post('https://*.search.windows.net/*', () =>
    HttpResponse.json({ value: searchResultFactory.buildList(3) })
  ),
  http.post('https://*.openai.azure.com/*', () =>
    HttpResponse.json(completionFactory.build({ content: 'Resposta simulada' }))
  )
);

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Regras MSW:

- `onUnhandledRequest: 'error'` — qualquer HTTP não mockado falha o teste (evita chamadas acidentais à rede).
- Handlers específicos por cenário via `server.use(...)` dentro do teste quando o default não bastar.
- Não mockar módulos internos que o teste de integração deve exercitar de verdade.

#### Dados de domínio → Factories

Objetos de teste criados por **factory functions** tipadas (não literais inline repetidos):

```typescript
// tests/fixtures/factories/search-result.factory.ts
export function buildSearchResult(overrides?: Partial<SearchResult>): SearchResult {
  return {
    id: 'POL-001-A',
    content: 'O cliente pode solicitar a devolução em até 7 dias úteis...',
    score: 0.92,
    documentId: 'POL-001',
    ...overrides,
  };
}

export function buildSearchResultList(count: number): SearchResult[] {
  return Array.from({ length: count }, (_, i) =>
    buildSearchResult({ id: `CHUNK-${i}`, score: 0.9 - i * 0.05 })
  );
}
```

Factories devem ter defaults sensatos e aceitar `overrides` parciais para cenários edge case.

### Padrão de fixtures (RAG)

Fixtures em `tests/fixtures/` centralizam dados reutilizáveis alinhados ao corpus NovaTech (Anexo B). **Não duplicar** chunks ou perguntas inline nos arquivos de teste.

| Arquivo | Conteúdo |
|---------|----------|
| `chunks.ts` | Chunks simulados com IDs estáveis (`POL-001-A`, `PROC-042v2-B`, `SLA-2024-A`, etc.) e metadados (`documentId`, `section`, `content`) |
| `queries.ts` | Perguntas de teste agrupadas por cenário (devolução, frete especial, SLA, multi-domínio, sem cobertura) |
| `expected-responses.ts` | Respostas esperadas ou critérios de validação (trechos obrigatórios, chunk IDs citados, comportamento quando não há cobertura) |

Estrutura recomendada para fixtures RAG:

```typescript
// tests/fixtures/queries.ts
export const ragQueries = {
  devolucao: {
    prazoGeral: 'Qual o prazo de devolução?',
    cargaPerigosa: 'Posso devolver carga perigosa?',
  },
  sla: {
    gold: 'Qual o SLA do cliente Gold?',
    tierInexistente: 'Qual o SLA do cliente Platinum?', // deve citar SLA-2024-A / FAQ-15
  },
  frete: {
    especialManaus: 'Frete para 600kg para Manaus?',
    padraoAbaixo500kg: 'Frete para 300kg para Salvador?', // sem cobertura na base
  },
} as const;

// tests/fixtures/chunks.ts
export const chunks = {
  pol001Prazo: buildChunk({ id: 'POL-001-A', documentId: 'POL-001', /* ... */ }),
  pol001Excecoes: buildChunk({ id: 'POL-001-B', /* ... */ }),
  proc042v2Nordeste: buildChunk({ id: 'PROC-042v2-B', /* ... */ }),
  slaTiers: buildChunk({ id: 'SLA-2024-A', /* ... */ }),
} as const;

// tests/fixtures/expected-responses.ts
export const expected = {
  devolucaoPrazo: {
    mustMention: ['7 dias úteis'],
    mustCiteChunks: ['POL-001-A'],
  },
  tierPlatinum: {
    mustMention: ['Gold', 'Silver', 'Standard'],
    mustNotInvent: ['Platinum'],
    mustCiteChunks: ['SLA-2024-A'],
  },
  freteAbaixo500kg: {
    mustIndicateNoCoverage: true, // assistente não deve inventar tabela
  },
} as const;
```

Cenários obrigatórios em testes RAG (retrieval + geração ou validação determinística):

- **Recuperação correta** — pergunta mapeia aos chunks esperados (ver mapa de cobertura do Anexo B).
- **Contradição PROC-042 vs v2** — quando ambos os chunks aparecem, resposta deve priorizar v2 ou sinalizar ambiguidade.
- **FAQ vs documento formal** — FAQ-32/FAQ-38 não substituem POL/PROC para regras críticas.
- **Pergunta sem cobertura** — resposta deve indicar ausência de informação, não alucinar.
- **Tier inexistente (Platinum)** — deve corrigir com base em SLA-2024-A.

Fixtures são **fonte única de verdade** nos testes; alterações no corpus de referência devem atualizar fixtures e testes juntos.

## Project Management Rules (Delivery Manager)
<!-- TODO (Delivery Manager — Ex. 2.3) -->

## Build & Deploy
<!-- TODO (Tech Lead — Ex. 2.1) -->
