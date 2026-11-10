# Thailand Company Registry & Government Tenders MCP

Connect your AI assistant to Thai company registry and government procurement data through [The Company Atlas](https://thecompanyatlas.com) and the Model Context Protocol.

Company data covers about 1.8 million juristic persons registered with the Department of Business Development (DBD). Tender data is sourced from the Thai Government Procurement system, e-GP ([gprocurement.go.th](https://www.gprocurement.go.th)).

Search:

- Registered companies by English name, entity type, TSIC industry code and registration status
- Company profiles with registered capital, latest reported revenue, profit, assets and equity, and tender history. The 13-digit juristic person id is also the bidder id in tender records, so a company links to the tenders it bid on and won
- Government suppliers by Thai or English name
- Procurement projects by Thai project name, buying agency, bidder, province, budget and announcement date
- Lots, bidders, proposed and agreed prices, and winners

The MCP connection is provided by The Company Atlas. This repository does not run a separate server.

## Connect

```text
https://thecompanyatlas.com/mcp/thailand
```

Add this URL as a custom MCP connector in Claude, Cursor, or another MCP-compatible client. Sign in with Google when prompted.

- Transport: Streamable HTTP
- Auth: Google OAuth
- Docs: [thecompanyatlas.com/tools/thailand-company-tenders](https://thecompanyatlas.com/tools/thailand-company-tenders)

## ภาษาไทย

Thailand Company Registry & Government Tenders MCP ช่วยให้ผู้ช่วย AI ค้นหาข้อมูลนิติบุคคลไทยจากกรมพัฒนาธุรกิจการค้า และข้อมูลการจัดซื้อจัดจ้างภาครัฐจากระบบ e-GP

คุณสามารถ:

- ค้นหานิติบุคคลตามชื่อภาษาอังกฤษ ประเภทนิติบุคคล รหัสประเภทธุรกิจ (TSIC) และสถานะ
- ดูข้อมูลบริษัท ทุนจดทะเบียน รายได้ กำไร สินทรัพย์ ส่วนของผู้ถือหุ้น และประวัติการเสนอราคาและชนะการประมูล
- ค้นหาผู้ค้ากับภาครัฐตามชื่อภาษาไทยหรือภาษาอังกฤษ
- ค้นหาโครงการจัดซื้อจัดจ้างตามชื่อโครงการ หน่วยงาน ผู้เสนอราคา จังหวัด วงเงินงบประมาณ และวันที่ประกาศ
- ดูรายชื่อผู้เสนอราคา ราคาที่เสนอ ราคาที่ตกลงซื้อหรือจ้าง และผู้ชนะ

เพิ่ม URL นี้ใน Claude, Cursor หรือแอปที่รองรับ MCP:

```text
https://thecompanyatlas.com/mcp/thailand
```

ลงชื่อเข้าใช้ด้วย Google เมื่อระบบขอ จากนั้นลองถาม:

- *"ค้นหาบริษัท ดีเคเอสเอช (ประเทศไทย) จำกัด และงานจัดซื้อจัดจ้างภาครัฐที่ชนะ"*
- *"โครงการของกรมทางหลวงที่มีวงเงินเกิน 10 ล้านบาท เรียงจากมากไปน้อย"*
- *"ใครเสนอราคาในโครงการ 67049078583 และใครชนะในราคาเท่าไร"*

## Tools

### `Thailand_search_companies`

Search Thai registered companies. Filters are combined with AND logic; include at least one.

| Parameter | Type | Description |
|---|---|---|
| `q` | string, optional | English name substring, 3–120 characters |
| `entityType` | enum, optional | `Company Limited`, `Public Company Limited`, `Ordinary Partnership`, `Juristic Ordinary Partnership`, `Trade of Association` |
| `tsic` | string, optional | TSIC industry code prefix, 2–5 digits, e.g. `41` |
| `status` | enum, optional | Registration status, e.g. `Operating`, `Defunct`, `Liquidation Completion`, `Dissolution`, `Bankrupt` |
| `active` | boolean, optional | Filter by active status |
| `page` / `limit` | integer, optional | Pagination, `limit` up to 50 (default 20) |

Returns `{ data, total, page, limit }`. The company `id` is `TH` + the 13-digit juristic person id.

### `Thailand_get_company`

Full profile by 13-digit juristic person id (e.g. `0105523002118`, with or without the `TH` prefix).

| Parameter | Type | Description |
|---|---|---|
| `id` | string | Juristic person id / tax id |

Returns `registry` (DBD record plus latest reported revenue, profit, assets and equity in THB) and `tenders`: number of lots bid on, number won, total won value and the 20 most recent bids.

### `Thailand_search_bidders`

Search government suppliers by Thai or English name, ordered by total procurement value.

| Parameter | Type | Description |
|---|---|---|
| `query` | string | Name substring, Thai or English |
| `limit` | integer, optional | Up to 50 (default 20) |

### `Thailand_list_tenders`

Search Thai government procurement projects. Filters are combined with AND logic.

| Parameter | Type | Description |
|---|---|---|
| `query` | string, optional | Thai project name or English translation substring |
| `agency` | string, optional | Buying agency name, Thai or English, e.g. `Highways` |
| `deptId` | string, optional | Exact agency id |
| `bidderTin` | string, optional | 13-digit juristic person id of a bidder |
| `wonOnly` | boolean, optional | With `bidderTin`, only tenders the bidder won |
| `provinceMoiId` | string, optional | 6-digit Ministry of Interior province code, e.g. `100000` for Bangkok |
| `projectCost` | enum, optional | Evaluation basis: `C` price only, `P` price and quality, `Q` minimum quality |
| `minBudget` | number, optional | Minimum budget in THB |
| `announcedFrom` / `announcedTo` | date, optional | Announcement date range, `YYYY-MM-DD` |
| `sortBy` | enum, optional | `announceDate` (default), `projectMoney`, `priceAgree` |
| `page` / `limit` | integer, optional | Pagination, `limit` up to 100 (default 20) |

Returns `{ data, total, page, limit }`. Amounts are Thai baht.

### `Thailand_get_tender`

One procurement project by id (e.g. `67049078583`) with its lots, every bidder with proposed price and qualification result, the winner's agreed price, contracts, agency and location.

## Example prompts

- *"Find operating public companies in building construction (TSIC 41)."*
- *"Show the financials and government tender wins for DKSH Thailand, 0105523002118."*
- *"Department of Highways tenders over 10 million baht, largest first."*
- *"Who bid on project 67049078583, and who won at what price?"*

## Setup — Claude

1. Go to [claude.ai/customize/connectors](https://claude.ai/customize/connectors)
2. Click **Add custom connector**
3. Name it **Thailand Companies & Tenders** and paste `https://thecompanyatlas.com/mcp/thailand`
4. Save and sign in with Google when prompted
