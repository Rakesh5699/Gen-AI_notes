## 2. Raw Data Inventory

The ingestion team receives files from multiple sources. Some files are clean and digital; others contain scanned pages, rotated tables, missing metadata, embedded images, or formula-generated values. The inventory below intentionally includes different formats and extraction expectations.

| Asset ID       | Client            | Format      | Pages/Rows   | Expected extraction          | Notes                                    |
|----------------|-------------------|-------------|--------------|------------------------------|------------------------------------------|
| ARK-MSA-001    | Arka Finance      | PDF         | 32 pages     | Clauses, tables, signatures  | Born-digital text with footer references |
| ARK-DPA-002    | Arka Finance      | DOCX        | 18 pages     | Headings, lists, tables      | Track changes removed before ingestion   |
| BLR-PRICE-00 3 | BlueLeaf Retail   | XLSX        | 12 sheets    | Rate cards, discounts        | Merged cells and formulas                |
| BLR-SLA-004    | BlueLeaf Retail   | PDF         | 9 pages      | SLA table, escalation matrix | Table crosses page boundary              |
| CRM-OPS-005    | CityRide Mobility | CSV         | 48,200 rows  | Incident records             | Needs column type normalization          |
| CRM-FORM-00 6  | CityRide Mobility | Scanned PDF | 4 pages      | OCR fields, checkbox         | Low contrast and slight rotation         |
| KB-RUN-007     | Shared            | Markdown    | 14 files     | Runbook sections             | Good for heading-based chunks            |
| WEB-FAQ-008    | Shared            | HTML        | 23 pages     | FAQ content, links           | Needs boilerplate removal                |

Parsing difficulty increases when a document combines multiple layouts: paragraph text, tables, images, and notes. Good parsers preserve structure; weak parsers flatten everything into a noisy text stream.

## 4. Simple Table: Policy Rules

The following table is intentionally simple. It should be correctly extracted by most PDF table parsers. It tests basic row and column detection, short text values, and numeric values.

| Policy Area       | Rule                                                          | Owner            | Review Cycle        |
|-------------------|---------------------------------------------------------------|------------------|---------------------|
| Refunds           | Refund requests must be raised within 7 days of purchase.     | Customer Support | Quarterly           |
| Data Export       | Client admins can request export in CSV or JSON format.       | Platform Ops     | Monthly             |
| Access Review     | Privileged access must be reviewed every 30 days.             | Security         | Monthly             |
| Incident Response | Critical incidents require acknowledgement within 30 minutes. | SRE              | After each incident |
| Vendor Review     | High-risk vendors require annual risk assessment.             | Procurement      | Annual              |

## Narrative Around the Table

This  table  appears  between  paragraphs,  which  is  common  in  enterprise  documents.  A  good  parser  should  preserve  the paragraph before the table, the table content, and the paragraph after the table in the correct reading order.

For RAG systems, simple tables are often converted into row-wise text chunks such as: Policy Area: Refunds; Rule: Refund requests must be raised within 7 days; Owner: Customer Support.

## 10. Multi-tenant Retrieval and Access Control

In a multi-tenant RAG system, each document, chunk, embedding, and retrieval request should be associated with a tenant identifier. Client isolation should happen before the LLM receives any context.

| Layer               | Control                              | Failure mode if missing                     |
|---------------------|--------------------------------------|---------------------------------------------|
| Authentication      | Identify user and organization       | Unknown user may access application         |
| Authorization       | Map user to client_id and role       | User may retrieve another clients documents |
| Vector retrieval    | Filter by client_id or namespace     | Retriever may return unauthorized chunks    |
| Prompt construction | Send only allowed context            | LLM may see sensitive data                  |
| Audit logging       | Record query, source chunks, user_id | No traceability during incident review      |

Correct flow: authenticate user, get client\_id from backend, retrieve only matching chunks, build prompt with authorized context, return answer with citations.

Wrong flow: retrieve from all clients and tell the model to ignore unauthorized content. This is unsafe because the LLM should not be used as the access-control boundary.

## 12. Evaluation Dataset

A  RAG  system  should  be  evaluated  separately  for  retrieval  quality  and  answer  quality.  This  page  includes  a  miniature evaluation set with expected source references. It is useful for testing whether citations point to the correct section.

