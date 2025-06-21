# Airtable Base – Architecture Blueprint

> **Base ID:** `appfKG8NbAlew52Kf`  ·  **Version 0.2**  ·  **Updated:** 19 Jun 2025  ·  **Owner:** Mart
> This doc lives alongside `README.md` and tracks ONLY the Airtable data‑model. All future tweaks go here.

---

## 1 · Purpose

Design a clean, scalable schema that powers the Source Ventures automation stack for at least the next 3 years (<50 k records). Optimised for:

* Easy manual edits by team (Airtable UI)
* Robust API writes (single bot)
* Seamless upgrade path to Postgres if needed

---

## 2 · Core Tables & Fields

<!-- Table: Companies -->

### 2.1 Companies

#### Field: slug
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: `LOWER(SUBSTITUTE({name}, " ", "-"))`
- **Notes**: Unique identifier (ID) for the company. Auto-generated from name.

#### Field: logo
- **Type**: Attachment
- **Required**: No
- **Notes**: Company logo (image, optional).

#### Field: name
- **Type**: Single line text
- **Required**: Yes
- **Notes**: Full name of the company.

#### Field: companyType
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Startup
  - Holding
  - Management Company
- **Notes**: Type of company.

#### Field: sector
- **Type**: Single-select
- **Required**: No
- **Options**:
  - SaaS
  - Fintech
  - Climate/Greentech
  - Health/Biotech
  - Marketplace
  - Deeptech/AI
  - Consumer
  - Mobility
  - Food/Agri
  - Proptech
  - Edtech
  - E-commerce
  - HR/Future of Work
  - Defense
- **Notes**: Only for startups.

#### Field: founded
- **Type**: Date
- **Required**: No
- **Notes**: Year of incorporation. Cannot be in the future.

#### Field: status
- **Type**: Single-select
- **Required**: No
- **Options**: Active, Exited, Dead
- **Notes**: Only for startups.

#### Field: tags
- **Type**: Multi-select
- **Required**: No
- **Notes**: Free tags for filtering/search.

#### Field: files
- **Type**: Link to Files
- **Required**: No
- **Notes**: All files related to the company.

---

<!-- Table: Lines -->

### 2.2 Lines

#### Field: id
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: `LOWER(CONCATENATE({company}, "-", {type}, "-", {round}, "-", {spv}))`
- **Notes**: Unique identifier, concatenates company, type, round, spv.

#### Field: company
- **Type**: Link to Companies
- **Required**: Yes
- **Notes**: Linked company (startup or other).

#### Field: type
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Equity
  - SAFE
  - Convertible
  - Loan
  - BSA Air
  - Bond
  - Other
- **Notes**: Type of investment.

#### Field: round
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Pre-seed
  - Seed
  - Bridge
  - Series A
  - Series B
  - Series C
- **Notes**: Investment round.

#### Field: status
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Open
  - Exited
  - Dead
  - Converted
- **Notes**: 
  - 'Open': active line, investment ongoing
  - 'Exited': exit, sale, or repayment
  - 'Dead': total loss, company liquidated
  - 'Converted': for SAFE/BSAIR converted to equity

#### Field: spv
- **Type**: Link to Funds/SPVs
- **Required**: Yes
- **Notes**: Investment vehicle used.

#### Field: currency
- **Type**: Single-select
- **Required**: Yes
- **Options**: EUR, USD
- **Notes**: Currency of the investment line.

#### Field: date
- **Type**: Date
- **Required**: Yes
- **Notes**: Closing date.

#### Field: amountRaised
- **Type**: Number
- **Required**: No
- **Notes**: Total amount raised in the round (in currency of the line).

#### Field: postMoney
- **Type**: Number
- **Required**: No
- **Notes**: Post-money valuation (in currency of the line).

#### Field: svTicket
- **Type**: Number
- **Required**: No
- **Notes**: Amount invested by Source Ventures (in currency of the line).

#### Field: docsUrl
- **Type**: Attachment
- **Required**: No
- **Notes**: Attachment (PDF, SHA-named).

#### Equity fields (for type = 'Equity')

##### Field: nbSharesSubscribed
- **Type**: Number
- **Required**: Yes (input)
- **Notes**: Number of shares subscribed (from Subscription Form).

##### Field: sharePrice
- **Type**: Number
- **Required**: Yes (input)
- **Notes**: Price per share (in currency of the line).

##### Field: preMoneyShares
- **Type**: Number
- **Required**: Yes (input)
- **Notes**: Number of shares before the round (from latest Captable).

