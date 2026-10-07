# Deploy na Vercel — Dash "Elton - Seguidores"

Guia para publicar o dashboard na Vercel como site estático.

O dash é **um único arquivo** (`index.html`): HTML, CSS e JavaScript juntos. Ele não tem build, backend, banco de dados próprio nem variáveis de ambiente.

---

## 1. Antes de começar

### Estrutura da pasta

```
elton-seguidores/
├── index.html        ← o dashboard (obrigatório)
├── vercel.json       ← opcional, recomendado (ver seção 3)
├── DEPLOY-VERCEL.md  ← este guia
└── CONTEXTO.md       ← documentação do dash
```

### A planilha precisa estar pública

O navegador de quem abre o dash lê a planilha direto do Google, pelo endpoint `gviz/tq?tqx=out:csv`. Para isso funcionar, a planilha precisa estar compartilhada assim:

1. Abra a planilha (`16qNO3FVA-cUI2WwK_UBkWhwx8MQUhi_lgSB-307_jY0`).
2. Clique em **Compartilhar → Acesso geral → "Qualquer pessoa com o link" → Leitor**.

Se ela ficar restrita, o dash mostra o erro *"erro 4xx na aba … (a planilha está pública?)"*.

### Teste local

1. Abra o `index.html` direto no navegador (duplo clique).
2. Se carregar os números da aba **"Base Seguidores Elton"**, está pronto para subir.

---

## 2. Opções de deploy

### Opção A — Pelo GitHub (recomendada, com deploy automático)

1. Crie um repositório no GitHub (ex.: `dash-elton-seguidores`) e suba o `index.html`. Suba também o `vercel.json`, se for usar.
2. Na Vercel, clique em **Add New… → Project → Import Git Repository** e escolha o repositório.
3. Configure o projeto:

   | Campo | Valor |
   |---|---|
   | Framework Preset | **Other** |
   | Root Directory | `./`, ou a subpasta onde está o `index.html` |
   | Build Command | *(vazio; desative o override)* |
   | Output Directory | *(vazio, ou `.`)* |
   | Install Command | *(vazio)* |
   | Environment Variables | nenhuma |

4. Clique em **Deploy**.
5. Daqui em diante, cada `git push` na branch principal publica a nova versão sozinho.

### Opção B — Pela linha de comando (Vercel CLI)

Use quando quiser publicar sem passar pelo GitHub.

```bash
npm i -g vercel          # instala a CLI (uma vez)
cd elton-seguidores      # pasta com o index.html
vercel login             # entra na conta
vercel                   # cria o projeto e gera uma URL de preview
vercel --prod            # publica em produção
```

Respostas para as perguntas da CLI na primeira vez:

| Pergunta | Resposta |
|---|---|
| Set up and deploy? | `Y` |
| Link to existing project? | `N` |
| Project name | `elton-seguidores` |
| In which directory is your code located? | `./` |
| Override settings? | `N` (a Vercel detecta como site estático) |

### Opção C — Mesmo projeto do dash TMB (subpasta)

Use quando o dash TMB já está na Vercel e você quer o Elton no mesmo domínio.

1. Coloque o arquivo em uma subpasta do repositório do TMB, com este caminho: `elton-seguidores/index.html`.
2. Faça o push.
3. O dash fica acessível em `https://<dominio-do-tmb>/elton-seguidores/`.

> Use a barra final na URL (`/elton-seguidores/`).

---

## 3. `vercel.json` (opcional, recomendado)

Esse arquivo evita que a Vercel ou o navegador guardem uma versão antiga do `index.html` em cache depois de uma atualização. Os **dados** não precisam disso, porque o dash já busca a planilha sem cache a cada carregamento.

```json
{
  "cleanUrls": true,
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=0, must-revalidate" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
      ]
    }
  ]
}
```

> **Não** adicione um `Content-Security-Policy` restritivo. O dash precisa acessar dois serviços externos:
> - `docs.google.com`, para os dados;
> - `fonts.googleapis.com` / `fonts.gstatic.com`, para as fontes.

---

## 4. Domínio próprio (opcional)

1. No projeto da Vercel, abra **Settings → Domains → Add**.
2. Informe o domínio, por exemplo `elton.seudominio.com.br`.
3. No provedor de DNS, crie o registro que a Vercel indicar (normalmente um `CNAME` apontando para `cname.vercel-dns.com`).
4. Aguarde a propagação. O HTTPS é emitido automaticamente.

---

## 5. Proteger o acesso (opcional)

O dash não tem login. Qualquer pessoa com a URL vê os números.

Na Vercel, dá para limitar o acesso em **Settings → Deployment Protection**: Vercel Authentication ou Password Protection, conforme o plano.

> A planilha continua pública via link. Proteger o site **não** esconde a planilha de quem tiver o ID dela.

---

## 6. Atualizar o dash

| Situação | O que fazer |
|---|---|
| Mudou só a planilha (novos dias, novas campanhas) | Nada. O dash lê ao vivo e atualiza sozinho a cada 60 min ou no botão "↻ Atualizar". |
| Mudou o código (`index.html`) | `git push` (Opção A) ou `vercel --prod` (Opção B). |
| Trocou o nome da aba ou da planilha | Edite `SHEET_CFG` no topo do `<script>` e publique de novo. |
| Quer ativar metas | Preencha `METAS_SEG` no topo do `<script>` e publique de novo. |
| Mudar o valor padrão do Planejamento | Edite `PLAN_DEFAULT` e publique de novo. |

---

## 7. Checklist pós-deploy

- [ ] A URL abre e mostra o selo **"● Google Sheets"** em verde no canto do cabeçalho.
- [ ] A Principal mostra Investimento, Visitas, Seguidores e custos com valores.
- [ ] Não aparece o aviso amarelo de colunas ausentes em **Métricas tráfego**. Se aparecer, faltam colunas brutas na aba (ver `CONTEXTO.md`).
- [ ] **Resumo diário → Copiar resumo** copia o texto com o título `*Elton - Distribuição*`.
- [ ] O Planejamento mostra R$ 7.500,00 como investimento do mês.
- [ ] O layout funciona no celular.

---

## 8. Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| "Erro ao carregar dados: erro 4xx…" | Planilha não está pública | Compartilhar como "Qualquer pessoa com o link → Leitor". |
| "aba … sem a coluna 'Nome da campanha'" | Nome da aba diferente, e o Google devolve a 1ª aba | Conferir o nome exato da aba e o `SHEET_CFG.tab`. |
| CTR, CPC, Hook e Hold aparecem como "—" | Colunas brutas ausentes na aba | Adicionar as colunas (ver `CONTEXTO.md`, seção "Colunas"). |
| Valores muito altos ou estranhos | Coluna formatada como texto ou data no Sheets | Formatar a coluna como **Número** na planilha. |
| Versão antiga aparece após deploy | Cache do navegador | Ctrl+F5; usar o `vercel.json` da seção 3. |
| 404 na subpasta (Opção C) | URL sem barra final ou pasta com outro nome | Acessar `/elton-seguidores/` e conferir o caminho do arquivo. |
