# FranchiseOps AI — Milestone 3

**Infosys Springboard Internship 7.0 · Batch 4**  
**Project:** FranchiseOps AI — An Intelligent Multi-Agent Platform for Franchise Operations  
---

## 1. Project overview

FranchiseOps AI is an agentic operations platform intended to help a franchise chain monitor outlet performance and identify operational issues such as stock shortages, staffing risks, audit failures and ineffective campaigns. Specialist agents analyse operational data, the Intelligence Engine combines their findings, and the policy/approval workflow determines whether an action can run automatically or needs human approval.

The platform includes a live simulator for the demonstration environment. Sales, stock, attendance and staff data are seeded/simulated unless a real source is connected through the Data Feed Center. Weather and public-holiday context can be obtained from external feeds. The project should therefore be presented as a working demonstration and decision-support system—not as a production deployment connected to a real franchise's POS/HR systems unless those feeds have actually been configured.

### Main capabilities

- Continuous/scheduled outlet monitoring and event-triggered agent runs.
- Planner and re-planning across specialist agents using LangGraph.
- Outlet health scoring and cross-agent action proposals.
- Policy gate, role-based approval chains, four-eyes checks, audit history, ticket ownership, SLAs and escalation.
- AI Copilot with read-only SQL answers, agent-based analysis and cited knowledge-base answers.
- Hybrid RAG over public documents and private uploads.
- Six what-if simulators and the Digital Twin.
- Reports, notifications, data-feed ingestion, role-based pages and account administration.

## 2. System architecture

```text
Public websites / PDF documents
              |
              v
       RAG Builder notebook
  crawl -> download -> validate -> extract
  -> deduplicate -> chunk -> embed
  -> FAISS + BM25 -> rerank/evaluate -> export
              |
              v
      Google Drive knowledge corpus
              |
              v
     FranchiseOps AI application
              |
  +-----------+------------+----------------+
  |                        |                |
  v                        v                v
SQL / live data       Specialist agents   Hybrid RAG
  |                        |                |
  +------------------------+----------------+
                           |
                           v
                    Qwen Copilot
                           |
                           v
          Cited response / decision / ticket
                           |
                           v
              Policy gate and human approval
                           |
                           v
               Approved action + audit log
```

### Technology stack

| Layer | Technology | Role in this project |
|---|---|---|
| Notebook/runtime | Google Colab, Python | Runs the RAG preparation and application startup workflow. A T4 GPU is recommended for model inference and embeddings. |
| Web/API | FastAPI, Uvicorn | Serves the application pages and API endpoints. |
| Frontend | HTML, CSS, vanilla JavaScript, server-driven UI blocks | Renders dashboards, agent pages, forms, tables and Copilot responses. |
| Database | SQLite (WAL mode) | Stores business/demo data, users, agent runs, actions, approvals, chats, knowledge chunks and audit records. |
| Agent orchestration | LangGraph | Coordinates planning, agent execution, re-planning and decision flow. |
| Classical ML | scikit-learn, NumPy, pandas, SciPy | Outlet/customer clustering, attrition classification, anomaly detection and simulator calculations. |
| Retrieval | BM25, FAISS, Sentence Transformers, Reciprocal Rank Fusion | Keyword + dense semantic retrieval and hybrid ranking. |
| Reranking | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Reorders candidate passages for query relevance. |
| Language model | Qwen2.5-Instruct (GPU-dependent size) | Explains retrieved evidence and produces Copilot responses; it is not the source of operational numeric facts. |
| Translation | NLLB-200 distilled 600M | Supports multilingual question/answer translation where enabled. |
| Notifications | Gmail SMTP; optional Slack/Teams/SMS hooks | OTP/report/alert delivery when credentials and integrations are configured. |
| Hosting/demo | Colab proxy or ngrok | Exposes the running Colab application for demonstration. |

## 3. Repository and submission structure