##### Field: newSharesFromRound
- **Type**: Number
- **Required**: Yes (input)
- **Notes**: Number of new shares created in the round (from Captable).

##### Field: roundSize
- **Type**: Formula (calculated)
- **Formula**: `newSharesFromRound * sharePrice`
- **Notes**: Total amount raised in the round (in currency of the line).

##### Field: investmentAmount
- **Type**: Formula (calculated)
- **Formula**: `nbSharesSubscribed * sharePrice`
- **Notes**: Exact amount invested by Source Ventures (in currency of the line).

##### Field: postMoneyValuation
- **Type**: Formula (calculated)
- **Formula**: `(preMoneyShares + newSharesFromRound) * sharePrice`
- **Notes**: Post-money valuation of the company (in currency of the line).

##### Field: ownership
- **Type**: Formula (calculated)
- **Formula**: `nbSharesSubscribed / (preMoneyShares + newSharesFromRound)`
- **Notes**: Ownership percentage (non-diluted) after the round.

#### SAFE/BSA-AIR fields (for type = 'SAFE' or 'BSA Air')

##### Field: safeAmount
- **Type**: Number
- **Required**: Yes (input)
- **Notes**: Amount invested (in currency of the line).

##### Field: cap
- **Type**: Number
- **Required**: No (input)
- **Notes**: Valuation cap (in currency of the line).

##### Field: discount
- **Type**: Percent
- **Required**: No (input)
- **Notes**: Discount rate (from contract, optional).

##### Field: floor
- **Type**: Number
- **Required**: No (input)
- **Notes**: Valuation floor (in currency of the line).

##### Field: bsaExercisePrice
- **Type**: Number
- **Required**: No (input, BSA-AIR only)
- **Notes**: Exercise price for BSA-AIR (in currency of the line).

##### Field: expirationDate
- **Type**: Date
- **Required**: No (input, BSA-AIR only)
- **Notes**: Expiration date for BSA-AIR (rarely used in practice).

##### Field: conversionValue
- **Type**: Number
- **Required**: No (input, at conversion)
- **Notes**: Value at which the SAFE/BSA-AIR is converted (in currency of the line).

##### Field: conversionShares
- **Type**: Formula (calculated)
- **Formula**: `conversionValue / sharePrice` (at conversion)
- **Notes**: Number of shares received at conversion.

##### Field: ownership (post-conversion)
- **Type**: Formula (calculated)
- **Formula**: `conversionShares / (preMoneyShares + newSharesFromRound)`
- **Notes**: Ownership percentage after conversion.

---

<!-- Table: Files -->

### 2.3 Files

#### Field: id
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: `LOWER(CONCATENATE({company}, "-", {category}, "-", {subCategory}, "-", {year}))`
- **Notes**: Unique identifier for the file.

#### Field: company
- **Type**: Link to Companies
- **Required**: Yes
- **Notes**: Linked company (primary relationship).

#### Field: file
- **Type**: Attachment
- **Required**: Yes
- **Notes**: The actual file (PDF, image, etc.).

#### Field: category
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Investment (Equity, Equity Markup)
  - Investment (SAFE)
  - Investment (BSA-AIR)
  - General Meeting
  - Financial Statement
  - Markup
  - Reporting
- **Notes**: Main category of the file.

#### Field: subCategory
- **Type**: Single-select
- **Required**: No
- **Options** (by category):
  - **Investment (Equity, Equity Markup)**
    - KBIS startup
    - Statuts startup
    - Captable
    - Bon de souscription
    - Pacte d'actionnaires signé
    - Adhésion au pacte d'actionnaires / mini-pacte
    - KBIS SPV Source Ventures
    - Statuts SPV Source Ventures signés
    - RBE SPV Source Ventures
    - RIB d'AK (optionnel)
  - **Investment (SAFE)**
    - Certificate of Good Standing / Existence / Authorization
    - SAFE contract
    - KBIS SPV Source Ventures
    - Statuts SPV Source Ventures signés
    - RBE SPV Source Ventures
    - Payment instruction
  - **Investment (BSA-AIR)**
    - KBIS startup
    - Statuts startup
    - BSA-AIR contract
    - KBIS SPV Source Ventures
    - Statuts SPV Source Ventures signés
    - RBE SPV Source Ventures
    - RIB startup (optionnel)
  - **General Meeting**
    - Convocation AG
    - Procès-verbal AG
    - Mandat de représentation
    - Feuille de présence
    - Résolutions
    - Rapport de gestion
    - Rapport du commissaire aux comptes
    - Liste des actionnaires
    - Pouvoir
    - Autre (AG)
  - **Financial Statement**
    - Bilan
    - Compte de résultat
    - Liasse fiscale
    - Rapport de gestion
    - Synthèse financière
    - Autre (Financial Statement)
  - **Markup**
    - Term sheet
    - Shareholders agreement (markup)
    - Board minutes
    - New cap table
    - Press release
    - Autre (Markup)
  - **Reporting**
    - PDF
    - Email (.aml)
    - Other (Reporting)
