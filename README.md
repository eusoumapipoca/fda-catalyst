# FDA Catalyst Watch

Dashboard público da estratégia de position trading em catalisadores regulatórios FDA, integrado no [Confluens](https://confluens.carrd.co). Só percentagens; os valores em euros ficam no journal local.

## Ficheiros

| Ficheiro | Papel |
| --- | --- |
| `index.html` | Página estática. Lê `candidatos.json` em runtime; não precisa de build. |
| `candidatos.json` | Única fonte de verdade: candidatos, estados, janela, alertas. |
| `fda_scan.py` | Fases 0, 1 e 4 do funil (enumeração, triagem, monitorização). |
| `prompts/fase2_ficha_rapida.md` | Prompt curto: avança ou descarta um nome do Radar. |
| `prompts/fase3_analise_profunda.md` | Prompt completo (framework v2) para Advanced Research. |
| `radar/` | Saída semanal do script (`radar_AAAA-MM-DD.md` e `.json`). |

## Publicar em GitHub Pages (sem Git instalado)

1. Cria o repositório público `fda-catalyst` em github.com/eusoumapipoca.
2. Em "Add file → Upload files", sobe `index.html`, `candidatos.json` e `README.md`.
3. Em Settings → Pages, escolhe "Deploy from a branch", branch `main`, pasta `/ (root)`.
4. A página fica em `https://eusoumapipoca.github.io/fda-catalyst`. Aponta o Confluens para lá.

O `fda_scan.py`, os prompts e a pasta `radar/` ficam só na tua máquina (por exemplo `D:\Pessoal\FDA`); não precisam de estar no repositório.

## Ciclo semanal (segunda-feira)

1. `python fda_scan.py --monitor` — expira quem caiu abaixo de 6 meses e gera alertas em `candidatos.json`.
2. `python fda_scan.py` — Fases 0 e 1; escreve `radar/radar_<data>.md` com os tickers novos.
3. Para cada ticker do Radar, corre o prompt da **Fase 2** no chat. Descartados vão para `candidatos.json` com `"estado": "Descartado"` (uma linha basta; o script deixa de os apanhar).
4. Para quem avança, corre o prompt da **Fase 3** em Advanced Research. Cola o bloco JSON devolvido em `candidatos.json`.
5. Sobe o `candidatos.json` atualizado pela interface web do GitHub ("Add file → Upload files" substitui o existente). A página atualiza em 1–2 minutos.

## Configuração

- `pip install requests`
- Opcional: `setx FINNHUB_API_KEY "a_tua_chave"` para o teto de capitalização (a mesma chave do `earnings_runup.py`). Sem chave, o filtro é ignorado e o script avisa.
- Opcional: `setx EDGAR_USER_AGENT "Nome email@dominio.pt"` — a SEC pede um contacto nos pedidos.

## Regras da framework v2

Preço-base = fecho anterior aos dados. Negativo ≠ janela aberta (campo `recuperou`). Saída = PDUFA − 15 dias, assumindo +3 meses de deslize. CRL a ~30%. Expiração automática abaixo de 6 meses. Exclusões: subgrupo após primário falhado, queda sem recuperação, +60–70%, aquisição.
