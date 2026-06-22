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
| 3 | Postgres Triggers with Listen/Notify — LaunchPad Lab (Medium) | Artigo técnico | https://medium.com/launchpad-lab/postgres-triggers-with-listen-notify-565b44ccd782 |
| 4 | tcn — trigger function to notify listeners (PostgreSQL Docs) | Documentação oficial | https://www.postgresql.org/docs/current/tcn.html |
| 5 | Notify Triggers in PostgreSQL — Yarsa Labs Blog | Artigo prático (com Node.js) | https://blog.yarsalabs.com/notify-triggers-in-postgresql/ |

---

## 🧠 Engenharia de Prompts e "Cicatrizes"

Perguntas estratégicas testadas no NotebookLM, com observações sobre o processo:

### Prompt 1
> "Resuma em 5 pontos como funciona o mecanismo LISTEN/NOTIFY no PostgreSQL"

**Resposta obtida:** *(cole aqui o resumo gerado pela IA)*

**Observação:** *(ex: a IA citou corretamente as fontes 1 e 2, resposta direta e útil)*

### Prompt 2
> "Quais são as boas práticas para evitar sobrecarga ao usar triggers de notificação em alta frequência?"

**Resposta obtida:** *(cole aqui)*

**Observação/Dificuldade:** *(ex: precisei reformular o prompt pedindo exemplos práticos, pois a primeira resposta ficou genérica)*

### Prompt 3
> "Crie um glossário com os termos: trigger, canal, payload, listener, AFTER/BEFORE"

**Resposta obtida:** *(cole aqui)*

### Prompt 4
> "Dado o contexto do meu projeto AlertaBot, quais riscos de performance devo considerar ao usar essa abordagem?"

**Resposta obtida:** *(cole aqui)*

**Observação:** *(ex: a IA generalizou demais até eu fornecer mais contexto sobre o volume de dados esperado)*

### Prompt 5
> "Compare LISTEN/NOTIFY com outras formas de monitorar mudanças no banco (polling, CDC, webhooks)"

**Resposta obtida:** *(cole aqui)*

> 💡 **Dica de ouro aplicada:** documentei não só as respostas, mas também as referências citadas pela IA e as dificuldades de troubleshooting (prompts que precisaram ser reformulados para obter respostas mais precisas).

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