- **Notes**: More precise type of file (optional, especially for investment files and reportings).

#### Field: year
- **Type**: Single-select
- **Required**: No
- **Options**: 2020, 2021, 2022, 2023, 2024, 2025, 2026, 2027, 2028, 2029, 2030
- **Notes**: Year of the file (for organization and filtering).

#### Field: companyLogo
- **Type**: Lookup
- **Required**: No
- **Notes**: Lookup to company logo for display purposes.

---

<!-- Table: Funds/SPVs -->

### 2.4 Funds / SPVs

#### Field: name
- **Type**: Text
- **Required**: Yes
- **Notes**: Name of the fund or SPV.

#### Field: fundType
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - SPV
  - Holding
  - Management Company
- **Notes**: Type of fund or vehicle.

#### Field: owner
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - Source Ventures
  - Source Ventures 2
  - Martin Charpentier
  - Equisafe
  - Maresquier Partner
  - Roundtable
  - Other
- **Notes**: Who owns the fund/SPV.

#### Field: manager
- **Type**: Single-select
- **Required**: No
- **Options**:
  - Source Ventures
  - Source Ventures 2
  - Martin Charpentier
  - Equisafe
  - Maresquier Partner
  - Roundtable
  - Other
- **Notes**: Who manages the fund/SPV.

#### Field: externalManager
- **Type**: Text
- **Required**: No
- **Notes**: If managed externally, specify the manager.

#### Field: label
- **Type**: Formula or Text
- **Required**: No
- **Notes**: Display name for the fund/SPV.

#### Field: currency
- **Type**: Single-select
- **Required**: Yes
- **Options**: EUR, USD
- **Notes**: Currency of the fund/SPV.

---

<!-- Table: LPs -->

### 2.5 LPs

#### Field: id
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: CONCATENATE({name}, " ", {lastName})
- **Notes**: Unique identifier for the LP (full name).

#### Field: profilePicture
- **Type**: Attachment
- **Required**: No
- **Notes**: LP profile photo.

#### Field: name
- **Type**: Single line text
- **Required**: Yes
- **Notes**: First name.

#### Field: lastName
- **Type**: Single line text
- **Required**: Yes
- **Notes**: Last name.

#### Field: secondNames
- **Type**: Single line text
- **Required**: No
- **Notes**: Other given names.

#### Field: nationality
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - 🇫🇷 France
  - 🇧🇪 Belgium
  - 🇨🇭 Switzerland
  - 🇩🇪 Germany
  - 🇪🇸 Spain
  - 🇮🇹 Italy
  - 🇳🇱 Netherlands
  - 🇱🇺 Luxembourg
  - 🇦🇹 Austria
  - 🇵🇹 Portugal
  - 🇬🇧 United Kingdom
  - 🇮🇪 Ireland
  - 🇸🇪 Sweden
  - 🇩🇰 Denmark
  - 🇫🇮 Finland
  - 🇳🇴 Norway
  - 🇵🇱 Poland
  - 🇨🇿 Czech Republic
  - 🇸🇰 Slovakia
  - 🇭🇺 Hungary
  - 🇷🇴 Romania
  - 🇧🇬 Bulgaria
  - 🇬🇷 Greece
  - 🇺🇸 United States
- **Notes**: Main nationality.

#### Field: jobTitle
- **Type**: Multi-select
- **Required**: No
- **Options**:
  - CEO
  - COO
  - CFO
  - CTO
  - CPO
  - CMO
  - CRO
  - Head of Product
  - Head of Growth
  - Head of Sales
  - Head of Marketing
  - Head of Operations
  - Head of Finance
  - Head of Legal
  - Product Manager
  - Product Owner
  - Lead Product Designer
  - UX Designer
  - UI Designer
  - Software Engineer
  - Full Stack Developer
  - Backend Engineer
  - Frontend Engineer
  - Data Scientist
  - Data Analyst
  - DevOps
  - Cloud Architect
  - Customer Success
  - Account Manager
  - Business Developer
  - Sales Manager
  - Marketing Manager
  - Community Manager
  - Legal Counsel
  - Lawyer
  - Accountant
  - Auditor
  - Investor Relations
  - Board Member
  - Founder
  - Co-founder
  - Partner
  - Operating Partner
  - Advisor
