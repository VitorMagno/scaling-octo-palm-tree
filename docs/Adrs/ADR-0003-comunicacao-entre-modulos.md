# 0003. Comunicação entre módulos: eventos para fatos, interfaces para consultas

- **Status:** Aceito
- **Data:** 2026-10-14

## Contexto
Pelo ADR 0001, um módulo não referencia outro. Ainda assim, os módulos precisam colaborar. Por exemplo, quando uma solicitação é criada, o módulo de Aprovações precisa criar a aprovação pendente, e Notificações precisa avisar o solicitante quando o status muda.

Foram consideradas as seguintes abordagens:

| Abordagem | Problema |
| --- | --- |
| Referência direta entre módulos | Acoplamento total e risco de referências circulares |
| Banco compartilhado (ler tabelas de outro módulo) | Acoplamento escondido: mudanças de esquema quebram outros módulos sem aviso |
| Orquestração na Api | A regra de negócio migra dos módulos para a Api |
| Polling | Atraso e carga desnecessária |
| Interface pública síncrona | Adequada para consultas, mas quem publica um fato precisaria conhecer cada interessado |
| Eventos | Consistência eventual e mais cuidado com rastreabilidade e idempotência |

## Decisão

- **Fatos aos quais outros módulos reagem** (ex.: `SolicitacaoCriada`, `SolicitacaoAprovada`) são comunicados por **eventos**. O módulo publica o evento sem saber quem o consome.
- **Consultas que precisam de resposta imediata** (ex.: obter dados resumidos de uma solicitação) usam uma **interface pública** do módulo, resolvida por injeção de dependência.
- Eventos e interfaces públicas ficam em um projeto de contratos por módulo (ex.: `Aprovacoes.Solicitacoes.Contracts`). Outros módulos podem referenciar apenas esse projeto, nunca o módulo em si.
- Na primeira versão, os eventos são despachados em processo. Na fase 2, passam a ser gravados com o padrão outbox e publicados no RabbitMQ.

## Consequências
**Positivas**

- Novos consumidores (notificações, auditoria) se inscrevem nos eventos sem alterar o módulo que publica.
- Os módulos permanecem desacoplados e podem ser extraídos no futuro com mudanças limitadas.
- O mesmo modelo de eventos atende ao outbox, à fila e às notificações em tempo real via SignalR.

**Negativas**

- Consistência eventual: por um curto período, a solicitação pode existir sem a aprovação correspondente.
- Os consumidores precisam ser idempotentes, porque um evento pode ser entregue mais de uma vez.
- O fluxo fica menos explícito no código; será necessário tracing (OpenTelemetry) para acompanhar uma requisição entre módulos.
- Os projetos de contratos precisam ser mantidos estáveis e versionados com cuidado.
