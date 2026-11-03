# Teste por ferramenta

A mesma auditoria rodou em cada ferramenta de agente que tem embalagem em [`embalagens/`](../../embalagens/), pra saber se a skill é achada, se o script roda, se o subagente funciona e se o resultado bate com o do Claude Code. O README só diz que funciona numa ferramenta depois que esse teste passou nela (#14).

## A rodada

| Campo | Valor |
|---|---|
| Data | 02/11/2026, das 21:46 às 22:43 |
| Commit da skill | `8d2c52c` (fonte), com a correção do gerador descrita abaixo |
| Alvo | `https://guavovic.vercel.app`, site do autor, só a checagem passiva |
| Skill | `auditoria-seguranca` |
| Pedido | o mesmo texto em todas: usar a skill, rodar a checagem passiva, montar a matriz do que se aplica a um site estático pessoal, sem teste ativo, salvar o JSON como `passiva.json` e responder em pt-BR com o relatório curto |
| Pasta | uma pasta vazia por ferramenta, fora do repositório, só com a embalagem daquela ferramenta (sem código-fonte do site) |

Observações de método:

- Cada ferramenta rodou sem interação (`claude -p`, `codex exec`, `gemini -p`, `cursor-agent -p`), com o mínimo de permissão que deixasse o script rodar: nada de modo "aceita tudo".
- A Vercel bloqueia o IP por alguns minutos depois das requisições a `/.env` e `/.git` que a checagem passiva faz. As rodadas que chegaram a rodar o script ficaram com uns 10 minutos ou mais entre uma e outra.
- "Achados batem" compara o resumo do `passiva.json` de cada ferramenta com o do Claude Code e confere se o relatório cita os mesmos avisos.
- Nenhuma embalagem ficou instalada fora da pasta de teste no fim. A extensão do Gemini foi ligada por `gemini extensions link` e desligada depois.

## Resultado por ferramenta

| | Claude Code | OpenAI Codex | Gemini CLI | Cursor | GitHub Copilot |
|---|---|---|---|---|---|
| Versão e modelo | 2.1.316, `claude-opus-5-5` | CLI 0.166.0, `gpt-6-luna` | 0.66.0, `gemini-3.5-flash-lite` | CLI 2026.10.22, modelo Auto | não rodou |
| Embalagem | junction em `~/.claude/skills` | `.agents/skills` e `.codex/agents` na pasta | extensão (`embalagens/gemini`) | `.cursor/` na pasta | `.github/` |
| Skill achada e carregada | sim, pela ferramenta de skill | sim, leu o `SKILL.md` | sim, `activate_skill` | sim, leu o `SKILL.md` e a regra `.mdc` | |
| Script rodou | sim | sim, com caminho relativo à pasta | sim, com o caminho que a extensão deu | sim | |
| Subagente | não, o agente principal fez a parte braçal | não, o agente principal fez a parte braçal | não, o agente principal fez a parte braçal | sim, `auditoria-executor` rodou o script | |
| Matriz do que se aplica | sim | sim | sim | sim | |
| Resumo do `passiva.json` | 43 ok, 5 info, 3 avisos | 43 ok, 5 info, 3 avisos | 43 ok, 5 info, 3 avisos | 42 ok, 5 info, 3 avisos, 1 bloqueado | |
| Achados batem | referência | sim | sim | sim (o bloqueado é o anti-bot da Vercel) | |
| Tempo | cerca de 70 s | cerca de 40 s | cerca de 35 s | cerca de 76 s | |
| Status | passou | passou com ressalva | passou com ressalva | passou com ressalva | não testado |

Os três avisos são os mesmos em todas: CORS com `*` na home (sem credencial), dois `h1` na página inicial e detalhe técnico na página 404. Nenhuma falha. O Claude Code foi o único que citou o Turnstile na CSP; os outros não erraram, só não comentaram.

## Ressalvas e problemas

### OpenAI Codex

Na primeira tentativa, com `-s workspace-write` e rede liberada, todo comando de shell veio "blocked by policy" no Windows. O Codex achou a skill, não conseguiu rodar nada e disse isso no relatório, sem inventar resultado. O que funcionou foi `-c windows.sandbox="unelevated"` com aprovação de comando e rede liberada no sandbox. Só o `windows.sandbox="unelevated"`, sem aprovação, deixou o comando parado. Fora do Windows não foi testado.

### Gemini CLI

A extensão foi aceita, listou as três skills e a skill foi ativada na primeira tentativa. As duas primeiras rodadas no modelo padrão pararam em erro da API (503 de alta demanda, depois "fetch failed") e a terceira na cota grátis do dia (20 requisições por dia no modelo padrão). Pedir `gemini-3.5-flash` caiu de volta no mesmo modelo e na mesma cota. A rodada que passou usou `gemini-3.5-flash-lite`. Falta repetir no modelo padrão quando a cota permitir; a skill não mudou entre as rodadas.

### Cursor

A primeira rodada falhou por dois motivos:

- **Nome do subagente com aspas.** O gerador escrevia `name: "auditoria-executor"` e o Cursor leu as aspas como parte do nome, então a chamada do subagente não achava o agente. Corrigido no gerador: `name` sai sem aspas em todas as embalagens.
- **Hooks do Claude Code.** O Cursor lê hooks e skills de `~/.claude` (ligado por padrão). Um hook de `PreToolUse` do Claude Code da máquina de teste quebrou todo comando de terminal do agente. Sem terminal, o agente montou o `passiva.json` copiando o resultado das outras pastas de teste, avisando no relatório que fez isso. Por isso a rodada foi descartada. O risco fica registrado: sem conseguir rodar o script, o agente pode entregar evidência que não é dele.

A segunda rodada, com a correção do nome e com a pasta de usuário isolada (sem `~/.claude`), passou e usou o subagente. Quem tem Claude Code e Cursor na mesma máquina precisa saber que os hooks e as skills de um valem no outro, e que uma skill instalada nos dois lugares aparece duas vezes.

### GitHub Copilot

Não rodou: na máquina de teste o comando `copilot` é só o atalho do VS Code e responde que o Copilot CLI não está instalado. Fica "ainda não testado" no README até o teste com o Copilot CLI autenticado (#14).

## O que ficou de fora

- Copilot, pelo motivo acima.
- Gemini no modelo padrão, por cota.
- Codex e Cursor fora do Windows.
- As skills `auditoria-legal` e `auditoria-site`. Elas usam o mesmo script e a mesma estrutura de embalagem, mas não rodaram nesta rodada.
- Modo ativo e alvo local: o teste é sobre a embalagem, não sobre a cobertura da skill, que já está medida no [Juice Shop](juice-shop-2026-10-26.md).
