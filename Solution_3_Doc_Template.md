# Proposal: Intelligent Docx Template Engine

## 1. System Architecture

The system is designed as a 3-stage pipeline: **Ingestion & Analysis** -> **Template Registry** -> **Production Generation**.

### Phase 1: Ingestion (AI-Powered Template Discovery)
Instead of forcing users to manually add `{{brackets}}` to Word docs (which is hard for non-tech users), we use a **One-Shot LLM Analysis** step.

1.  **User Upload**: Uploads a standard filled `Sample_Offer_Letter.docx`.
2.  **Parsing**: System converts DOCX -> Markdown/Text (using `python-docx` or `pypandoc`) to preserve structure.
3.  **AI Analysis (The Brain)**:
    *   **Input**: The text content of the document.
    *   **Prompt**: "Identify all dynamic entities in this contract (Names, Dates, Amounts, Addresses). Map them to a standard variable naming convention (e.g., `candidate_name`, `start_date`). Return a JSON map of `Original Text` -> `Variable Key`."
4.  **Template Compilation**: The system uses the JSON map to search-and-replace the text in the original DOCX with Jinja2 tags (e.g., replaces "John Doe" with `{{ candidate_name }}`).
5.  **Output**: A saved `Master_Template.docx` and a `schema.json` defining the fields.

### Phase 2: User Verification (Frontend)
The user sees a "Detected Fields" review screen.
*   *candidate_name*: "John Doe" (Detected)
*   *salary_amount*: "$120,000" (Detected)
*   *User Action*: Confirm or Rename fields.

### Phase 3: Bulk Generation Engine
1.  **Input Source**: User uploads an Excel/CSV file where column headers match the `schema.json` keys (e.g., Column A = `candidate_name`).
2.  **Processing Loop**:
    *   Load `Master_Template.docx` using `docxtpl` (Python library).
    *   For each row in Excel:
        *   Render context: `{ 'candidate_name': 'Alice Smith', ... }`.
        *   Save formatted DOCX.
        *   Convert to PDF (using headless LibreOffice or CloudConvert API).
3.  **Delivery**: Zip file download.

## 2. Technology Stack
*   **Template Logic**: `docxtpl` (Python) - Allows Jinja2 syntax inside Word documents.
*   **PDF Conversion**: `LibreOffice` (Headless mode) - Free, reliable local conversion.
*   **AI Analysis**: `Gemini 1.5 Pro` - chosen for large context window (can read whole contracts) and strong entity extraction capabilities.

## 3. Why this wins?
*   **Zero-Config**: Users upload *existing* used files, not special templates.
*   **Scalable**: `docxtpl` is extremely fast (milliseconds per doc).
*   **Flexible**: Supports logic (if/else) inside Word docs if advanced users want to edit the Jinja2 tags manually.
