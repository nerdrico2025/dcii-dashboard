# Página do painel semanal de GTM – DCTI-Institute

Página de uma tela para o Scott ler em 1 minuto. É um arquivo HTML único (HTML + CSS + JavaScript puro), sem frameworks, sem CDN, sem fontes externas e sem build. Ela não faz nenhuma requisição a outros domínios.

## Abrir localmente
O navegador bloqueia `fetch` em `file://`. Por isso a página precisa de um servidor local:

```bash
cd /Users/rafael/Documents/DCTI/dashboard/site
python3 -m http.server 8000
```

Abra **http://localhost:8000**. Para parar o servidor, use Ctrl+C.

- Idioma pela URL: `http://localhost:8000/?lang=pt-BR` ou `?lang=en-CA`.
- Semana pela URL: `?week=2026-10-26` (é o formato do `report_url` gravado pelo workflow B). Sem esse parâmetro, abre a mais recente. Se a semana não existir em `index.json`, abre a mais recente e mostra um aviso nos dois idiomas.
- A escolha de idioma fica salva no navegador (localStorage). Se o navegador bloquear o localStorage, a página funciona do mesmo jeito.
- Ordem de prioridade do idioma: `?lang` → escolha salva → `default_language` do JSON (en-CA).

## Estrutura
```
site/
├── index.html          página (tudo num arquivo só)
├── README_site.md      este arquivo
└── data/
    ├── index.json      lista de semanas, mais recente primeiro
    ├── latest.json     cópia da semana mais recente (reserva, caso index.json falhe)
    ├── 2026-10-26.json semana de EXEMPLO ("sample": true, dados fictícios)
    └── SCHEMA.md       contrato do JSON (o workflow de publicação deve segui-lo)
```

- Os dados de **2026-10-26** são **fictícios**. Enquanto `sample = true`, a página mostra o aviso "Sample data / Dados de exemplo".
- **Quem escreve em `data/`:** o workflow C do n8n, que grava `<week_start>.json`, `index.json` e `latest.json`, nessa ordem. Ele nunca grava o `index.html`, que é mantido por você no repositório.
- Na primeira publicação real, o workflow C tira a entrada de exemplo do `index.json` e substitui o `latest.json`. O arquivo `2026-10-26.json` de exemplo é sobrescrito se a semana real for a mesma; senão, apague-o do repositório à mão.
- **O que vem do JSON e o que é fixo na página:**
  - os pilares mostram os 8 itens de `kpi.pillars_all`, com valor formatado pela `unit` (EUR em Paid media);
  - o selo "Provisional / Provisório" vem de `kpi.funnel[].provisional`;
  - a página não tem regras de negócio fixas no código.

## Imprimir em PDF (o "PowerPoint de uma página")
1. Abra a página no idioma desejado. A impressão sai no idioma que está na tela.
2. Use **Arquivo › Imprimir** (Cmd+P) e escolha **Salvar como PDF**.
3. O CSS já define **A4 paisagem**, margens de 8 mm e uma única folha. O seletor de semana e o de idioma não aparecem na impressão.
4. No Chrome:
   - desmarque *Cabeçalhos e rodapés*, para não sair a URL;
   - deixe *Gráficos de fundo* marcado, para as cores dos selos saírem.

Testei no Chrome: os exemplos em en-CA e pt-BR couberam em 1 folha. Se uma semana tiver textos muito longos, use a escala de impressão em 90%.

## Acessibilidade e privacidade
- Status sempre com ícone + texto + cor. Contraste AA. Fonte do corpo com 16 px.
- O idioma é escolhido em botões operáveis por teclado, com foco visível e `aria-pressed`.
- A tabela do funil tem `<caption>` e `<th scope>`. No celular, ela rola na horizontal com a coluna de produto fixa.
- Respeita `prefers-reduced-motion`. A página não tem animações.
- `noindex, nofollow` e uma Content-Security-Policy que só permite carregar dados do próprio site.

## Testar rápido
- `?lang=pt-BR` e `?lang=en-CA`: todos os rótulos, datas (Intl) e valores em EUR mudam. Em en-CA, o status Gray aparece como "Grey".
- **Semana inexistente:** `?week=2030-01-07`. Deve abrir a mais recente, com o aviso amarelo nos dois idiomas.
- **Dado faltando:** apague um campo em uma cópia do JSON (ex.: `kpi.funnel[0].value = null`). A célula deve mostrar "No data yet". Um rótulo removido aparece como "—".
- **JSON quebrado:** a página mostra uma mensagem nos dois idiomas, nunca uma tela em branco.