- **Notes**: Roles/skills (startup operators, legal, finance, tech, growth).

#### Field: company
- **Type**: Single line text
- **Required**: No
- **Notes**: Main company of the LP.

#### Field: email
- **Type**: Email
- **Required**: Yes
- **Notes**: Primary email.

#### Field: email2
- **Type**: Email
- **Required**: No
- **Notes**: Secondary email.

#### Field: linkedinOrTwitter
- **Type**: URL
- **Required**: No
- **Notes**: LinkedIn or Twitter link.

#### Field: phone
- **Type**: Phone
- **Required**: No

#### Field: dateOfBirth
- **Type**: Date
- **Required**: Yes

#### Field: placeOfBirth
- **Type**: Single line text
- **Required**: Yes

#### Field: personalAddress
- **Type**: Single line text
- **Required**: Yes

#### Field: updatedCredential
- **Type**: Date
- **Required**: No
- **Notes**: Last KYC update.

#### Field: idsExpirationDate
- **Type**: Date
- **Required**: No
- **Notes**: ID expiration date.

#### Field: idDocuments
- **Type**: Attachment
- **Required**: No
- **Notes**: Passport, ID card, residence permit.

#### Field: certificateOfNonConviction
- **Type**: Attachment
- **Required**: No

#### Field: holdingName
- **Type**: Single line text
- **Required**: No
- **Notes**: Holding name if applicable.

#### Field: holdingType
- **Type**: Single-select
- **Required**: No
- **Options**: SASU, SARL, SA, etc.

#### Field: holdingNationality
- **Type**: Single-select
- **Required**: No
- **Options**:
  - 🇫🇷 France
  - 🇧🇪 Belgium
  - 🇨🇭 Switzerland
  - 🇩🇪 Germany
  - 🇪🇸 Spain
  - 🇮🇹 Italy
  - 🇳🇱 Netherlands
  - 🇱🇺 Luxembourg
  - 🇦🇹 Austria
  - 🇵🇹 Portugal
  - 🇬🇧 United Kingdom
  - 🇮🇪 Ireland
  - 🇸🇪 Sweden
  - 🇩🇰 Denmark
  - 🇫🇮 Finland
  - 🇳🇴 Norway
  - 🇵🇱 Poland
  - 🇨🇿 Czech Republic
  - 🇸🇰 Slovakia
  - 🇭🇺 Hungary
  - 🇷🇴 Romania
  - 🇧🇬 Bulgaria
  - 🇬🇷 Greece
  - 🇺🇸 United States
- **Notes**: Nationality of the holding company.

#### Field: holdingAddress
- **Type**: Single line text
- **Required**: No

#### Field: holdingIdSiret
- **Type**: Single line text
- **Required**: No

#### Field: holdingIdRcs
- **Type**: Single line text
- **Required**: No

#### Field: shareCapital
- **Type**: Single line text
- **Required**: No

#### Field: kbis
- **Type**: Attachment
- **Required**: No

#### Field: otherDocuments
- **Type**: Attachment
- **Required**: No

#### Field: network
- **Type**: Single-select
- **Required**: No
- **Options**: Source Ventures, Martin, Victor, Hugo, etc.

#### Field: createdTime
- **Type**: Date
- **Required**: No

#### Field: editorialisation
- **Type**: Long text
- **Required**: No

#### Field: spvFundraising
- **Type**: Link to SPV Fundraising
- **Required**: No

---

<!-- Table: Investments -->

### 2.6 Investments

| Field | Type | Notes |
| **Investor** | Link → *Investors* | |
| **Deal** | Link → *Deals* | |
| Fund      | Link → *Funds* | optional |
| Amount    | Currency | |
| Date      | Date | |
| Equity_%  | Number (pct) | |

### 2.6 Fundraising

#### Field: id
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: CONCATENATE({line}, " - ", {lp})
- **Notes**: Unique identifier (line + LP).

#### Field: line
- **Type**: Link to Lines
- **Required**: Yes
- **Notes**: Linked investment line.

#### Field: lp
- **Type**: Link to LPs
- **Required**: Yes
- **Notes**: Linked LP.

#### Field: investment
- **Type**: Currency
- **Required**: Yes
- **Notes**: Amount invested by the LP.

