Abaixo está o antes/depois do teste ruim, reescrito conforme os padrões do AGENTS.md e da skill testing-patterns.md.

Antes (rejeitar)
// Teste gerado pelo Copilot sem guidance
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
Depois (padrão NovaTech)
import { http, HttpResponse } from 'msw';
import { server } from '../../setup';
import { queryHandler } from '../../../src/functions/query/handler';
import { buildQueryRequest } from '../../fixtures/factories/query-request.factory';
import { buildSearchResult } from '../../fixtures/factories/search-result.factory';
import { ragQueries } from '../../fixtures/queries';
import { chunks } from '../../fixtures/chunks';
describe('query handler', () => {
  describe('when question is valid', () => {
    it('returns 200 with answer and cited chunk IDs for a devolucao question', async () => {
      // Arrange
      const request = buildQueryRequest({
        question: ragQueries.devolucao.prazoGeral,
      });
      server.use(
        http.post('https://*.search.windows.net/*', () =>
          HttpResponse.json({
            value: [buildSearchResult(chunks.pol001Prazo)],
          })
        ),
        http.post('https://*.openai.azure.com/*', () =>
          HttpResponse.json({
            choices: [
              {
                message: {
                  content:
                    'O prazo para devolução é de 7 dias úteis após o recebimento confirmado.',
                },
              },
            ],
          })
        )
      );
      // Act
      const result = await queryHandler(request);
      // Assert
      expect(result.status).toBe(200);
      expect(result.body.answer).toContain('7 dias úteis');
      expect(result.body.sources).toEqual(
        expect.arrayContaining([
          expect.objectContaining({ id: 'POL-001-A' }),
        ])
      );
    });
  });
  describe('when question is missing', () => {
    it('returns 400 with validation error details', async () => {
      // Arrange
      const request = buildQueryRequest({ question: '' });
      // Act
      const result = await queryHandler(request);
      // Assert
      expect(result.status).toBe(400);
      expect(result.body.error).toMatch(/question/i);
    });
  });
});
Melhorias, uma a uma
#	Problema no “antes”	Melhoria no “depois”	Por quê
1
Nome 'query endpoint works'
describe('query handler') + it('returns 200 with answer and cited chunk IDs...')
Nome genérico não documenta comportamento. Frases em inglês no describe/it funcionam como especificação legível.
2
Um único test plano
Dois blocos: cenário válido e cenário inválido
Um teste só cobre o caminho feliz. Validação de input (Zod) também precisa de teste explícito.
3
Sem Arrange / Act / Assert
Comentários // Arrange, // Act, // Assert
Deixa claro o que é setup, execução e verificação — facilita revisão e manutenção.
4
body: '{"question": "test"}' inline
buildQueryRequest({ question: ragQueries.devolucao.prazoGeral })
Factory tipada monta o request corretamente; fixture traz pergunta de domínio real, não "test".
5
Pergunta "test" sem significado
ragQueries.devolucao.prazoGeral → "Qual o prazo de devolução?"
Permite assertar conteúdo concreto (7 dias úteis, chunk POL-001-A) alinhado ao Anexo B.
6
Nenhum mock de API
MSW intercepta Azure AI Search e Azure OpenAI
Sem mock, o teste chamaria serviços reais (flaky, lento, quebra no CI).
7
expect(result).toBeDefined()
expect(result.status).toBe(200), toContain('7 dias úteis'), sources com POL-001-A
toBeDefined() só prova que algo voltou — não prova contrato HTTP, resposta correta nem citação de fontes RAG.
8
Não verifica contrato da resposta
Asserta status, answer e sources
O endpoint deve devolver resposta com fontes (source_document / chunk IDs) — isso é requisito central do assistente.
9
Handler genérico handler
queryHandler importado de src/functions/query/handler
Espelha a estrutura real do repo (tests/unit/functions/query/handler.test.ts ↔ src/...).
10
Sem cenário de erro
Segundo it retorna 400 com mensagem de validação
Erros de input são comportamento público; omiti-los infla coverage sem provar robustez.
Resumo
O teste original executa código e checa existência — isso quase nunca falha e não protege o produto.

O teste reescrito especifica um contrato verificável: pergunta real do domínio NovaTech, APIs mockadas via MSW, resposta com status, conteúdo esperado e chunks citados — exatamente o que o assistente RAG precisa garantir.