The Milestone 3 instruction asks for a `Milestone3` folder containing both notebooks, this README, `taught_questions.csv`, the presentation in PPTX and PDF formats, and evidence screenshots.

Recommended layout:

```text
Milestone3/
├── README.md
├── FranchiseOpsAI_RAG_Builder_Milestone3_Task1.ipynb
├── FranchiseOps_AI_Final_Code.ipynb
├── taught_questions.csv
├── presentation/
│   ├── FranchiseOps_AI_Presentation.pptx
│   └── FranchiseOps_AI_Presentation.pdf
└── screenshots/
    ├── rag_150_sources_and_chunks.png
    ├── teach_copilot_50_of_50.png
    ├── copilot_rag_sources_1_2.png
    ├── inventory_simulator_before.png
    ├── inventory_simulator_after.png
    ├── workforce_simulator_before.png
    ├── workforce_simulator_after.png
    ├── pnl_simulator_before.png
    ├── pnl_simulator_after.png
    ├── marketing_simulator_before.png
    ├── marketing_simulator_after.png
    ├── sentiment_simulator_before.png
    ├── sentiment_simulator_after.png
    ├── audit_simulator_before.png
    ├── audit_simulator_after.png
    └── admin_copilot_insights.png
```

**Notebook naming:** The repository may use the consolidated `FranchiseOps_AI_Milestone3_All_In_One_Crawler_Updated.ipynb` instead of an app-only notebook. That consolidated notebook contains both the RAG preparation stage and the application stage. If it is submitted, rename the tree entry accordingly and explain that it combines the two stages. Do not run the same crawl twice by running both the standalone builder and the all-in-one notebook unnecessarily.

## 4. Running the notebooks

### 4.1 RAG Builder notebook

1. Open `FranchiseOpsAI_RAG_Builder_Milestone3_Task1.ipynb` in Google Colab.
2. Select **Runtime → Change runtime type → T4 GPU** (recommended).
3. Run the notebook from the first cell. Authorise Google Drive access when prompted.
4. The crawler reads its configured `FRANCHISE_SEEDS`, visits public web pages, discovers PDF links, downloads candidate PDFs and stores them in Google Drive.
5. The extraction stage validates PDF files, deduplicates file contents using SHA-256, extracts text from readable pages and writes a PDF catalogue.
6. The chunking stage combines the project SOP pack and extracted PDF pages into source-attributed chunks.
7. The embedding/index stage builds dense vectors, FAISS and BM25 indexes, runs sample retrieval checks and exports a corpus for the application.
8. Review the milestone evidence output. The target is not complete unless the extracted-PDF and PDF-source-in-chunks counts both meet the required threshold.

**Current crawler configuration in the updated notebook:**

| Setting | Value |
|---|---:|
| Configured seed entries | 145 |
| Unique seed URLs | 144 |
| Unique seed domains | 61 |
| Crawl depth | 2 |
| HTML page budget | 400 |
| Per-seed PDF discovery cap | 30 |
| Global PDF candidate cap | 350 |
| Per-seed time budget | 18 seconds |
| Maximum PDF size | 50 MB |
| Chunk size | 400 words |
| Chunk overlap | 80 words |
| Required extracted PDF target | 150 unique PDF contents |

The seed count is the number of configured starting points, not the number of successful documents. A seed may yield no PDFs, multiple PDFs, blocked pages or duplicate files. The crawler can only confirm collection after download and validation.

### 4.2 FranchiseOps AI application notebook

