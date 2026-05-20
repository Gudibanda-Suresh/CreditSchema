# participant/v1 — Credit Participant Attributes

Attaches to: **`Participant.participantAttributes`**

Two schemas in this pack, distinguished by `@type`:

| Schema | Role | Required fields |
|---|---|---|
| `Borrower` | Individual borrower (`role = BORROWER`) | `fullName`, `nik` |
| `Lender` | Lending institution (`role = LENDER`) | `institutionName`, `ojkLicenseNumber` |

## Borrower fields

| Field | Type | Required | Description |
|---|---|---|---|
| `fullName` | string | Yes | Full name as on KTP |
| `nik` | string (16 digits) | Yes | National Identity Number |
| `npwp` | string | No | Tax ID (NPWP) |
| `birthPlace` | string | No | City of birth from KTP |
| `birthDate` | date | No | Date of birth (YYYY-MM-DD) |
| `registeredAddress` | string | No | Full KTP address |
| `email` | email | No | Contact email for OTP |
| `phone` | string | No | E.164 mobile number |

## Lender fields

| Field | Type | Required | Description |
|---|---|---|---|
| `institutionName` | string | Yes | Full registered entity name |
| `ojkLicenseNumber` | string | Yes | OJK operating license |
| `branchOffice` | string | No | Handling branch name |
| `relationshipManagerName` | string | No | Account Officer name |

## Wire placement

```json
{
  "participants": [
    {
      "role": "BORROWER",
      "participantAttributes": {
        "@context": "https://schema.ion.id/finance/participant/v1/context.jsonld",
        "@type": "Borrower",
        "fullName": "Budi Santoso",
        "nik": "3273012505800001"
      }
    },
    {
      "role": "LENDER",
      "participantAttributes": {
        "@context": "https://schema.ion.id/finance/participant/v1/context.jsonld",
        "@type": "Lender",
        "institutionName": "PT Bank Rakyat Indonesia (Persero) Tbk",
        "ojkLicenseNumber": "KEP-046/DIR/1992"
      }
    }
  ]
}
```
