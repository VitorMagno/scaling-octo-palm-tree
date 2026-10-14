# 0002. Usar controllers em vez de minimal APIs

- **Status:** Aceito
- **Data:** 2026-10-14

## Contexto
O ASP.NET Core oferece dois modelos para expor endpoints HTTP:

- **Controllers:** classes que agrupam ações relacionadas, com roteamento, filtros e model binding baseados em atributos.
- **Minimal APIs:** endpoints declarados como funções (`app.MapGet(...)`), com menos cerimônia e um pouco menos de overhead.

Os dois são suportados e evoluídos pela Microsoft, e o desempenho de ambos é mais que suficiente para este sistema. A API terá vários recursos por módulo (solicitações, aprovações, tenants), com autorização por papel e por tenant, validação, paginação, versionamento e documentação OpenAPI.

## Decisão
A API usará controllers, com um ou mais controllers por módulo. Cada módulo expõe seus próprios controllers, e o host os registra.

## Consequências
**Positivas**

- Organização explícita por recurso, que acompanha a divisão em módulos.
- Filtros, atributos de autorização (`[Authorize]` com policies) e convenções aplicáveis a grupos de ações.
- Modelo familiar para a maioria dos times .NET e comum em sistemas corporativos, o que facilita a leitura do código por avaliadores e recrutadores.

**Negativas**

- Mais cerimônia e arquivos do que minimal APIs para endpoints simples.
- Um overhead pequeno por requisição, irrelevante na escala deste sistema.
- Exige cuidado para que os controllers não acumulem regra de negócio; eles devem apenas traduzir HTTP para chamadas ao módulo.
