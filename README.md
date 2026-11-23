# Nipli MCP Server

Polish company data for AI assistants: the KRS business register, financial statements, ownership and personal links, registry events, EU grants and public tenders.

Nipli runs a hosted, read-only [Model Context Protocol](https://modelcontextprotocol.io) server. There is nothing to install: add one URL to your MCP client and ask questions in plain language.

| | |
|---|---|
| **Endpoint** | `https://api.nipli.pl/mcp` |
| **Transport** | Streamable HTTP (stateless) |
| **Auth** | OAuth 2.1 (sign in with a Nipli account) or `Authorization: Bearer <API key>` |
| **Registry name** | `pl.nipli/nipli` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=nipli) |
| **Tools** | 21, all read-only |
| **Website** | [nipli.pl/mcp](https://nipli.pl/mcp) |

## What you can ask

- "Find the company with NIP 7740001454 and show its board and shareholders."
- "Which software companies in Małopolskie have revenue above 10 million PLN?"
- "Show revenue, net profit and margins of this company for all available years."
- "Who sits on boards together with this person, and in which companies?"
- "Which companies in my industry changed their management board this month?"
- "Who received EU funding in PKD 62.01.Z in 2025?"
- "List public tenders with CPV 45000000 published by this contracting authority."

## Coverage

- About 1.2 million entities from the National Court Register (KRS): companies, foundations, associations.
- About 4.6 million financial statements, with metrics for about 565 thousand companies.
- Sole proprietorships (JDG, from the CEIDG register) are **not** available. A search by the full NIP of a sole proprietor returns a single free result marked as unavailable.
- National identification numbers (PESEL) and dates of birth are never returned. People who objected under GDPR are hidden.

## Connect

### Claude (claude.ai and Claude Desktop)

Settings → Connectors → Add custom connector, then paste:

```
https://api.nipli.pl/mcp
```

Claude opens the Nipli sign-in page and asks for your consent. No key to copy.

### ChatGPT

Settings → Connectors → add a remote MCP server with the same URL. Sign in when prompted.

### Claude Code

```bash
claude mcp add --transport http nipli https://api.nipli.pl/mcp
```

Run `/mcp` inside Claude Code to finish the OAuth sign-in. To use an API key instead:

```bash
claude mcp add --transport http nipli https://api.nipli.pl/mcp --header "Authorization: Bearer YOUR_NIPLI_API_KEY"
```

### Cursor and other clients with a JSON config

```json
{
  "mcpServers": {
    "nipli": {
      "url": "https://api.nipli.pl/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_NIPLI_API_KEY"
      }
    }
  }
}
```

### VS Code

```json
{
  "servers": {
    "nipli": {
      "type": "http",
      "url": "https://api.nipli.pl/mcp"
    }
  }
}
```

Ready-to-copy files are in [`examples/`](examples).

## Authentication

Every request needs a token. A request without one gets `401` with a `WWW-Authenticate` header that points the client to the OAuth metadata, so OAuth-capable clients start the sign-in flow on their own.

- **OAuth 2.1** with PKCE (S256) and dynamic client registration. Metadata: [`/.well-known/oauth-protected-resource`](https://api.nipli.pl/.well-known/oauth-protected-resource) and [`/.well-known/oauth-authorization-server`](https://api.nipli.pl/.well-known/oauth-authorization-server).
- **API key** sent as `Authorization: Bearer <key>`. Create and rotate keys in the Nipli panel after you [sign up](https://nipli.pl/api). The same key works for the REST API.

## Tools

All tools only read data. Tool descriptions are in Polish; models handle them without any setup, and you can ask in any language.

### Companies

| Tool | What it returns | Cost |
|---|---|---|
| `search_companies` | Companies matching a name, NIP, KRS or REGON number | 1 token |
| `find_companies` | Companies filtered by industry (PKD), region and revenue. Returns a free cost estimate first; fetches only with `confirm=true` | 2 tokens per company |
| `get_company` | Profile: registry data, board and shareholders, capital links, document list, Nipli Score | by data group, 9–59 tokens |
| `get_financials` | Financial metrics for all available years | 10 tokens (latest year) or 25 (full history) |
| `get_ranking` | Ranking by `revenue`, `netProfit` or `assets` | 2 tokens per company |
| `get_industry` | Industry statistics for a PKD code: number of companies, median revenue, profitability, legal forms | 2 tokens per company |
| `bulk_companies` | Selected data groups for up to 25 KRS numbers at once. Free estimate first; fetches only with `confirm=true` | per returned result |

### Web presence and contacts

| Tool | What it returns | Cost |
|---|---|---|
| `get_company_web` | Website, social media, logo, description, Google Business details | 5 tokens |
| `get_company_tech` | Technologies detected on the company website | 10 tokens |
| `get_company_emails` | Company email addresses, deduplicated | 9 tokens per address |
| `get_company_phones` | Company phone numbers, deduplicated | 18 tokens per number |
| `get_company_contacts` | Named contacts: name, position, direct contact | 25 tokens per contact |
| `get_company_ubo` | Beneficial owners from the CRBR register: name, citizenship, country of residence, nature of control | 25 tokens per owner |

### People and links

| Tool | What it returns | Cost |
|---|---|---|
| `search_persons` | People listed in KRS (boards, supervisory boards, shareholders) by name | 1 token per person |
| `get_person` | All companies of a person with roles, portfolio summary, registry events, beneficial owner status | 25 tokens |
| `get_person_relations` | A person's companies and the people who sit in the same companies | 15 tokens |
| `get_company_persons` | People linked to a company with full names, including those from past court gazette notices | 1 token per person |

### Events, grants and tenders

| Tool | What it returns | Cost |
|---|---|---|
| `find_events` | Registry events across KRS: board, capital, address and PKD changes, registrations, restructurings, arrears. Filter by type, date, region, industry | 1 token per event |
| `get_company_events` | Timeline of registry events for one company | 1 token per event |
| `find_grants` | EU-funded projects by beneficiary, project title, programme, region, industry or value | 2 tokens per grant |
| `find_tenders` | Public procurement notices (BZP) by subject, CPV code, region or contracting authority NIP, with contractors when awarded | 2 tokens per notice |

## Pricing

Usage is billed in tokens, shared with the Nipli REST API. 1 token = 0.01 PLN. You pay only for data that was returned: an empty result costs nothing, and the expensive tools (`find_companies`, `bulk_companies`) show an estimate before they fetch.

Each response ends with the number of tokens used and the balance left, so the assistant can report the cost of an answer.

The first API key comes with a free trial: 100 requests within 30 days, no card required. Current plans: [nipli.pl/cennik](https://nipli.pl/cennik).

## Data sources

Public registers and open data: the National Court Register (KRS), the repository of financial documents (RDF), Monitor Sądowy i Gospodarczy (court and commercial gazette), the Central Register of Beneficial Owners (CRBR), the Public Procurement Bulletin (BZP), lists of EU-funded projects, and company websites.

## REST API

The MCP server is a thin layer over the Nipli REST API. If you prefer to call it directly:

- Documentation: [nipli.pl/api/dokumentacja](https://nipli.pl/api/dokumentacja)
- OpenAPI specification: [api.nipli.pl/api/docs-json](https://api.nipli.pl/api/docs-json)

## Status and support

- Health check: [`https://api.nipli.pl/mcp/health`](https://api.nipli.pl/mcp/health)
- Problems and requests: [open an issue](../../issues) in this repository.

## Terms and privacy

Use of the server is subject to the Nipli [terms of service](https://nipli.pl/regulamin) and [privacy policy](https://nipli.pl/polityka-prywatnosci).

The contents of this repository (documentation and configuration examples) are released under the [MIT License](LICENSE). The server itself is a hosted service and its source code is not part of this repository.

---

## Po polsku

Nipli udostępnia zdalny serwer MCP z danymi o polskich firmach z KRS: rejestr, sprawozdania finansowe, zarządy i wspólnicy, powiązania osób, zdarzenia rejestrowe, dotacje unijne i zamówienia publiczne. Serwer działa wyłącznie do odczytu i nie wymaga instalacji.

1. W Claude albo ChatGPT otwórz Ustawienia → Konektory i dodaj zdalny serwer MCP o adresie `https://api.nipli.pl/mcp`.
2. Zaloguj się kontem Nipli albo podaj klucz API w nagłówku `Authorization: Bearer`.
3. Zadaj pytanie zwykłym językiem, na przykład „Pokaż zarząd i wyniki finansowe spółki o NIP 7740001454”.

Zapytania rozliczane są w tokenach (1 token = 1 grosz), wspólnie z REST API, i tylko za dane, które wróciły. Pierwszy klucz API dostaje darmowy okres próbny: 100 zapytań przez 30 dni, bez karty. W bazie są wyłącznie podmioty z KRS; danych jednoosobowych działalności gospodarczych Nipli nie udostępnia.

Więcej: [nipli.pl/mcp](https://nipli.pl/mcp)
