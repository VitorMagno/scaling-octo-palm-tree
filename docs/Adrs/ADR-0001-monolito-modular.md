# 0001. Usar monólito modular em vez de microserviços

- **Status:** Aceito
- **Data:** 2026-10-14

## Contexto
O sistema é uma plataforma B2B multiempresa de solicitações e aprovações (acesso a sistemas, compras, reembolsos). O projeto está no início, é desenvolvido por uma pessoa e as fronteiras do domínio ainda estão sendo descobertas.

Foram consideradas três opções: monólito tradicional, monólito modular e microserviços.

Microserviços resolvem problemas específicos: permitir que times independentes façam deploy separadamente, escalar partes do sistema de forma diferente e isolar falhas de componentes críticos. Nenhum desses problemas existe aqui: há uma única pessoa desenvolvendo, a carga é pequena e uniforme e nenhuma parte precisa sobreviver isolada das demais.

Além disso, as fronteiras do domínio ainda não estão maduras. Errar uma fronteira entre microserviços exige mudar APIs, bancos e deploys, e frequentemente produz um "monólito distribuído", com os custos dos dois modelos e as vantagens de nenhum. Um monólito tradicional, por outro lado, não impõe fronteira nenhuma e tende a acumular acoplamento.

## Decisão
A aplicação será um monólito modular em .NET: um único processo e um único deploy, dividido em módulos (`Solicitacoes`, `Aprovacoes`, `Tenants`), cada um em seu próprio projeto. O projeto `Aprovacoes.Api` é o host que compõe os módulos.

As fronteiras são garantidas pelas referências entre projetos:

- `Aprovacoes.Api` referencia os módulos;
- um módulo não referencia outro módulo (apenas, quando necessário, o projeto público de contratos de outro módulo — ver ADR 0003);
- `Aprovacoes.Tests` referencia `Aprovacoes.Api`.

## Consequências
**Positivas**

- Um único deploy, mais simples de operar e mais barato no GCP.
- Depuração, testes de integração e transações locais mais simples, sem chamadas de rede entre módulos.
- Fronteiras verificadas pelo compilador desde o início.
- Caminho aberto para extrair um módulo como serviço independente no futuro, se surgir uma necessidade real.

**Negativas**

- Todos os módulos escalam juntos.
- Uma falha grave em um módulo pode derrubar o processo inteiro.
- Exige disciplina para não furar as fronteiras (por exemplo, acessando tabelas de outro módulo); testes de arquitetura serão adicionados para verificar isso.
- A comunicação entre módulos precisa de um mecanismo explícito, definido no ADR 0003.