1. Open `FranchiseOps_AI_Final_Code.ipynb` (or the consolidated all-in-one notebook, if that is the submitted version).
2. Select a T4 GPU runtime where available.
3. Set `APP_KIND="Franchise"`, `USE_DRIVE=True` and `PERSIST=True` in the application launch configuration.
4. Mount Google Drive and confirm that the application points to `MyDrive/FranchiseOps_AI/` for its knowledge corpus.
5. Add only the required credentials to **Colab Secrets**. Do not put tokens or passwords in notebook cells.
6. Run the application cells in order. Wait for the FastAPI health check to succeed, then open the Colab proxy/ngrok URL printed by the notebook.
7. Sign in using a demo or administrator-created account. Verify the role-specific menu and outlet scope before the demonstration.
8. Open **AI Copilot → Knowledge base** and use **Reload Drive RAG index** after a newer Builder corpus is generated. Confirm the imported chunk count and PDF-source count.

### Required/optional Colab Secrets

| Secret name | Required for | Notes |
|---|---|---|
| `EMAIL_ID` | Gmail OTP/report sending | Sender Gmail address. |
| `EMAIL_PASSWORD` | Gmail SMTP | Gmail App Password, not the normal Gmail login password. |
| `NGROK_AUTHTOKEN` | Optional public tunnel | Use only if the Colab proxy is unavailable. |
| `HF_TOKEN` | Optional Hugging Face access | May help with gated models or rate limits; public models may work without it. |
| `JWT_SECRET_KEY` | Optional explicit JWT signing secret | If omitted, the app creates/persists a generated secret in its data directory. |

Enable notebook access for the secrets in Colab. Never print their values. Never commit them to GitHub.

## 5. Agents and their responsibilities

The application registers eight specialist agents. The first five listed below contribute to the weighted outlet health score; Feedback, Anomaly and Digest provide specialist analysis/reporting.

| Agent | Main technique | Responsibility / output |
|---|---|---|
| Outlet | K-Means clustering (k=4) and tier/rank scoring | Groups outlets into performance tiers and identifies outlets needing review. |
| Inventory | Demand forecasting, lead-time/coverage checks and reorder rules | Estimates demand and days of cover; proposes reorder actions when stock risk is detected. |
| Staff / Workforce | Logistic Regression attrition prediction plus staffing rules | Estimates employee attrition risk and identifies staffing shortages. |
| Marketing | K-Means customer segmentation (k=3), campaign ROI calculations | Segments customer behaviour and proposes win-back campaigns or budget changes. |
| Audit | Threshold-based rules | Classifies audit outcomes and proposes corrective actions for failures. |
| Feedback | VADER sentiment and issue-category rules | Analyses ratings/comments and highlights service-recovery issues. |
| Anomaly | Isolation Forest plus explicit threshold checks | Flags unusual sales/volatility/sentiment patterns for review. |
| Digest | Aggregation, ranking and summarisation | Consolidates agent results into weekly/management digests. |

### Agent coordination and decision flow

- The planner selects agents based on the request or scheduled cycle.
- Re-plan rules can add agents based on findings (for example, inventory risk can add Marketing; a high audit risk can add Staff and Feedback).
- The Intelligence Engine combines agent findings and calculates outlet health using the configured weights:

  `Health = 0.30 × Outlet + 0.20 × Inventory + 0.20 × Staff + 0.15 × Marketing + 0.15 × Audit`

- Risk/action policies decide whether an action is automatic or requires approval.
- Approval chains, four-eyes checks, action ownership, SLAs, escalations and audit records govern consequential operations.
- Approved actions can create purchase orders or pause campaigns where an execution handler exists. Other proposals remain tickets for human handling; do not describe every ticket as an automatically executed real-world action.

## 6. AI Copilot: SQL + Agents + RAG

The Copilot routes questions to the layer best suited to answer them. It does not treat every question as a RAG question.

### SQL layer

- Handles numeric and operational questions using read-only, whitelisted SQL through the query/analyst modules.
- Retrieves values such as counts, averages, rankings, outlet details and operational metrics from SQLite.
- Displays the query/evidence when requested (for example, “show the exact query”).
- The language model should not invent operational numbers; numeric answers must be grounded in database evidence.

### Agent layer