| Test ID   | Question                                 | Expected source    | Expected answer element    | Failure signal             |
|-----------|------------------------------------------|--------------------|----------------------------|----------------------------|
| Q-001     | What is the refund window?               | Policy Rules table | 7 days                     | Answer says 30 days        |
| Q-002     | Which clause covers data residency?      | Clause 9.1         | Approved processing region | No clause citation         |
| Q-003     | What is P1 API response target?          | SLA table          | 15 minutes                 | Wrong severity row         |
| Q-004     | Which documents belong to BLR-ACME-2026? | Relationship table | Five related documents     | Only one document returned |
| Q-005     | Who owns access review?                  | Policy Rules table | Security                   | Owner missing              |

Evaluation should include adversarial questions, unrelated questions, and questions that require multi-hop retrieval across related documents.

## 13. Edge Cases for Parsers

The following cases often break real document ingestion pipelines. They are included here as guidance for testing parser quality before moving to embeddings and vector storage.

| Edge case                    | Example                             | Recommended handling                         |
|------------------------------|-------------------------------------|----------------------------------------------|
| Repeated headers and footers | Page number, confidentiality banner | Remove or store separately as metadata       |
| Hyphenated line breaks       | termi- nation assistance            | Normalize during cleaning                    |
| Rotated tables               | Landscape appendix in PDF           | Use layout-aware parser or OCR               |
| Merged cells                 | Pricing table with grouped plans    | Preserve hierarchy in row text               |
| Scanned signatures           | Signature block as image            | OCR if text is needed; store image reference |
| Boilerplate navigation       | Website header and footer           | Use boilerplate removal                      |
| Duplicate chunks             | Same policy in FAQ and PDF          | Deduplicate using source and hash            |

Bad parsing creates bad chunks. Bad chunks create bad retrieval. Bad retrieval creates bad answers.

## 15. Final Ingestion Checklist

Use this checklist before sending parsed content into chunking and embeddings. It helps identify whether the data is ready for production RAG.

| Checklist item   | Status to verify                                | Why it matters                       |
|------------------|-------------------------------------------------|--------------------------------------|
| Text extraction  | Paragraphs are readable and ordered             | Prevents noisy chunks                |
| Table extraction | Rows, columns, headers, and footnotes preserved | Protects factual answers             |
| Image handling   | Captions indexed and OCR done if required       | Avoids missing visual information    |
| Metadata         | source, page, client_id, document_id captured   | Enables citations and access control |
| Chunking         | Chunks preserve meaning and section boundaries  | Improves retrieval relevance         |
| Access control   | Retrieval filters use authenticated client_id   | Prevents data leakage                |
| Evaluation       | Golden questions tested with citations          | Measures real answer quality         |

Summary: RAG is about knowledge access. Fine-tuning is about behavior adaptation. For document intelligence, parsing quality is the foundation. If extraction is weak, no embedding model or LLM can fully fix the missing context.

End of synthetic 15-page parsing test document.

## Appendix A: Complex Clause Responsibility Matrix

Grouped clauses with owner/backup split inside the same responsibility cell. This page is useful for testing row grouping, merged-looking labels, split responsibility cells, and long evidence text.

Parsing challenge: preserve row boundaries, nested headers, split cells, grouped labels, numeric values, and footnotes/context around the table.

or aes a ani u ae dn de Ja ae e aiisosar ids e de

| Clause Group    | Obligation                                                                          | Responsible Team   | Responsible Team       | Trigger                                 | Evidence Required                             | Risk     |
|-----------------|-------------------------------------------------------------------------------------|--------------------|------------------------|-----------------------------------------|-----------------------------------------------|----------|
| Data Protection | Delete client data after contract termination unless retention is legally required. | Owner Backup       | Compliance Legal       | Termination notice received             | Deletion certificate + audit log eodxt        | High     |
| Data Protection | Notify client about any confirmed data incident within 72 hours.                    | Owner Backup       | Security DPO           | Incident classified as confirmed breach | Incident report, timeline, notification proof | Critical |
| Billing         | Apply annual platform fee adjustment only after renewal confirmation.               | Owner Backup       | Finance CSM            | Renewal order approved                  | Approved renewal sheet + invoice draft        | Medium   |
| Billing         | Do not bill inactive campuses during suspension period.                             | Owner Backup       | Revenue Ops Finance    | Campus status = susuadses               | ERP campus status export                      | High     |
| Support         | Provide P1 response within 30 minutes during school operating hours.                | Owner Backup       | Support L2 Ops Manager | Ticket priority = P1                    | Ticket timestamps + agent assignment log      | High     |
| Support         | Escalate unresolved P2 tickets after 4 business hours.                              | Owner Backup       | Support L1 Support L2  | Ticket age > 4 business hours           | Escalation log                                | Medium   |

