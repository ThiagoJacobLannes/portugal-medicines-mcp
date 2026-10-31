# portugal-medicines-mcp

MCP server that helps answer **"I need medicine X in city Y. What do I do?"** in Portugal, so Claude (or any MCP client) can say whether a medicine has a reported shortage, which pharmacies are on duty or open now, and who to call.

**Read-only, no key, no account.** It uses only public sources. Not affiliated with Infarmed, the SNS, SPMS, the Associação Nacional das Farmácias or farmaciasdeservico.net.

> **Pharmacy stock is not public in Portugal.** Only pharmacies and distributors can see it. This server never says a pharmacy has a medicine: call the pharmacy, or call **Linha 1400**, to confirm.
>
> **Not medical advice.** It is informational only. Never switch, stop or substitute a medicine without a pharmacist or doctor. In an emergency call **112**. For health advice call **SNS 24** on **808 24 24 24**.

## Tools

### `find_medicine_near`
The one-shot answer. Checks the medicine's availability, lists the pharmacies on duty today and the ones open now in the city (nearest first if you give a position), and returns the help lines and next steps.

Parameters: `medicine`, `city` (or `lat` / `lon`), `limit`.

### `availability`
Is a medicine in shortage? Searches by brand or active substance (Portuguese names work best: `"quetiapina"`, `"Ozempic"`, `"ácido acetilsalicílico"`; add a strength or form to narrow it). It checks three things Infarmed publishes on www.infarmed.pt:

- **shortages under monitoring**: the situations Infarmed is actively following, with its advice and circulars;
- **export temporarily suspended**: the monthly list of medicines whose export or distribution to other EU countries is suspended because of shortage or supply risk in Portugal;
- **prior-notification list (LNP)**: the quarterly list of medicines with availability concerns or critical medicines.

The latest lists are found automatically on Infarmed's availability page. `status` is one or more of `monitored_shortage`, `export_suspended`, `prior_notification`, `no_issue_in_published_lists`. "No issue" does not prove the medicine is available: see [What this server does not do](#what-this-server-does-not-do).

Parameters: `medicine`, `limit`.

### `pharmacies`
Pharmacies in a municipality (concelho), with address, phone, coordinates, today's hours, whether each one is **open now** and whether it is **on duty today** (official roster, duty day 09:00 to 09:00), with the duty type (for example "Permanente - Aberta 24h"). Sorted: on duty first, then open now, then distance.

Parameters: `city` (municipality, town or parish: `"Lisboa"`, `"Porto"`, `"Alvalade, Lisboa"`), `lat` / `lon` (adds `distance_km`; with no city, the municipality is found from the coordinate), `radius_km`, `on_duty_only`, `open_now_only`, `limit`. Only one municipality is covered per call.

### `where_to_get_help`
The help lines below, each with the page it was checked on.

## Help lines

| Line | Number | What for | Checked on |
|---|---|---|---|
| Emergency (INEM) | **112** | Risk to life or a serious situation | [gov.pt: A minha saúde](https://www.gov.pt/guias/a-minha-saude) |
| SNS 24 | **808 24 24 24** | Health advice when it is not an emergency | [gov.pt: A minha saúde](https://www.gov.pt/guias/a-minha-saude) |
| Linha 1400 | **1400** | Run by the pharmacies (ANF). Finds a pharmacy that can supply a medicine with reduced availability; also tells you the pharmacies on duty. 24 hours a day. Free from 19:00 to 09:00 and on weekends and national holidays; EUR 0.10/min from 09:00 to 19:00. Online form at [1400safe.pt](https://www.1400safe.pt/). | [1400safe.pt](https://www.1400safe.pt/) (the line's own site; not found on an SNS, Infarmed or gov.pt page) |
| Infarmed, Linha do Medicamento | **800 222 444** | Infarmed's medicines information centre (CIMI) | Contact block of [Infarmed circulars](https://www.infarmed.pt/documents/15786/12277909/Sistema+de+Pre%C3%A7os+de+Refer%C3%AAncia+-+3%C2%BA+Trimestre+de+2026+%28agosto%29/57ba41ea-aaea-1767-ad71-5afab2a50d27) (hours not stated) |

## Install

Requires Python 3.10+.

```bash
git clone https://github.com/ThiagoJacobLannes/portugal-medicines-mcp
cd portugal-medicines-mcp
python -m pip install -e .
```

On Windows, if you keep code in a cloud-synced folder (Google Drive, OneDrive), create the virtual environment **outside** it. Sync clients drop `desktop.ini` files into folders, which can crash packages on startup.

## Add to Claude Code

```bash
claude mcp add portugal-medicines -- python -m portugal_medicines_mcp.server
```

## Add to Claude Desktop

Merge `claude_desktop_config.snippet.json` into your Claude Desktop `claude_desktop_config.json`, then restart Claude Desktop:

```json
{
  "mcpServers": {
    "portugal-medicines": {
      "command": "python",
      "args": ["-m", "portugal_medicines_mcp.server"]
    }
  }
}
```

## Data sources

| Source | What it gives | Access | Terms |
|---|---|---|---|
| [Infarmed: Gestão da disponibilidade do medicamento](https://www.infarmed.pt/web/infarmed/gestao-da-disponibilidade-do-medicamento) | Shortages under monitoring; links to the lists in force | Public page; www.infarmed.pt's robots.txt allows crawling | No open-data licence stated on the page. Public-sector information; cite Infarmed as the source. |
| Infarmed export lists (.xlsx, linked from the page above) | Medicines with export temporarily suspended (monthly); prior-notification list (quarterly) | Public download | Same as above. |
| [farmaciasdeservico.net](https://www.farmaciasdeservico.net/) | Every pharmacy per municipality, with address, phone, map position, today's hours and the duty roster | Public pages; robots.txt allows all | Owned by 1000 Empresas, Lda. Its terms allow reproducing content **with the source cited**, and say the information may be incomplete or out of date. It also sells a paid webservice for commercial use; this server reads the public pages on demand for personal use. |
| [OpenStreetMap Nominatim](https://nominatim.openstreetmap.org/) | Turns a coordinate into a municipality (only when you give lat/lon without a city) | Public API, at most 1 request/second, identifying User-Agent | Data © OpenStreetMap contributors, [ODbL](https://www.openstreetmap.org/copyright). [Usage policy](https://operations.osmfoundation.org/policies/nominatim/). |

Data from these sources keeps the source's terms. The code in this repository is MIT-licensed.

## What this server does not do

- **No pharmacy stock.** It is not public, and this server does not try to get it.
- **No Infomed and no full shortage search.** Infarmed's medicine database ([Infomed](https://extranet.infarmed.pt/INFOMED-fo/)) and its per-medicine shortage and discontinuation search are on `extranet.infarmed.pt`, and that host's `robots.txt` says `Disallow: /` for all automated clients. The online Prontuário Terapêutico (`app10.infarmed.pt/prontuario/`) is disallowed too. So this server does **not** search medicines, list presentations, prices, reimbursement or generics, or look up the full shortage database. Use those sites in a browser. If Infarmed publishes an open dataset or allows automated access, these features can be added.
- No keys taken from other people's apps or web pages, and no bypassing of bot protection.

## How it works

The server uses `httpx` with one session, a User-Agent naming this project, and at most one request per second per site. It reads Infarmed's availability page and the two spreadsheets it links (re-read every 6 hours, so the new monthly list is picked up automatically), and the farmaciasdeservico.net pages for one municipality (two pages, cached for 15 minutes). Medicine names are matched without regard to accents or case. If a page or spreadsheet changes shape, the tool returns a clear `{"error": ...}` instead of guessing.

## License

MIT
