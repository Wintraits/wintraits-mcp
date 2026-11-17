# Wintraits MCP server

Wintraits tells you which attributes and combinations the products that succeed on Amazon share, category by category, and checks a product, a listing or a product idea against them. It works as a remote MCP server, so any MCP client (Claude, ChatGPT, VS Code, Cursor, n8n…) can call it.

**Not only for Amazon sellers.** The data comes from Amazon because its sales ranking is public, but the winning attributes describe what buyers choose in a category (ingredients, format, size, claims, certifications, pack, price band). Use them to design or source a product, brief a manufacturer or agency, set price and format, or write the product page of your own shop (Shopify, WooCommerce…), another marketplace, a catalogue or the packaging. Each trait says whether it is about the **product** (travels best to other channels), the **presentation** online or the **store**. Outside Amazon it is a reference, not proof.

- **Endpoint:** `https://wintraits.com/mcp` (streamable HTTP)
- **Key:** optional (`Authorization: Bearer <key>`, free from https://wintraits.com/agentes). **Open beta: every tool is free and unlimited until 31 January 2027**; anyone who creates a key during the beta gets founder pricing (−50 % for 12 months). Technical limits during the beta: 60 calls/day without a key, 600 with one.
- **Website:** https://wintraits.com · beta: https://wintraits.com/beta · methodology: https://wintraits.com/metodologia

## Coverage

- **Scan** (what today's best sellers have, trait by trait): 25 categories (beauty, personal care, small appliances, electronics, home, books, board games, puzzles, perfumes…) on Amazon Spain, France, Germany and Italy, with new categories every week. `wintraits_buscar_categoria` finds a category from plain words; `wintraits_categorias_disponibles` lists them all with their data date.
- **Match** (which launches succeed, followed from day one for six months): five beauty categories on Amazon Spain (face serums, moisturisers, eye contour, sunscreens and facial cleansers). Sealed exam for serums and sunscreens on 27 December 2026: rules fixed in advance, result published either way. Until then Match is descriptive, not validated.

## What's new

- Find a category from plain words in English or Spanish (`wintraits_buscar_categoria`) and the top 10 best sellers of a Scan category by index (`wintraits_ranking`).
- Match now covers five beauty categories on Amazon Spain.
- Many ASINs at once, for agencies and catalogues: REST API `POST https://wintraits.com/api/lote` (see below) and, with no coding, https://wintraits.com/varios-asin with CSV download.
- Winning attributes of any Scan category without pasting a listing (`wintraits_scan_perfil`) and how the category has moved (`wintraits_scan_historia`).
- Wintraits Scan Index split into product and presentation; a part that cannot be computed comes back empty with its reason, never 0.
- Check from a technical sheet with no online listing (`ficha_tecnica`), from an ASIN, and weekly follow-up of a listing.
- Every `cita` now has `texto_para_citar`, ready to quote.

## Tools

| Tool | What it does |
|---|---|
| `wintraits_categorias_disponibles` | Categories and countries studied. |
| `wintraits_buscar_categoria` | Finds the category code from plain words ("air fryer", "board games"). Start here. |
| `wintraits_ranking` | Scan: top 10 best sellers of a category by Wintraits Scan Index. |
| `wintraits_scan_perfil` | Scan: every trait that sets today's best sellers apart, with %, type and medians. No listing needed. |
| `wintraits_scan_historia` | Scan: how the category's Wintraits Scan Index has moved over time. |
| `wintraits_chequeo_rapido` | Scan: whether your listing (or technical sheet, or draft) has, trait by trait, what today's best sellers in its category have. |
| `wintraits_chequeo_asin` | Scan: the same check from just the ASIN or Amazon link; detects the category. |
| `wintraits_seguir_ficha` | Scan: weekly follow-up of a listing (free key; only the ASIN is stored). |
| `wintraits_atributos_ganadores` | Attributes and combinations the winners have, before writing or improving a listing. |
| `wintraits_diagnosticar_producto` | Wintraits Match Index of a specific listing and what it lacks. |
| `wintraits_comparar_fichas` | Which of two listings looks more like the winners, attribute by attribute. |
| `wintraits_panorama_categoria` | How many launches succeed, winning price range, gaps, traps and trends. |
| `wintraits_indice_categoria` | Out of 100 launches, how many succeed, by quarter. |
| `wintraits_opciones_concepto` | Attributes you can set when evaluating a concept. |
| `wintraits_evaluar_concepto` | Wintraits Match Index of a product before manufacturing it. |
| `wintraits_hallazgos` | Validated findings, checked against later launches. |
| `wintraits_evidencia` | The counts behind the findings. |
| `wintraits_opinion` | Rate a report (with its `id_informe`). |

Every answer includes a `cita` (URL, date and `texto_para_citar`): quote it with its link. These are statistical patterns (correlation, not causation): only change what is true of the product. Category data can be quoted with attribution; bulk download is not allowed. Apps or services that show Wintraits data to their own users: get a key at https://wintraits.com/agentes and show «Datos: Wintraits» with a link.

## REST API and many ASINs at once

The same data is available as JSON under `https://wintraits.com/api/` (same optional key and limits as the MCP; `?lang=en` for English). Full list of requests: https://wintraits.com/agentes#api.

MCP tools check one listing per call. For a catalogue:

```bash
curl -X POST https://wintraits.com/api/lote -H "Authorization: Bearer <key>" \
  -H "Content-Type: application/json" -d '{"asins": ["B0XXXXXXXX", "https://www.amazon.es/dp/B0YYYYYYYY"], "pais": "ES"}'
# -> 202 {"id": "..."}
curl https://wintraits.com/api/lote/<id>       # progress and one row per listing
curl https://wintraits.com/api/lote/<id>.csv   # the same table as CSV
```

Listings are analysed in turn (a few minutes); results are kept for 30 days; one running batch per account.

## Setup

VS Code (`.vscode/mcp.json`):

```json
{
  "servers": {
    "wintraits": {
      "type": "http",
      "url": "https://wintraits.com/mcp",
      "headers": { "Authorization": "Bearer ${input:wintraits_key}" }
    }
  },
  "inputs": [
    { "id": "wintraits_key", "type": "promptString", "description": "Wintraits key (optional)", "password": true }
  ]
}
```

Claude (claude.ai or Desktop): Settings → Connectors → Add custom connector → `https://wintraits.com/mcp`.

ChatGPT (developer mode): Settings → Connectors → Create → URL `https://wintraits.com/mcp`, no authentication.
Cline: MCP Servers → Configure → add to `cline_mcp_settings.json` (nothing to install or build; the header is optional):

```json
{
  "mcpServers": {
    "wintraits": {
      "type": "streamableHttp",
      "url": "https://wintraits.com/mcp",
      "headers": { "Authorization": "Bearer <key>" },
      "disabled": false
    }
  }
}
```
Any other client: add a remote server with URL `https://wintraits.com/mcp`; optionally the header `Authorization: Bearer <key>`.

---

## En español

Wintraits dice qué atributos y combinaciones tienen los productos que triunfan en Amazon, categoría a categoría, y revisa un producto, una ficha o una idea frente a ellos. Sirve también fuera de Amazon: para diseñar el producto, el encargo al fabricante, el envase o la ficha de tu tienda o de otro marketplace (es una referencia, no una prueba). Es un servidor MCP remoto: `https://wintraits.com/mcp`. Beta abierta: todo gratis y sin límite hasta el 31-ene-2027 (clave gratis en https://wintraits.com/agentes, con precio fundador). Scan: qué tienen hoy los más vendidos en 25 categorías (belleza, cuidado personal, pequeño electrodoméstico, electrónica, hogar, libros, juegos, puzles…) de Amazon España, Francia, Alemania e Italia, con los atributos ganadores de cada categoría sin pegar ninguna ficha (`wintraits_scan_perfil`); busca la categoría con palabras (`wintraits_buscar_categoria`). Match: lanzamientos seguidos desde el primer día en cinco categorías de belleza de Amazon España (sérums, hidratantes, contorno de ojos, solares y limpiadores); examen sellado de sérums y solares el 27-dic-2026, con el resultado publicado salga como salga. Son patrones estadísticos (correlación, no causalidad); cítalos con `cita.texto_para_citar`. Novedad: varios ASIN a la vez, para agencias y catálogos, con la API REST (`POST https://wintraits.com/api/lote`, documentada en https://wintraits.com/agentes#api) o sin programar en https://wintraits.com/varios-asin (descarga en CSV).