Table 1: Added as complex parsing appendix for table extraction, OCR fallback, and layout-aware RAG testing.

## Appendix B: Regional Pricing and Usage Add-on Matrix

Pricing table with multi-level headers, regional columns, add-on columns, billing rules, exception rows, and mixed numeric/text values.

Parsing challenge: preserve row boundaries, nested headers, split cells, grouped labels, numeric values, and footnotes/context around the table.

| Plan       | Student Volume          | Annual Platform Charges           | Annual Platform Charges           | Annual Platform Charges           | Usage Add-ons                    | Usage Add-ons                    | Billing Rule                |
|------------|-------------------------|-----------------------------------|-----------------------------------|-----------------------------------|----------------------------------|----------------------------------|-----------------------------|
|            |                         | India Region                      | India Region                      | International                     | SMS                              | hasp                             |                             |
| Starter    | 0 - 2,000 students      | Base Support                      | INR 4.5L INR 60K                  | USD 7,200                         | INR 0.18/message                 | INR 0.42/message                 | Quarterly advance           |
| Growth     | 2,001 - 10,000 students | Base Support                      | INR 11L INR 1.4L                  | USD 18,000                        | INR 0.15/message                 | INR 0.38/message                 | 50% advance + monthly usage |
| Enterprise | 10,001+ students        | Base Support                      | Custom Included                   | Custom                            | Negotiated                       | Negotiated                       | Signed order form required  |
| Exception  | Government schools      | Discount may apply after approval | Discount may apply after approval | Discount may apply after approval | No discount on pass-through cost | No discount on pass-through cost | Requires CFO approval       |

Table 2: Added as complex parsing appendix for table extraction, OCR fallback, and layout-aware RAG testing.

## Appendix C: Invoice Line Items with Tax Split

Invoice-style line item table with item groups, quantity, rate, CGST/SGST split, totals, and summary row. Useful for invoice parsing and tax extraction tests.

Parsing challenge: preserve row boundaries, nested headers, split cells, grouped labels, numeric values, and footnotes/context around the table.

| Item Group     | Line Item                                              | Qty          | Rate          | Tax Split    | Tax Split    | Total         |
|----------------|--------------------------------------------------------|--------------|---------------|--------------|--------------|---------------|
|                |                                                        |              |               | CGST         | SGST         |               |
| ERP Platform   | Annual School360 Enterprise Subscription - 12 campuses | 1            | INR 11,00,000 | 9%           | %6           | INR12,98,000  |
|                | Parent communication add-on - estimated message pack   | 2,00,000 ssg | INR 0.38/msg  | 9%           | 9%           | INR 89,680    |
| Implementation | Data migration + training + go-live support            | 1            | INR 2,40,000  | 9%           | %6           | INR 2,83,200  |
| Summary        | Subtotal and taxes                                     |              | INR 14,16,000 | INR 1,27,440 | INR 1,27,440 | INR 16,70,880 |

Table 3: Added as complex parsing appendix for table extraction, OCR fallback, and layout-aware RAG testing.

## Appendix D: Multimodal Document Processing Flow

Diagram-style image showing how PDFs, DOCX files, scanned invoices, Excel/CSV metadata, parser/OCR, extracted text/tables/metadata, and RAG-ready chunks connect.

Parsing challenge: image text, arrows, labels, and captions may not appear in normal PDF text extraction. OCR or multimodal parsing may be required.

<!-- image -->

Added at the end for complex image parsing, OCR fallback, layout-aware extraction, and multimodal RAG testing.

## Appendix E: Contract Risk Dashboard Image

Dashboard-style image containing a bar chart, legend, and small matrix. Useful for testing chart extraction, numeric value capture, and caption-aware indexing.

Parsing challenge: extract chart title, bar values, legend labels, and table values from an embedded image.

## Visual Appendix 2: Contract Risk Dashboard Snapshot

