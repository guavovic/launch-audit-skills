---
name: auditoria-site
description: "Audita se um site está pronto pra lançar e se mantém saudável: formulários que funcionam de ponta a ponta (validação, confirmação, página de obrigado, bloqueio de envio duplo), página 404, links quebrados, domínio próprio com HTTPS e e-mail configurado (SPF, DKIM, DMARC), velocidade (PageSpeed, Core Web Vitals, imagens comprimidas, teste em rede móvel), SEO (title e meta description únicos, sitemap.xml, robots.txt, canonical, dados estruturados, Search Console, palavras-chave), prévia social (favicon, og:image), analytics com eventos e funil de desistência, monitor de uptime, rastreamento de erro, rodapé com dados, e itens de conversão (CTA fixo no mobile, FAQ, avaliações e cases reais, Google Meu Negócio e mapa quando for negócio local). Diz o que se aplica ao tipo de site. Use quando o usuário pedir \"o site tá pronto?\", \"checklist de lançamento\", \"auditoria do site\", \"SEO do site\", \"por que o site tá lento?\", \"revisão do site\", ou a revisão periódica de 3 a 6 meses."
---

<!-- Arquivo gerado por ferramentas/gerar_embalagens.py a partir de skills/ e agents/. Não edite à mão: mude a fonte e rode o script de novo. -->

> `<pasta-desta-skill>` é a pasta onde está este SKILL.md. As três skills ficam lado a lado, então `<pasta-desta-skill>/../auditoria-seguranca` é a pasta da skill de segurança.

# Auditoria do site

**Regras locais:** se existir `regras-locais.md` nesta pasta, leia antes de tudo. Ele guarda as regras de quem usa a skill (alvos proibidos, onde salvar o relatório, qual navegador usar, como abrir issue) e vale acima do padrão daqui.

Objetivo: o site **funciona de verdade, carrega rápido, aparece no Google, compartilha bonito e avisa quando cai**. Cada achado com evidência: URL, print, nota do Lighthouse, resposta HTTP.

Segurança fica na skill `auditoria-seguranca` e risco jurídico na `auditoria-legal`. Se o usuário pedir "auditoria completa", rode as três. Na rede, a ordem é: Lighthouse e navegador primeiro, script passivo por último (o anti-bot pode bloquear o IP depois dele). No relatório, a ordem é: segurança, legal, site.

## 1. Perfil

1. **Tipo**: portfólio, landing de produto, institucional de negócio, negócio local (endereço físico), loja, app web, blog.
2. **Objetivo do site**: contato, venda, cadastro, mostrar trabalho. Isso decide o que é "conversão".
3. **Stack e hospedagem**: Astro, Next, estático, Vercel, Netlify, VPS. Descoberta a stack, leia só o arquivo de `references/stacks/` de cada tecnologia usada: `vercel.md`, `astro.md`, `pwa.md` (service worker ou manifesto). Stack sem arquivo segue só o checklist geral.

Itens de conversão e de negócio local **só entram quando fazem sentido**:

| Item | Portfólio | Landing / produto | Negócio local | Loja |
|---|---|---|---|---|
| CTA fixo no mobile | opcional | sim | sim | sim |
| FAQ (perguntas reais, sem número fixo) | - | sim | sim | sim |
| Avaliações e cases **reais** | depoimento se houver | sim | sim | sim |
| Foto da equipe | - | opcional | recomendado | opcional |
| Google Meu Negócio, mapa, rota, marcação LocalBusiness | - | - | sim | se tiver loja física |
| Promessa de tempo de resposta | só se cumprir | só se cumprir | só se cumprir | só se cumprir |
| Página de obrigado | se tiver formulário | sim | sim | sim (pedido) |
| Breadcrumbs | - | se tiver muitas páginas | se tiver muitas páginas | sim |

Nada de inventar avaliação, case, número de clientes ou selo: isso é risco legal, não melhoria.

## 2. Verificação

Parte braçal (Lighthouse, script passivo, npm audit, buscas no código, axe no navegador): delegue ao subagente `auditoria-executor` se a ferramenta tiver subagente; se não tiver, ou se ele estiver desligado, você mesmo faz a parte braçal com as mesmas regras dele (só o alvo pedido, nada que grave em produção, segredo com só 4 caracteres à mostra). Ao delegar, passe o alvo, o que verificar, onde salvar e o caminho completo do script e de `references/comandos.md` da skill `auditoria-seguranca` (ele não tem como descobrir sozinho onde as skills foram instaladas). Eu fico com perfil, análise, severidade e relatório.

Siga `references/checklist.md`. Ferramentas:

- **Navegador**: Playwright (o perfil indicado nas regras locais, se houver). Testar em 1366x768 e 390x844 (celular), com print de cada problema.
- **Checagem automática do `<head>`, robots, sitemap, 404, TLS e DNS**: o script `<pasta-desta-skill>/../auditoria-seguranca/scripts/checagem_passiva.py` (ver a skill de segurança). Rode o Lighthouse **antes** dele, porque o anti-bot da hospedagem pode bloquear o IP depois.
- **Velocidade**: `npx --yes lighthouse@12 <url> --chrome-flags="--headless=new" --output=json` (mobile) e com `--preset=desktop`, salvando numa pasta temporária. A API do PageSpeed sem chave usa uma cota anônima que costuma estar esgotada: se o usuário tiver uma chave, passe `&key=`; senão, use o Lighthouse local. Simular rede móvel lenta no Playwright pra ver o "abrir no 5G/4G ruim".
- **Links**: `npx --yes linkinator <url> --recurse` (ou rastrear pelo sitemap).
- **SEO e social**: ler o `<head>` de cada página; conferir `sitemap.xml`, `robots.txt`; validar prévia com o depurador da Meta e o Rich Results Test do Google.
- **DNS e e-mail**: `nslookup` pra SPF, DKIM, DMARC, CAA.
- **Formulários**: no local ou com dado claramente de teste, com ok do usuário antes de enviar em produção.

## 3. Relatório

Salve em `auditorias/<projeto>-site-AAAA-MM-DD.md` (na raiz do projeto auditado, ou onde as regras locais mandarem), com o JSON da checagem passiva ao lado.

1. **Veredito**: pronto / pronto depois de X / não lançar.
2. **Notas** do Lighthouse (desempenho, acessibilidade, boas práticas, SEO) desktop e mobile, e os Core Web Vitals.
3. **Achados** em três grupos: **Quebra** (algo não funciona), **Perde visita ou venda** (lento, sem SEO, sem prévia, sem CTA), **Polimento**.
4. **Não se aplica**, com o porquê.
5. **Próxima revisão**: 3 a 6 meses, ou antes de campanha.

## 4. Depois

- Ofereça issues; correção só com ok, uma branch por tema.
- Sugira o que precisa de conta do usuário (Search Console, Google Meu Negócio, UptimeRobot, Sentry): eu não crio conta em nome de ninguém, só passo o passo a passo.
- Revisão periódica: ofereça agendar (se a ferramenta tiver agendamento).