#### Field: payment
- **Type**: Checkbox
- **Required**: No
- **Notes**: Payment received (manual, ideally API-linked in future).

#### Field: holdingType
- **Type**: Single-select
- **Required**: No
- **Options**: Holding, Personnal
- **Notes**: Type of vehicle used by the LP.

#### Field: fees
- **Type**: Checkbox
- **Required**: No
- **Default**: Checked
- **Notes**: LP is charged fees (checked by default).

#### Field: carried
- **Type**: Checkbox
- **Required**: No
- **Default**: Checked
- **Notes**: LP benefits from carried (checked by default).

#### Field: feesHT
- **Type**: Formula
- **Required**: No
- **Formula**: IF({fees} = TRUE(), {investment} * 0.05, 0)
- **Notes**: 5% fees if applicable.

#### Field: vat
- **Type**: Formula
- **Required**: No
- **Formula**: IF({holdingNationality} = "🇫🇷 France" OR {holdingNationality} = "", {feesHT} * 0.2, 0)
- **Notes**: VAT only for French investors.

#### Field: feesTTC
- **Type**: Formula
- **Required**: No
- **Formula**: {feesHT} + {vat}
- **Notes**: Total fees including VAT.

#### Field: createdTime
- **Type**: Date
- **Required**: No

---

<!-- Table: KPIs -->

### 2.7 KPIs

#### Field: id
- **Type**: Formula
- **Required**: Yes
- **Unique**: Yes
- **Formula**: CONCATENATE({company}, "-", {period}, "-", {metric})
- **Notes**: Unique identifier for each monthly KPI per startup.

#### Field: company
- **Type**: Link to Companies
- **Required**: Yes
- **Notes**: Startup concerned.

#### Field: period
- **Type**: Single-select
- **Required**: Yes
- **Options**: 2023-01, 2023-02, ..., 2024-12, etc.
- **Notes**: Month of the KPI (format YYYY-MM).

#### Field: metric
- **Type**: Single-select
- **Required**: Yes
- **Options**:
  - MRR (Monthly Recurring Revenue)
  - ARR (Annual Recurring Revenue)
  - EBITDA
  - Revenue
  - Cash
  - Cash Burn
- **Notes**: KPI type.

#### Field: value
- **Type**: Number
- **Required**: No
- **Notes**: KPI value.

#### Field: currency
- **Type**: Single-select
- **Required**: Yes
- **Options**: €, $
- **Notes**: Currency of the KPI.

#### Field: file
- **Type**: Link to Files
- **Required**: No
- **Notes**: Source file (reporting, balance sheet, etc).

#### Field: sourceType
- **Type**: Single-select
- **Required**: No
- **Options**: Reporting, Balance Sheet, Other
- **Notes**: Source of the KPI.

---

## 3 · Relations & Roll‑ups

* *Companies* 1‑n Files → all files linked to company
* *Companies* 1‑n Reportings → rollup `Last_MRR`, `Last_Burn`.
* *Lines* 1‑n Investments → rollup `Total_SV`, `Avg_Ticket`.
* *Investors* 1‑n Investments → rollup `Total_Committed`, `IRR`.

---

## 4 · Automations & Alerts

| Trigger                        | Action                                                |
| ------------------------------ | ----------------------------------------------------- |
| `KYC_Status = "Expired"`       | Bot posts alert in `#compliance-alerts`               |
| New attachment in *Reportings* | Webhook → bot parses & fills KPI table                |
| Weekly cron                    | Check missing annual accounts per SPV; post checklist |

---

## 5 · Risks & Mitigations

| Risk                        | Mitigation                                                   |
| --------------------------- | ------------------------------------------------------------ |
| 50 k record hard‑cap        | Archive >3y data to Postgres, keep rolling view in Airtable |
| API 5 rps limit             | Bot uses Redis queue + upsert batching                       |
| Public URLs for attachments | Option: encrypt PDF before upload or use private S3 bucket   |

---

## 6 · Migration Plan (legacy → v0.2)

1. Export legacy tables via `airtable-export`.
2. Python script maps:

   * `Lines` ➜ *Investments*
   * LP/Outside/Co‑Investors ➜ *Investors* (field `LegacyType`)
3. Re‑link old IDs → new foreign keys.

---

## 7 · Revision Log

| Date        | Change                   | Author |
| ----------- | ------------------------ | ------ |
| 19 Jun 2025 | Initial v0.2 schema dump | Nova   |

---

→ **Next:** validate fields, especially KPI JSON structure & roll‑ups. Then generate CSVs for table import + migration script.
