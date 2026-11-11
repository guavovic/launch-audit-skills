# guavovic-website

Primeira rodada real das três skills, em 21/10/2026, no portfólio pessoal do autor ([guavovic.com](https://guavovic.com)). Os achados abaixo foram todos corrigidos em 22/10/2026, antes desta publicação.

## O projeto

Portfólio pessoal com fim profissional, em Astro e TypeScript, estático na Vercel, com três funções no servidor: formulário de contato (Turnstile e Resend), um mural de visitas (Upstash Redis) e um proxy pro Last.fm com a chave guardada no servidor. Sem login, sem venda, sem rastreador e sem cookie próprio.

A rodada leu o código e fez só checagem passiva em produção (headers, TLS, DNS, `<head>`, Lighthouse). Nenhum teste ativo contra o site no ar.

## O que se aplicava e o que não

| Proteção | Aplica? | Por quê |
|---|---|---|
| HTTPS, HSTS, headers, CSP | sim | qualquer site no ar |
| Segredo fora do front e do git | sim | três funções com chave de serviço |
| Validação no servidor e anti-spam | sim | contato e mural recebem texto de fora |
| Rate limit | sim | mural e contato |
| Aviso de privacidade (LGPD) | sim | coleta nome, e-mail e apelido, e o site é profissional |
| Controle de acesso por objeto, sessão, 2FA, CSRF | não | não há login nem cookie de sessão |
| WAF próprio | não | a Vercel já desafia quem varre caminho sensível |
| Backup | não | nada importante guardado, o mural é descartável |
| Banner de cookies | não | não há cookie não essencial nem rastreador |
| Termos de uso, reembolso, dados de empresa, ECA Digital, guarda de logs do Marco Civil | não | não vende, não tem conta de usuário, não tem fim econômico direto |

Voltam a valer se o site passar a vender ou ter conta de usuário.

## Veredito

Nada crítico nem alto. Um achado médio e três baixos.

| Severidade | Frente | Achado | Correção |
|---|---|---|---|
| Média | Legal | O site coletava dado pessoal (contato, Turnstile, mural) e não dizia isso em lugar nenhum | Página curta de privacidade: o que coleta, pra quê, quem processa, por quanto tempo e como pedir pra apagar, com link perto do botão de enviar, no mural e no rodapé |
| Baixa | Segurança | O escape do JSON-LD não fazia nada: o comentário prometia que nenhum texto fecharia a tag `<script>`, mas a troca usava uma string que já era o próprio `<`. Sem como explorar, porque o conteúdo era fixo | Troca corrigida e conferida no HTML gerado: nenhum `<` sobra dentro do bloco e o JSON continua válido |
| Baixa | Segurança | `npm audit` com 3 altas numa dependência indireta do adaptador da Vercel, usada só no build. A correção sugerida pelo npm era um downgrade de versão major do adaptador | A skill recomendou não aplicar o downgrade. Depois, a versão corrigida da dependência foi forçada por `overrides` no `package.json`, com as rotas geradas idênticas às de antes e `npm audit` zerado |
| Baixa | Site | Imagens dos projetos sem `width` e `height` | Dimensões declaradas nas capas. O CLS já tinha caído por outra mudança no layout, então o ganho foi de garantia, não de nota |

Lighthouse no dia: desktop 100 em desempenho, acessibilidade, boas práticas e SEO; mobile 98, 100, 100, 100.

**Pontos fortes que a rodada confirmou:** nota A+ no securityheaders, TLS 1.3 com 1.0 e 1.1 recusados, CSP sem `unsafe-inline`, nenhuma chave no git (atual e histórico), só a chave pública do Turnstile no front, sem sourcemap público, erro genérico nas APIs, conferência de origem, token de admin comparado em tempo constante, apelido do mural restrito a letras, números e poucos símbolos, proxy sem parâmetro livre, fontes e músicas com licença registrada.

**Não é problema (e o relatório disse por quê):** dois `h1` (o segundo fica numa tela escondida), `server: Vercel` no header, `Disallow: /api/` no robots e `security.txt` ausente (opcional).

**Em aberto, sem detalhe:** itens de operação e de verificação na tela, nenhum de segurança. Entram numa próxima revisão.

## O que deu errado nas próprias skills

A rodada também testou as skills, e quatro coisas mudaram nelas antes da v0.1.0:

- **O script travou a própria auditoria.** Consultar caminhos sensíveis fez o anti-bot da Vercel bloquear o IP por alguns minutos, e o Lighthouse rodado depois disso falhou. Agora o script consulta esses caminhos por último, a resposta do anti-bot vira `bloqueado` (não verificado) em vez de falha, e na auditoria completa o Lighthouse roda antes do script.
- **O PageSpeed não respondeu.** A API sem chave usa uma cota anônima, que estava esgotada. A skill de site passou a usar o Lighthouse local quando não há chave.
- **`npm audit fix --force` teria piorado as coisas.** Ele propôs o downgrade do adaptador. A skill de segurança agora avisa pra não aplicar às cegas e pra ver se a falha roda em produção antes de classificar.
- **Escape lido não é escape testado.** No JSON-LD, o comentário garantia que o escape funcionava, e só rodando a linha dava pra ver que não. A skill de segurança agora manda testar todo escape feito à mão, com a mesma linha rodada no `node -e`.
