<div align="center">

# launch-audit-skills

Skills de agente (formato aberto [Agent Skills](https://agentskills.io)) que auditam site, API ou aplicativo **antes de ir ao ar**, em três frentes: **segurança**, **risco jurídico no Brasil** e **qualidade de lançamento**. Toda conclusão vem com evidência, e o relatório também diz o que o projeto **não precisa**. Testadas em 02/11/2026 no Claude Code, no OpenAI Codex, no Gemini CLI, no Cursor e no GitHub Copilot.

![licença](https://img.shields.io/github/license/guavovic/launch-audit-skills?style=flat-square&label=licen%C3%A7a)

[Placar](docs/placar.md) • [Mudanças](CHANGELOG.md)

</div>

## O que tem

| Skill | Responde | Cobre |
|---|---|---|
| `auditoria-seguranca` | "O projeto está seguro?" | Segredo exposto (código, histórico do git, bundle do front), controle de acesso (IDOR/BOLA), injeção, XSS, SSRF, CSRF, CORS, sessão e JWT, upload, erro que vaza código, rota de debug, rate limit e anti-bot, dependência vulnerável ou inventada por IA, prompt injection, backup, rollback, logs |
| `auditoria-legal` | "O site pode ser processado?" | LGPD, cookies e rastreamento, política de privacidade, termos, CDC e direito de arrependimento, Decreto 7.962, Marco Civil, ECA Digital, acessibilidade (LBI e WCAG 2.2), direito autoral, publicidade enganosa |
| `auditoria-site` | "O site está pronto?" | Formulários de ponta a ponta, 404, links, HTTPS e e-mail do domínio, Core Web Vitals, SEO técnico, prévia social, analytics e funil, uptime, rastreamento de erro |

Junto vêm um **script de checagem passiva** (Python sem dependências, saída em JSON, compara com a rodada anterior) e o subagente **`auditoria-executor`**, que roda a parte braçal (script, Lighthouse, buscas) e devolve a evidência. Na ferramenta sem subagente, o agente principal faz essa parte com as mesmas regras.

## Como pensa

- **Proporcional.** Primeiro descobre o tipo de projeto (portfólio, SaaS, loja, API, app com LLM) e monta uma matriz do que se aplica. WAF num portfólio estático ou banner de cookies num site sem rastreador aparecem como "não precisa", com o motivo.
- **Sem chute.** Cada achado tem arquivo e linha, requisição ou saída de comando. O que não deu pra provar vai pra "não verificado".
- **Seguro por padrão.** Teste ativo só em `localhost`. Em produção, só checagem passiva: headers, TLS, DNS, arquivos que não deviam estar públicos. Segredo encontrado nunca é escrito por inteiro.
- **Feito pro Brasil.** A frente jurídica parte da LGPD, das resoluções da ANPD, do CDC e do ECA Digital, e avisa quando é hora de falar com um advogado.

## Instalação

A fonte fica em `skills/` e `agents/`. Cada ferramenta tem uma cópia pronta em [`embalagens/`](embalagens/), com o lugar exato de cada arquivo no [`embalagens/README.md`](embalagens/README.md). Copie as três skills juntas: as skills legal e site usam o script da skill de segurança pela pasta vizinha. O teste em cada ferramenta está em [docs/placar/ferramentas-2026-11-02.md](docs/placar/ferramentas-2026-11-02.md).

### Claude Code

Testado em 02/11/2026. Como plugin, dentro de uma sessão:

```
/plugin marketplace add guavovic/launch-audit-skills
/plugin install launch-audit-skills@launch-audit-skills
```

Ou pelo terminal, com `claude plugin marketplace add guavovic/launch-audit-skills` e `claude plugin install launch-audit-skills@launch-audit-skills`. As skills são chamadas pelo pedido ("audita a segurança desse projeto", "o site tá pronto pra lançar?") ou pelo nome, como `/launch-audit-skills:auditoria-seguranca`. Pra receber as mudanças depois: `/plugin marketplace update launch-audit-skills`.

Sem plugin, copie as pastas de `skills/` pra `~/.claude/skills/` e o arquivo de `agents/` pra `~/.claude/agents/`. Use um jeito ou o outro, não os dois juntos, senão cada skill aparece duas vezes.

### OpenAI Codex

Testado em 02/11/2026. Copie `embalagens/codex/.agents/` e `embalagens/codex/.codex/` pra raiz do projeto (ou as skills em `~/.agents/skills/` e o agente em `~/.codex/agents/`). No Windows, o `codex exec` só roda comando com `-c windows.sandbox="unelevated"` e aprovação.

### Gemini CLI

Testado em 02/11/2026, com o modelo `gemini-3.5-flash-lite` porque a cota grátis do dia do modelo padrão tinha acabado. `gemini extensions install <clone>/embalagens/gemini`, ou `gemini extensions link` pra acompanhar o clone.

### Cursor

Testado em 02/11/2026. Copie `embalagens/cursor/.cursor/` pra raiz do projeto (ou skills e agente em `~/.cursor/skills/` e `~/.cursor/agents/`). O Cursor também lê hooks e skills de `~/.claude`, então um hook do Claude Code vale lá.

### GitHub Copilot

Testado em 02/11/2026 no Copilot CLI. Copie `embalagens/copilot/.github/` pra raiz do repositório (no Copilot CLI, skills e agente também valem em `~/.copilot/skills/` e `~/.copilot/agents/`).

### Regras locais

Regras só suas (alvos que não podem ser tocados, onde salvar o relatório, qual perfil de navegador usar) vão num `regras-locais.md` dentro da pasta de cada skill. Ele é lido antes de tudo e fica fora do git. Na instalação por plugin do Claude Code a pasta é substituída a cada atualização, então quem usa regras locais prefere a cópia manual (ou um link pra um clone do repositório).

## Limites

Não substitui pentest feito por gente nem parecer jurídico. A skill aponta o que encontrou e o que não conseguiu verificar. Use só em projetos seus ou com autorização por escrito do dono.

## Placar

As skills são medidas contra aplicações vulneráveis de propósito, que têm a lista oficial das falhas, e comparadas com ferramentas conhecidas. Método e resultados em [docs/placar.md](docs/placar.md).

---

Licença MIT · Feito por [Gustavo Victor](https://github.com/guavovic)