- Handles questions requiring current agent analysis, such as “what needs attention?” or “why is this outlet flagged?”.
- Runs or reuses agent-cycle results and can expose an agent trace.
- Agent execution is distinct from document retrieval.

### RAG layer

- Uses public policy/SOP PDFs and other imported knowledge documents.
- BM25 provides keyword retrieval; dense embeddings provide semantic retrieval.
- Reciprocal Rank Fusion combines candidate rankings.
- A cross-encoder reranks candidate passages for relevance.
- Retrieved passages are supplied to Qwen to create a concise answer with numbered citations and a Sources section.
- Uploaded private documents can be stored with user ownership and access boundaries.
- If the evidence does not support an answer, the Copilot should say that it does not know rather than fabricate a policy or fact.

**Models/configuration:**

| Component | Configured model/technique |
|---|---|
| Dense embedder | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` |
| Cross-encoder reranker | `cross-encoder/ms-marco-MiniLM-L-6-v2` |
| Sparse retrieval | BM25 (Okapi) |
| Hybrid fusion | Reciprocal Rank Fusion (RRF) |
| Language model | Qwen2.5-Instruct; selected size depends on available GPU memory |
| Translation | NLLB-200 distilled 600M where enabled |

### How to verify a RAG response

1. Ask a domain/policy question that needs a document, not a live outlet metric.
2. Confirm the response shows the RAG layer as used.
3. Expand **Sources** and check that the titles/URLs are actual indexed source documents—not only “Taught answer” or sample SOP records.
4. Confirm that the cited source passages support the answer.
5. Try a question outside the corpus and verify that the Copilot admits that it cannot find supporting evidence.

A `RAG` indicator or a plausible-sounding response alone does not prove that a public PDF was indexed. Check the source titles, source URLs, extracted-PDF catalogue and chunk counts.

## 7. Knowledge-base sources and collection evidence

### Configured source coverage

The RAG Builder is configured with 145 seed entries (144 unique URLs across 61 domains). The configured source families include:

- **India food safety and food processing:** FSSAI, Ministry of Food Processing Industries and India.gov.in.
- **Food safety standards and guidance:** WHO, FAO/Codex, US FDA, CDC, ServSafe, BRCGS, SQF and ISO 22000.
- **Workforce and labour:** Ministry of Labour and Employment, ILO, EPFO and ESIC.
- **Franchise operations and management:** International Franchise Association and related franchise guidance.
- **Tax, finance and payments:** CBIC/GST resources, RBI and NPCI/UPI resources.
- **Consumer and legal compliance:** Department of Consumer Affairs, India Code, BIS, NCDRC and MCA.
- **Agriculture, procurement and cold chain:** eNAM, APEDA, Coffee Board, Spices Board, MPEDA, NABARD and National Horticulture Board.
- **Business and industry research:** MSME, CII, OECD, World Bank, IFC, UNCTAD, UNIDO, NSDC and MoSPI.
- **Research discovery:** arXiv queries covering retail forecasting, attrition, customer NLP, anomaly detection, RAG and knowledge graphs.

The Milestone 3 additions include 38 additional URL entries covering India-focused food-safety, labour/social-security, GST, consumer-protection, digital-payment, MSME, agricultural-quality, cold-chain and corporate/legal resources. The seed list is a crawl configuration, not proof that every website or PDF was successfully downloaded.

### Collection and indexing results — fill from the final successful run

| Evidence item | Latest known diagnostic snapshot | Final verified result |
|---|---:|---:|
| PDF files downloaded to Drive | 150 | `[UPDATE]` |
| Unique PDF contents successfully extracted | 0 (catalogue was missing at that check) | `[UPDATE]` |
| PDF source URLs represented in chunks | 0 at that check | `[UPDATE]` |
| Builder chunks in `all_chunks.jsonl` | 20 at that check | `[UPDATE]` |
| Chunks in uploaded `kb_export.json` | 0 in the inspected export | `[UPDATE]` |
| App-imported Drive chunks | 20 at that check | `[UPDATE]` |
| FAISS vectors | `[NOT VERIFIED]` | `[UPDATE]` |
| BM25 documents | `[NOT VERIFIED]` | `[UPDATE]` |

**Important status note:** The last diagnostic supplied during development showed 150 PDF files downloaded, but no `pdf_document_catalog.csv`, zero extracted PDF contents, zero PDF source URLs represented in chunks, 20 Builder chunks, and an inspected `kb_export.json` with an empty `chunks` array. Therefore, the 150+ PDF knowledge-base target was not yet proven at that point. Re-run the PDF extraction, source-attributed chunking, index-building and export stages, then replace the “Final verified result” values above with actual outputs. Do not claim 150+ indexed sources based only on the download-folder file count.

### Expected Google Drive output

```text
MyDrive/FranchiseOps_AI/
├── rag_pdfs/                         # Downloaded public PDF files
├── manifest.json                     # Crawl/download metadata
├── source_inventory.csv              # Seed/source inventory (when generated)
├── pdf_document_catalog.csv          # PDF validation and extraction results
├── synthetic_kb/
│   ├── franchise_kb.jsonl             # Project SOP/sample knowledge documents
│   ├── pdf_extracted.jsonl            # Extracted, source-attributed PDF pages
│   └── all_chunks.jsonl               # Combined chunk corpus for indexing/import
├── faiss_indexes/                    # Dense FAISS index and metadata
├── bm25_indexes/                     # BM25 index artefact
└── kb_export.json                     # App handoff corpus (format kb-v1)
```

The exact set of files may vary by notebook version. `pdf_document_catalog.csv` and `all_chunks.jsonl` should be inspected to verify extraction and source provenance. `kb_export.json` should contain a non-empty `chunks` array before it is treated as a populated corpus export.

## 8. Copilot teaching: 50+ unanswered questions

The application includes an unanswered/weak-answer workflow and an Admin teaching interface. The supplied notebook also writes a `taught_questions.csv` bank containing **60 question/answer/keyword rows** for import or teaching. A prepared CSV is not, by itself, proof that 50 questions were discovered as unanswered and successfully taught during the milestone run.

### Required workflow

1. Ask genuine franchise-domain questions in the Copilot or paste the test batch into **Admin → Teach Copilot → Test them now**.
2. Collect questions that produce “I do not know”, a weak match or an unrelated answer. Avoid duplicate/rephrased copies of the same question when counting unique gaps.
3. Add a grounded answer and useful keywords in **Admin → Teach Copilot**, or import the supplied CSV if the import option is available.
4. For document/policy questions, add the supporting public document to the RAG corpus instead of forcing an unsupported static answer.
5. Re-ask the question and confirm that the response is relevant and cites the taught answer or knowledge-base source appropriately.
6. Review **Admin → Copilot insights**. Record the actual discovered, open and taught counts. The milestone evidence must show **50 / 50** taught questions.

### Question coverage in the supplied bank

The 60 prepared entries cover account/login help, roles and permissions, outlet operations, reports, data feeds, approvals, Digital Twin, FSSAI, food hygiene, HACCP, GST, labour compliance, HR processes, franchise agreements, inventory, customer service, marketing and operational procedures.

**Final milestone value:**

- Unique unanswered questions found: `[UPDATE FROM COPILOT INSIGHTS]`
- Unique questions taught: `[UPDATE FROM COPILOT INSIGHTS]`
- Teach Copilot counter: `[TARGET: 50 / 50]`
- Post-teaching answer rate: `[UPDATE WITH MEASURED VALUE]`

## 9. Simulator improvements (Milestone 3)

The six franchise simulators are implemented in `sim_fr.py`. They read their starting values from the application database, expose assumptions, use configurable random seeds and return uncertainty ranges. Their purpose is to run what-if scenarios without mutating live operational data.

| Simulator | Improvements implemented | Outputs / evidence to capture |
|---|---|---|
| **Inventory** | Added weekday/weekend and month-end demand seasonality; stochastic supplier delivery delays; compares three policies—Continuous Review `(s, Q)`, Periodic Review `(R, s, S)` and Dynamic Demand-Responsive Buffer `(s_t, S_t)`—under shared random paths. | Stock-out probability, expected lost sales, volume fill rate, cycle service level, policy cost comparison and P10–P90 ranges. |
| **Workforce** | Models attrition across three tenure bands (`<1 year`, `1–3 years`, `>3 years`); stochastic hiring lead time; compares overtime/temp coverage with permanent hiring. | Leavers and cost by tenure band, uncovered hours, hiring lead time, hiring-vs-overtime cost comparison and P10–P90 ranges. |
| **Outlet P&L** | Added price elasticity (baseline elasticity −1.2), separate wage and rent inflation inputs, operating-cost decomposition and best/base/worst-case scenarios. | Profit and cost impact, loss-making outlets and P10–P90 profit range. |
| **Marketing** | Added diminishing returns using a power-law response, campaign fatigue/wearout, channel-level conversion/ROI simulation and constrained budget reallocation. | ROI by channel, conversion response, recommended allocation and P10/P50/P90 ROI ranges. |
| **Sentiment** | Extracts and ranks root complaints from feedback; simulates the effect of resolving the top complaint on detractors, CSAT and repeat visits. | Top complaint, estimated CSAT change, repeat-visit uplift, revenue impact and P10/P50/P90 ranges. |
| **Audit** | Estimates next-audit failure probability from open findings/status; models remediation time by finding category and expected fines. | Expected failures, compliance days, fine exposure and P10–P90 ranges. |

The simulators use Monte Carlo paths (100 paths in the documented implementation) and configurable seeds for repeatable results. Their assumptions are displayed in the simulator UI. Use the same scenario inputs and random seed for before/after comparisons wherever possible.

### Before-and-after evidence

Add the requested screenshots below. Use the same input values where possible so the comparison demonstrates the new model behaviour rather than a different scenario.

#### Inventory simulator

**Before**  

<img width="1600" height="833" alt="image" src="https://github.com/user-attachments/assets/1acd7567-fbba-4548-9b6e-db932916dab6" />

**After** 

<img width="1600" height="835" alt="image" src="https://github.com/user-attachments/assets/4968da0f-9cfd-487d-b84b-d6cead3dc72b" />


#### Workforce simulator

**Before**  

<img width="1600" height="833" alt="image" src="https://github.com/user-attachments/assets/7dd80469-09af-43c1-bba8-3ee8e0a9d6f1" />

**After**  

<img width="1600" height="843" alt="image" src="https://github.com/user-attachments/assets/44a50714-e34c-4387-8771-88e58b31b324" />


#### Outlet P&L simulator

**Before**  

<img width="1600" height="837" alt="image" src="https://github.com/user-attachments/assets/6993e984-5fac-4b4c-ba55-c547157e19a0" />

**After**  

<img width="1600" height="831" alt="image" src="https://github.com/user-attachments/assets/20c91928-f15f-42af-8e4e-9a7f0105b4a5" />


#### Marketing simulator

**Before**  

<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/10890c35-9094-44ec-a5e0-61cfb2100f6d" />

**After**  

<img width="1600" height="836" alt="image" src="https://github.com/user-attachments/assets/94598e27-43e1-4892-979a-735ca95b56cb" />


#### Sentiment simulator

**Before**  

<img width="1600" height="832" alt="image" src="https://github.com/user-attachments/assets/7830666c-adfc-4199-bf58-e43559b2abc3" />

**After**  

<img width="1600" height="839" alt="image" src="https://github.com/user-attachments/assets/c3bd5d8a-1b94-4de4-99f3-19ce66a1403f" />


#### Audit simulator

**Before**  

<img width="1600" height="833" alt="image" src="https://github.com/user-attachments/assets/874e3d4a-3bda-4b58-99cc-d7582c71c8ee" />

**After** 

<img width="1600" height="835" alt="image" src="https://github.com/user-attachments/assets/cf9ae0ea-c77b-4318-a07f-4e58d687c464" />


## 10. Milestone 2 integration and access control

The final application integrates the login, account/profile, role-based access and administrative controls into the FranchiseOps application.

### Roles

| Role | Access summary |
|---|---|
| **Admin** | Full platform access, system checks, user/role management, autonomy policy, Copilot teaching and administration. |
| **Franchise Owner** | Cross-outlet operational visibility, action approvals and weekly digest; no unrestricted Admin-only controls. |
| **Store Manager** | Outlet-scoped operational views and ticket resolution for the assigned outlet. |
| **Staff** | Limited outlet-scoped access and permitted Copilot/notification functions. |
| **Auditor** | Read-oriented audit and compliance visibility as configured by the app's role/capability map. |
| **Customer** | Limited Copilot/profile access where the Customer role is enabled. |

The exact menu and capabilities are enforced by the application RBAC configuration, not just by hiding frontend buttons. A user attempting an unauthorised action should receive HTTP 403.

### Account and session controls

- Self-registration must not grant privileged roles; privileged roles are assigned by an Admin.
- Administrator lock/deactivation should prevent login and invalidate current sessions.
- Locked accounts must not be able to bypass the lock using security-question or OTP recovery; only an Admin should unlock the account.
- Role changes are enforced by backend capability checks.
- Do not include real demo credentials, temporary passwords, OTPs or security answers in this README or in Git history.

## 11. Screenshots required for evaluation

Store screenshots under `Milestone3/screenshots/` and keep filenames consistent with the placeholders above.

### RAG Builder: 150+ sources and chunks

Capture the Builder completion/evidence output showing unique extracted PDFs, distinct PDF source URLs represented in chunks, total chunks, and index creation. A screenshot showing only downloaded PDF files is insufficient.

<img width="1572" height="967" alt="image" src="https://github.com/user-attachments/assets/f643b078-8837-41c8-974b-0b4f86e3b6b0" />

### Teach Copilot: 50 / 50

Capture Admin → Teach Copilot or Copilot insights showing the verified taught count of 50 / 50 (or greater, if displayed).

<img width="1576" height="927" alt="image" src="https://github.com/user-attachments/assets/bf33af49-0b66-481b-a316-bfb7f41c1b03" />

<img width="1537" height="937" alt="image" src="https://github.com/user-attachments/assets/4195ec3e-7c83-40c8-b9ac-542218194358" />


### Copilot answer with citations [1] and [2]

Capture a relevant knowledge question and its answer with the expanded Sources section. The source titles/URLs must correspond to actual retrieved documents, and the cited passages must support the answer.

<img width="1527" height="512" alt="image" src="https://github.com/user-attachments/assets/8872a21e-8362-4e87-bee4-58aaad88a463" />

### Admin → Copilot insights


## References

- Infosys Springboard Internship 7.0, Batch 4 — *Milestone 3: Knowledge Base, AI Copilot, Presentation and Demo — FranchiseOps AI*.
- *FranchiseOps AI: Project Explained — what it is, how it works, and why it is agentic AI*.
- Public sources collected by the RAG Builder; the final list of successfully processed documents should be taken from `source_inventory.csv`, `pdf_document_catalog.csv` and the generated source manifest after a successful run.

---

## Contributions

- **Dhanush R** - Added 150+ Websites and PDFs for RAG Sources
- **Ajay D** - Finetuned AI Copilot
- **Chokkarapuwar Sujal** - Improved the Agents Simulation and integrated Milestone-2
- **Varsha Spoorthi Shree Narravula** - Taught 50+ questions that were unanswered by Copilot