Parsing challenge: extract chart labels, legends, values, and nearby explanatory text.

<!-- image -->

| High risk: 12 clauses High risk: 12 clauses     |
|-------------------------------------------------|
| Medium risk: 21 clauses Medium risk: 21 clauses |
| Low risk: 45 clauses Low risk: 45 clauses       |

| Client Client     | Open Open   | Critical Critical   | Owner Owner             |
|-------------------|-------------|---------------------|-------------------------|
| Arka Arka         | 17 17       | 4 4                 | Legal Legal             |
| BlueLeaf BlueLeaf | 22 22       | 6 6                 | Procurement Procurement |
| CityRide CityRide | 16 16       | 2 2                 | Ops Ops                 |

Expected extraction: chart title, series values, legend labels, table values, and risk summary.

Added at the end for complex image parsing, OCR fallback, layout-aware extraction, and multimodal RAG testing.

## Appendix F: Data Lineage and Access Boundary Image

Lineage-map style image with nodes, arrows, access boundaries, and metadata badges. Useful for diagram OCR and relationship extraction.

Parsing challenge:

diagram text must be OCRed and mapped to relationships such as user auth, namespace, retrieval, and

LLM gateway.

<!-- image -->

Added at the end for complex image parsing, OCR fallback, layout-aware extraction, and multimodal RAG testing.

## Appendix G: Additional Scanned Invoice for OCR Testing

Synthetic scanned tax invoice with vendor details, customer details, invoice number, GSTIN, line items, tax split, total amount, payment terms, footer notes, approval stamp, and handwritten-style receipt text.

Parsing challenge: this page is intentionally embedded as an image-like scan. A normal text parser may miss invoice values unless OCR is enabled.

|   # | Description                   |   Qty | Rate       |   Amount |
|-----|-------------------------------|-------|------------|----------|
|   1 | Annual platform subscription  |     1 | INR 95,000 |   95,000 |
|   2 | Implementation and onboarding |     1 | INR 22,500 |   22,500 |
|   3 | Support add-on / message pack | 3,500 | INR 0.40   |    1,400 |

<!-- image -->

Added at the end for complex image parsing, OCR fallback, layout-aware extraction, and multimodal RAG testing.

## Appendix H: Additional Scanned Utility Bill for OCR Testing

Synthetic scanned utility bill with meter-style charges, usage rows, tax values, total payable amount, payment terms, stamp, handwritten-style note, and noisy/rotated scan effects.

Parsing challenge: extract bill number, issuer, line items, usage quantity, tax split, total payable, and payment notes from a scanned image.

<!-- image -->

Added at the end for complex image parsing, OCR fallback, layout-aware extraction, and multimodal RAG testing.

|   # | Description                   |   Qty | Rate      |   Amount |
|-----|-------------------------------|-------|-----------|----------|
|   1 | Electricity fixed charges     |     1 | INR 1,250 |    1,250 |
|   2 | Energy usage - peak units     |   842 | INR 8.20  |    6,904 |
|   3 | Energy usage - off-peak units |   313 | INR 5.70  |    1,784 |
|   4 | Meter service adjustment      |     1 | INR 340   |      340 |

## Appendix I: Profile Image for Multimodal Parsing

This page adds a portrait-style image to test how a parser or multimodal RAG system handles photographic content, captions, image metadata, and surrounding text. Text-only loaders may extract the caption but cannot understand the visual content unless OCR, vision, or multimodal parsing is used.

<!-- image -->

Caption: Portrait-style instructor image with a bright background. Useful for testing image extraction, captioning, person detection, layout preservation, and multimodal document understanding.

| Element        | Parsing Challenge                                | Expected Handling                                                       |
|----------------|--------------------------------------------------|-------------------------------------------------------------------------|
| Portrait image | Pixels are not selectable PDF text.              | Extract image object or create visual summary with a multimodal model.  |
| Caption text   | Caption should stay attached to image context.   | Preserve caption near the image in reading order.                       |
| Image metadata | File name, page, and bounding box may be needed. | Store metadata for citation and retrieval.                              |
| RAG usage      | Text-only chunks may miss visual details.        | Use OCR or vision captioning before chunking if visual content matters. |

Added at the end for complex image parsing, multimodal RAG testing, caption grounding, and visual-content extraction checks.