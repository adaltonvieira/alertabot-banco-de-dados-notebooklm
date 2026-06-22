# alertabot-banco-de-dados-notebooklm

## 🎯 Contexto e Objetivos

Este repositório documenta meu Caderno Temático de estudos, construído com o auxílio do **NotebookLM**, como parte do desafio de projeto da [DIO](https://www.dio.me) "Acelere sua Aprendizagem com IA: Explore o Poder do NotebookLM".

**Assunto escolhido:** Modelagem e uso de Banco de Dados (PostgreSQL) aplicado a Bots de Alertas, com foco no mecanismo `LISTEN`/`NOTIFY` e triggers.

**Motivação:** Estou desenvolvendo um projeto pessoal chamado **AlertaBot**, um bot que precisa monitorar mudanças em um banco de dados PostgreSQL e disparar alertas em tempo real. Para construir essa funcionalidade com solidez técnica, usei a IA do NotebookLM como ferramenta de aprendizagem ativa, treinando-a com fontes confiáveis sobre o tema.

**Objetivos de estudo:**
- Entender como o PostgreSQL notifica aplicações externas sobre mudanças em tabelas;
- Aprender a modelar uma tabela de eventos/alertas de forma eficiente;
- Conhecer boas práticas e armadilhas de performance ao usar triggers e notificações;
- Consolidar esse conhecimento em um material de revisão (mini-guia) reutilizável para o desenvolvimento do AlertaBot.

---

## 📚 Curadoria de Fontes

Fontes selecionadas e carregadas no NotebookLM:

| # | Fonte | Tipo | Link |
|---|-------|------|------|
| 1 | PostgreSQL Docs — LISTEN | Documentação oficial | https://www.postgresql.org/docs/current/sql-listen.html |
| 2 | PostgreSQL Docs — NOTIFY | Documentação oficial | https://www.postgresql.org/docs/current/sql-notify.html |
| 3 | PostgreSQL Docs — CREATE TRIGGER | Documentação oficial | https://www.postgresql.org/docs/current/sql-createtrigger.html |
| 4 | tcn — trigger function to notify listeners (PostgreSQL Docs) | Documentação oficial | https://www.postgresql.org/docs/current/tcn.html |
| 5 | Notify Triggers in PostgreSQL — Yarsa Labs Blog | Artigo prático (com Node.js) | https://blog.yarsalabs.com/notify-triggers-in-postgresql/ |

---

## 🧠 Engenharia de Prompts e "Cicatrizes"

Perguntas estratégicas testadas no NotebookLM, com observações sobre o processo:

### Prompt 1
> "Resuma em 5 pontos como funciona o mecanismo LISTEN/NOTIFY no PostgreSQL"

**Resposta obtida (resumo da IA):**
1. **Registro de Ouvintes (LISTEN):** registra a sessão atual como ouvinte em um canal específico, podendo ser desfeito com `UNLISTEN`.
2. **Envio de Notificações e Payload (NOTIFY):** dispara um evento para um canal, podendo incluir uma carga útil em texto/JSON.
3. **Dependência de Transações:** notificações só são entregues após o `COMMIT`; se a transação for revertida, o evento é descartado.
4. **Automação via Triggers:** prática recomendada é disparar o `NOTIFY` dentro de triggers de `INSERT/UPDATE/DELETE`, ideal para sistemas em tempo real.
5. **Agrupamento e Performance:** o PostgreSQL agrupa notificações idênticas na mesma transação; uso excessivo em tabelas de alta frequência pode degradar a performance.

**Observação:** resposta direta, bem estruturada e já citando corretamente as fontes 1 e 2. Não precisei reformular.

### Prompt 2
> "Quais são as boas práticas para evitar sobrecarga ao usar triggers de notificação em alta frequência?"

**Resposta obtida (resumo da IA):**
1. Avaliar a necessidade real antes de notificar a cada mudança;
2. Filtrar eventos com `UPDATE OF coluna` e cláusula `WHEN (OLD.coluna IS DISTINCT FROM NEW.coluna)`;
3. Preferir triggers `FOR EACH STATEMENT` (em vez de `FOR EACH ROW`) quando possível, combinando com `REFERENCING` para inspecionar todas as linhas afetadas de uma vez;
4. Aproveitar a deduplicação nativa do PostgreSQL (mesmo canal + mesmo payload na mesma transação = uma única notificação);
5. Implementar throttling para tabelas muito dinâmicas;
6. Cuidado com a fila de notificações (limite padrão de **8GB**) — transações longas em sessões ouvintes impedem a limpeza da fila e podem travar futuros `NOTIFY`;
7. Ignorar "ecos": comparar o PID da notificação recebida com o PID da própria sessão para evitar reprocessamento.

**Observação/Dificuldade:** essa resposta trouxe um dado crítico que eu não esperava — o limite de 8GB na fila de notificações e o risco de travamento por transações longas. Vale a pena destacar isso no código do AlertaBot.

### Prompt 3
> "Crie um glossário com os termos: trigger, canal, payload, listener, AFTER/BEFORE"

**Resposta obtida (glossário):**
- **Trigger:** objeto associado a uma tabela/view que executa uma função automaticamente em resposta a `INSERT/UPDATE/DELETE/TRUNCATE`, disparando por linha (`FOR EACH ROW`) ou por instrução (`FOR EACH STATEMENT`).
- **Canal (channel):** identificador nomeado usado como destino de roteamento das notificações; clientes se inscrevem via `LISTEN`.
- **Payload:** mensagem de texto (geralmente JSON, via `json_build_object`) enviada junto ao `NOTIFY`; limite padrão de **8000 bytes**.
- **Listener:** sessão/aplicação registrada para escutar um canal via `LISTEN`; recebe canal, PID da sessão emissora e o payload.
- **AFTER/BEFORE:** momento de execução do trigger em relação à operação. `BEFORE` roda antes da operação e das checagens de restrição (pode alterar/cancelar a linha); `AFTER` roda depois, já com as restrições validadas (constraint triggers só funcionam com `AFTER`).

### Prompt 4
> "Como eu modelaria uma tabela de 'alertas' que dispara notificações automáticas para um bot externo?"

**Resposta obtida:** a IA gerou a modelagem completa em 4 passos — tabela `alertas`, função de trigger usando `json_build_object` + `pg_notify`, associação do trigger via `AFTER INSERT ... FOR EACH ROW`, e exemplo de listener em Node.js com `pg-listen`. O SQL completo está documentado na seção [Miniguia de Estudo](#-miniguia-de-estudo-entrega-final) abaixo.

**Observação:** resposta extremamente prática, já no formato pronto para implementar. Destacou dois cuidados importantes: payload deve ser menor que 8000 bytes (senão enviar apenas o `id` e buscar o resto via query), e manter transações curtas para garantir entrega em tempo real.

### Prompt 5
> "Compare LISTEN/NOTIFY com outras formas de monitorar mudanças no banco (polling, CDC, webhooks)"

**Resposta obtida (resumo):**
- **LISTEN/NOTIFY:** mecanismo nativo, modelo *push* assíncrono, ótimo para tempo real e sinalização interna; limitado por payload de 8000 bytes e pela fila de 8GB.
- **Polling:** cliente consulta o banco repetidamente; muito menos eficiente que LISTEN/NOTIFY, gera carga constante de CPU/rede mesmo sem mudanças.
- **CDC (Change Data Capture, ex: Debezium):** lê o WAL (log transacional) de forma passiva, fora do caminho da transação; mais robusto para alta escala e replicação entre microsserviços, mas mais complexo de operar.
- **Webhooks:** chamadas HTTP disparadas por eventos da aplicação; não exigem conexão persistente com o banco (diferente do LISTEN/NOTIFY) e são o padrão para integrar com serviços de terceiros.

**Observação importante:** a própria IA avisou que as fontes carregadas só cobrem LISTEN/NOTIFY em profundidade — as comparações com Polling, CDC e Webhooks vieram do conhecimento geral do modelo, não das fontes. Bom exemplo de **transparência da IA sobre os limites do que pode citar**, e mostra a importância de verificar essas partes de forma independente.

### Prompt extra
> "Dado o contexto do meu projeto AlertaBot, quais riscos de performance devo considerar?"

**Resposta obtida (resumo):** reforçou os 3 riscos centrais — sobrecarga geral por alta frequência de triggers, enchimento da fila de 8GB (acelerado por transações longas no próprio bot), e atraso na entrega por transações longas na aplicação que insere os alertas. Sugeriu como mitigação: throttling, filtro `WHEN`, transações curtas dos dois lados, payload pequeno (enviar `id` em vez do texto completo) e monitorar a fila com `pg_notification_queue_usage`.

> 💡 **Dica de ouro aplicada:** documentei não só as respostas, mas também os avisos de limite das fontes (ex: a IA sinalizando quando saiu do material carregado) e o dado mais valioso que surgiu — o limite de 8GB da fila de notificações, que vou monitorar diretamente no AlertaBot.

---

## 📖 Miniguia de Estudo (Entrega Final)

### Resumo Estruturado do Assunto

1. **O problema:** aplicações como bots de alertas precisam saber, em tempo real, quando algo muda no banco de dados.
2. **A solução nativa do PostgreSQL:** o par `LISTEN`/`NOTIFY` permite que uma sessão se inscreva em um canal e receba notificações assíncronas sempre que outro processo executar `NOTIFY` naquele canal.
3. **Automatizando com Triggers:** em vez de disparar `NOTIFY` manualmente, criamos um trigger (`AFTER INSERT/UPDATE/DELETE`) na tabela de interesse, que chama `pg_notify()` automaticamente.
4. **Do banco até o bot:** a aplicação (ex: Node.js com `pg-listen`, ou Python com `psycopg2`) mantém uma conexão persistente fazendo `LISTEN`, e processa o payload recebido para disparar o alerta (ex: mensagem no Telegram/WhatsApp).
5. **Cuidados de performance:** notificações são entregues apenas após o commit da transação; evitar disparar notificações excessivas (throttling/agrupamento) para não sobrecarregar o sistema.

### Glossário

| Termo | Definição |
|-------|-----------|
| **Trigger** | Função executada automaticamente pelo banco antes, depois ou no lugar de uma operação (INSERT/UPDATE/DELETE) em uma tabela. |
| **NOTIFY** | Comando que envia um evento de notificação (com payload opcional) para todos os clientes inscritos em um canal. |
| **LISTEN** | Comando que registra a sessão atual como "ouvinte" de um canal de notificação. |
| **Canal (channel)** | Nome usado para agrupar notificações relacionadas a um mesmo tipo de evento ou tabela. |
| **Payload** | String opcional enviada junto da notificação, contendo dados estruturados sobre o evento. |
| **pg_notify()** | Função SQL usada dentro de triggers para disparar notificações de forma programática. |

### Exemplo Prático: Tabela de Alertas + Trigger (gerado e validado no NotebookLM)

```sql
-- 1. Tabela de alertas
CREATE TABLE alertas (
    id SERIAL PRIMARY KEY,
    severidade VARCHAR(50) NOT NULL,
    mensagem TEXT NOT NULL,
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Função de trigger: empacota o evento em JSON e notifica o canal
CREATE OR REPLACE FUNCTION notificar_bot_alertas()
RETURNS trigger AS $$
DECLARE
    payload TEXT;
BEGIN
    payload := json_build_object(
        'evento', TG_OP,
        'tabela', TG_TABLE_NAME,
        'dados', row_to_json(NEW)
    )::text;

    PERFORM pg_notify('canal_bot_alertas', payload);

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 3. Associação do trigger à tabela
CREATE TRIGGER trigger_novo_alerta
AFTER INSERT ON alertas
FOR EACH ROW
EXECUTE FUNCTION notificar_bot_alertas();
```

```javascript
// 4. Listener no lado do bot (Node.js + pg-listen)
const createSubscriber = require('pg-listen');

const subscriber = createSubscriber({ connectionString: "sua_string_de_conexao_aqui" });

subscriber.notifications.on("canal_bot_alertas", (payload) => {
    console.log("Novo alerta recebido do banco:", payload);
    // enviarMensagemParaChat(payload.dados.mensagem);
});

subscriber.listenTo("canal_bot_alertas").then(() => {
    console.log("Bot aguardando alertas no canal 'canal_bot_alertas'...");
});
```

**Cuidados validados com a IA:**
- Payload deve ter **menos de 8000 bytes** — se o alerta for grande, envie só o `id` e busque o resto via query.
- A fila de notificações tem limite padrão de **8GB**; transações longas (no emissor ou no ouvinte) impedem sua limpeza e podem travar o `NOTIFY`.
- Mantenha as transações curtas dos dois lados para garantir entrega em tempo real.

### Prompts Reutilizáveis (para revisões futuras)

```
1. "Explique [conceito] do PostgreSQL como se eu fosse um desenvolvedor júnior"
2. "Quais erros comuns acontecem ao implementar [funcionalidade] e como evitá-los?"
3. "Gere um exemplo de código em [linguagem] usando [conceito] no contexto de um bot de alertas"
4. "Compare [abordagem A] com [abordagem B] em termos de performance e complexidade"
5. "Crie 5 perguntas de revisão sobre [tema] para eu testar meu próprio entendimento"
```

---

## 🛠️ Próximos Passos no AlertaBot

- [ ] Modelar a tabela `alertas` (id, tipo, payload, criado_em, processado)
- [ ] Criar trigger `AFTER INSERT` chamando `pg_notify('alertas_channel', ...)`
- [ ] Implementar listener na aplicação (Node.js/Python) para consumir os eventos
- [ ] Conectar o listener ao serviço de envio de mensagens (Telegram/WhatsApp/Email)
- [ ] Testar throttling para evitar flood de notificações

---

## 🤖 Sobre o uso do NotebookLM

Todo o processo de curadoria, leitura e síntese das fontes acima foi conduzido com apoio do NotebookLM, usado como ferramenta de aprendizagem ativa — e não apenas como gerador de respostas prontas. As perguntas foram refinadas iterativamente, e as respostas foram cruzadas com a documentação oficial para validação.

---

**Desafio:** Treinando uma IA de Aprendizagem: Explore o Poder do NotebookLM — [DIO](https://www.dio.me)
