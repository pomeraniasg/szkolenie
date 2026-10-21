# PRD — Hardware Service Decision Copilot

> Optional course example. This document does not replace `docs/PRD.md`, which the course group creates and which is the binding PRD.

---

## 1. Executive Summary

Hardware Service Decision Copilot is an MVP web application that helps customer support and hardware service staff decide on complaints and returns of electronic equipment. Staff submit a case through a form with one photo of the item; the system analyses the photo, applies the Company's complaint or return rules, and opens a chat where staff receive a justified recommendation and can ask follow-up questions. The product is advisory: staff make and record the final decision outside the application.

---

## 2. Problem Statement

Staff currently read the Company's complaint and return rules manually, inspect the item description and photo, and decide case by case. Results vary between staff members, justifications are rarely written down, and borderline cases take a long time because staff must search several documents. There is no single place where the item condition, the applicable rule and the reasoning are recorded together.

The application must give staff a consistent, justified first assessment in a few minutes, using the same rules every time.

---

## 3. Users / Personas

**Persona 1 — Support Agent (primary user)**
- Works in customer support and handles incoming complaints and return requests.
- Wants a quick, consistent recommendation, plus a clear explanation to relay to the customer.
- Expects the recommendation to name the rule it relies on and to say when a case needs a human decision.

**Persona 2 — Hardware Service Technician (secondary user)**
- Works in the hardware service team and checks items physically when a case reaches service.
- Wants to see how the system read the photo, so they can confirm or reject the assessment.
- Expects the system to flag damage it cannot classify instead of guessing.

Both personas use the same screens. The MVP has no roles or permissions.

---

## 4. Main Flows

### 4.1 Happy path — complaint

1. Staff opens the case form and selects "Complaint" as the case type.
2. Staff selects the equipment category, enters the model name, selects the purchase date, writes the complaint reason, and uploads one photo of the item.
3. Staff selects "Submit". The system validates all fields (see AC-01 to AC-08).
4. The system compresses the photo and sends it to the image analysis step with the complaint prompt. The screen shows a loading state.
5. The system passes the image description, the form data and the complaint rules to the decision agent.
6. The system opens the chat. The first message from the system contains a greeting, the decision category, the justification, the rule references and the suggested next steps.
   > **Revision 2026-10-21 (R1):** The loading state stays on screen ("Preparing the decision…") until the complete decision has been checked against AC-17 to AC-21. Only then does the chat open, with the full first message. No part of the decision is shown while it is being prepared (consistent with 4.5 step 3).
7. Staff reads the first message and may send follow-up messages (see 4.4).

### 4.2 Happy path — return

1. Staff opens the case form and selects "Return".
2. Staff fills in the same fields. The complaint reason is optional for a return.
3. Staff selects "Submit". The system validates the fields.
4. The system compresses the photo and sends it to the image analysis step with the return prompt. The screen shows a loading state.
5. The system passes the image description, the form data and the return rules to the decision agent.
6. The system opens the chat with the first message, which states whether the item shows no signs of use and whether it can be returned to stock and resold.
   > **Revision 2026-10-21 (R1):** As in 4.1 step 6, the chat opens only after the complete decision has been checked; nothing of the decision is shown before.
7. Staff may send follow-up messages (see 4.4).

### 4.3 Alternative path — unreadable photo

1. After submit, the image analysis step cannot read the photo (too dark, blurred, item not visible, or a file that is not an image).
2. The system stays on the form, keeps all entered values, and shows the message: "The photo cannot be analysed. Upload a clear photo of the whole item and submit again."
3. No decision is created and no chat is opened.

### 4.4 Alternative path — follow-up questions in chat

1. Staff types a message in the chat input and selects "Send".
2. The agent answers using the full conversation history: the form data, the image description, the first decision message and all earlier messages.
3. If the staff member supplies new facts (for example, "The customer says the screen was cracked on arrival"), the agent takes them into account and states whether the decision changes. The agent does not silently change the first decision; it posts a new message that states the change and the reason.
4. If the question is outside the scope of the Company's rules or the product (see section 11), the agent declines politely and says what it can help with.

### 4.5 Alternative path — analysis or agent failure

1. The image analysis step or the decision agent does not respond in time or returns an error.
2. The system shows the message: "The analysis failed. Your entries are saved on this screen. Try again."
3. Staff selects "Try again". The system repeats the failed step with the same data. No partial decision is shown.
   > **Revision 2026-10-21 (R2):** If the image analysis failed, "Try again" repeats the analysis with the same entries and photo. If the decision agent failed, "Try again" repeats only the decision, using the image description already obtained; the photo is not needed again.

### 4.6 Alternative path — ambiguous decision

1. The image analysis reports damage it cannot classify, or the agent cannot match the case to a single rule.
2. The decision category is set to ESCALATE (see section 11).
3. The first message explains which information is missing and recommends passing the case to a human reviewer.

### 4.7 Starting a new case

1. Staff selects "New case" at the top of the chat screen.
2. The system discards the current chat and opens an empty form.
3. The system shows a confirmation dialog before discarding a chat that has at least one follow-up message.

---

## 5. User Stories

- **US-01 (happy path, complaint):** As a support agent, I want to submit a complaint with a photo and receive a justified recommendation, so that I can answer the customer quickly and consistently.
- **US-02 (happy path, return):** As a support agent, I want to submit a return request and learn whether the item can go back to stock, so that I can decide without reading the full return policy.
- **US-03 (error case, photo):** As a support agent, I want the system to tell me when the photo is unusable and keep my form entries, so that I can retake the photo without re-entering the case.
- **US-04 (error case, ambiguity):** As a hardware service technician, I want the system to escalate cases it cannot classify and list what is missing, so that I do not act on a guess.
- **US-05 (session/state):** As a support agent, I want to ask follow-up questions in the same chat and have the agent remember the case details, so that I do not have to repeat them.
- **US-06 (session/state):** As a support agent, I want to start a new case and be warned before losing an open chat, so that I do not discard work by accident.

---

## 6. Acceptance Criteria

### Form

- **AC-01:** The case-type field offers exactly two options: "Complaint" and "Return". No other value can be selected.
- **AC-02:** The equipment category field offers the categories listed in section 8 (Functional constraints), and no others.
- **AC-03:** The model name field accepts 2 to 100 characters. A value outside this range disables "Submit" and shows "Enter 2–100 characters".
- **AC-04:** The purchase date field rejects a date later than today's date and shows "Purchase date cannot be in the future".
- **AC-05:** For a Complaint, the reason field is required and accepts 10 to 2000 characters. Below 10 characters, "Submit" is disabled and the field shows "Describe the problem (at least 10 characters)".
- **AC-06:** For a Return, the reason field is optional and accepts up to 2000 characters.
- **AC-07:** The photo field requires exactly one file. Submitting without a file shows "Upload a photo of the item" and does not send the request.
- **AC-08:** The "Submit" control is disabled while a request is in progress, so a second submit cannot be sent for the same case.

### Image Upload and Analysis

- **AC-09:** The photo field accepts JPG, PNG and WEBP files. Any other file type is rejected with "Unsupported file type. Use JPG, PNG or WEBP".
- **AC-10:** A file larger than 10 MB is rejected before upload with "File exceeds the 10 MB limit".
- **AC-11:** Before analysis, the backend compresses the photo to at most 1 MB while keeping the longest side at least 1024 px. If the photo cannot meet both limits, the system rejects it with the unreadable-photo message from section 4.3.
- **AC-12:** For a Complaint, the image analysis returns a structured description containing: whether damage is visible (yes / no / unclear), the damage type, the affected area, and the likely cause. Each field is present, even when the value is "unknown".
- **AC-13:** For a Return, the image analysis returns a structured description containing: whether any sign of use is visible (yes / no / unclear), the visible condition of the item, and whether the item appears complete (yes / no / unclear).
- **AC-14:** The complaint analysis and the return analysis each use a separate instruction, selected by the case type. A Return case never uses the complaint instruction, and a Complaint case never uses the return instruction.
- **AC-15:** When the analysis returns "unclear" for a field that decides the outcome, the decision category is ESCALATE.

### AI Decision

- **AC-16:** The decision agent receives three inputs: the form data, the structured image description, and the rule document for the selected case type. The agent does not receive the rule document for the other case type.
- **AC-17:** Every decision has exactly one category: ACCEPT, REJECT or ESCALATE. No other category is shown.
- **AC-18:** The first chat message states the category in the first two sentences, in bold.
- **AC-19:** The first chat message includes at least one reference to a numbered rule from the rule document. A decision without a rule reference is shown as ESCALATE.
- **AC-20:** The first chat message includes a "Next steps" list with 2 to 5 items.
- **AC-21:** The first chat message includes the disclaimer: "This is a recommendation. The final decision is recorded by a staff member." The disclaimer appears in every decision message.
- **AC-22:** When the decision agent cannot produce a valid decision, the category is ESCALATE and the message says that a human reviewer must decide.

### Chat

- **AC-23:** The first chat message is shown as the first message from the system, before any user message, with no action needed from the user.
- **AC-24:** The chat input accepts 1 to 2000 characters. An empty message cannot be sent.
- **AC-25:** Each user message receives an agent reply within 30 seconds, or the reply shows "The reply failed. Try again" with a retry control.
- **AC-26:** The agent replies in Polish, using the same language as the chat, unless the case data is in another language and the user asks otherwise.
- **AC-27:** Each agent request includes all previous messages in the session, the form data, the image description and the first decision message.
- **AC-28:** If a user message adds facts that change the decision, the agent posts a new message that states the new category and the reason. The first message stays unchanged in the chat history.
- **AC-29:** The chat shows the timestamp of each message.

### Session

- **AC-30:** A session starts when the user submits the form and ends when the user selects "New case" or closes the browser tab.
- **AC-31:** The session data (form values, image description, decision and chat messages) is kept only in the browser for the duration of the session. Reloading the page ends the session and returns the user to an empty form.
- **AC-32:** Selecting "New case" with at least one follow-up message shows the confirmation "Discard this chat? The decision cannot be recovered." Choosing "Cancel" keeps the chat unchanged.

### General

- **AC-33:** All UI text is in Polish, including validation messages, loading messages, error messages and button labels.
- **AC-34:** The UI works on screens from 360 px to 1440 px wide without horizontal scrolling.
- **AC-35:** The application does not store the uploaded photo after the analysis step completes. The photo is not shown again after the first decision.
- **AC-36:** Staff can complete the complaint and return flows without any operator intervention.

---

## 7. Out of Scope

- **Authentication and user accounts:** No login, no roles, no personal workspace in the MVP.
- **Customer and purchase history:** No lookup of existing customers or purchases from a database.
- **Session persistence:** No saving of sessions, decisions or actions to a database. Session data lives only in the browser (AC-31).
- **Audit trail:** No log of who viewed or decided a case.
- **Knowledge base (RAG):** The agent uses only the two rule documents from section 8. No product specifications or external knowledge base.
- **Multiple photos:** Only one photo per case.
- **Video or document upload:** Only images.
- **Administrative UI:** No screens for editing rules, categories or prompts.
- **Notifications:** No email, SMS or push notifications.
- **Integrations:** No connection to order systems, payment systems, shipping carriers or ticketing tools.
- **Automatic actions:** The application does not issue refunds, return labels, service orders or customer messages.
- **Multilingual UI:** Polish only.
- **Mobile apps:** Browser only. Native apps are not in scope.
- **Legal advice:** The agent does not give legal opinions beyond the rule documents.

---

## 8. Constraints

### Business

- The decision is advisory. The application never records a final decision and never changes stock, order or payment status.
- The Company's rules are the only basis for a decision. Where the rules do not cover a case, the decision is ESCALATE.
- The example rule documents in section 8 are placeholders for the MVP and must be replaced by the Company's approved texts before real use.
- The application must not promise a refund or a replacement to the customer. It may only state the recommended outcome and the next steps.

### Functional

- **Case types:** Complaint, Return.
- **Equipment categories (example list):** Laptop, Smartphone, Tablet, Smartwatch, Headphones, Speaker, Monitor, Desktop computer, Printer, Other electronics.
- **Model name:** 2 to 100 characters.
- **Purchase date:** calendar date, not later than today.
- **Complaint reason:** 10 to 2000 characters, required for Complaint.
- **Return reason:** up to 2000 characters, optional for Return.
- **Photo:** exactly one file; JPG, PNG or WEBP; up to 10 MB before compression; compressed to at most 1 MB before analysis (AC-11).
- **Chat message:** 1 to 2000 characters.
- **Language:** Polish for UI, agent replies and validation messages.
- **Browsers:** current versions of Chrome, Firefox, Safari and Edge on desktop and mobile.
- **Viewport:** 360 px to 1440 px wide.
- **Analysis timeout:** a single analysis or agent request that takes longer than 30 seconds is treated as failed (section 4.5).

### External document / data references

| Document name | File path | When it is used |
|---|---|---|
| Complaint Rules (example) | `docs/examples/rules/complaint-rules.md` (proposed) | Loaded as rules for the decision agent when the case type is Complaint |
| Return Rules (example) | `docs/examples/rules/return-rules.md` (proposed) | Loaded as rules for the decision agent when the case type is Return |
| Complaint analysis instruction | Defined in the ADR | Sent to the image analysis step for Complaint cases |
| Return analysis instruction | Defined in the ADR | Sent to the image analysis step for Return cases |

The rule documents are written in Polish, numbered, and contain at least these sections: eligibility window, condition requirements, excluded cases, and required customer information. Their content in the example is fictional and marked as such.

---

## 9. UI Description (wireframe level)

### 9.1 Case form screen

**Layout:** A title, a case-type selector at the top, then the fields in this order: equipment category, model name, purchase date, reason, photo upload, and a "Submit" button at the bottom.

**Elements:**
- **Case type selector:** two options, "Complaint" and "Return". Selecting one changes the label of the reason field and its required state (AC-05, AC-06).
- **Category:** dropdown with the list from section 8.
- **Model name:** single-line text field.
- **Purchase date:** date picker. Future dates are not selectable.
- **Reason:** multi-line text field with a character counter. Shows "Required" for Complaint and "Optional" for Return.
- **Photo upload:** a button and a drop area. After selecting a file, the screen shows a thumbnail, the file name and a "Remove" control.
- **Submit:** disabled until all required fields are valid. Changes to a loading state after selection (see 9.2).

**Error states:** each invalid field shows its message directly below it. The first invalid field receives focus on a failed submit.

**Empty state:** all fields empty; "Submit" disabled.

### 9.2 Loading state

- Shown after "Submit" is selected.
- The form fields are read-only. A progress message reads "Analysing the photo…", then "Preparing the decision…".
- The "Submit" control is replaced by a disabled "Processing…" label.
- A "Cancel" control returns to the form with entries kept. Cancelling discards the in-progress request.
- > **Revision 2026-10-21 (R1):** The loading state lasts until the decision is complete and checked. The decision is not displayed progressively; the chat opens with the full first message. Follow-up replies in the chat may still appear progressively (9.5).
- > **Revision 2026-10-21 (R2):** Cancel during "Preparing the decision…" returns to the form with the text entries kept and the photo field empty, because the photo is released once the analysis has completed (AC-35).

### 9.3 Analysis failure state

- Shown when the image analysis or the decision agent fails (section 4.5).
- The message appears above the form fields; all entries remain.
- Controls: "Try again" (repeats the last failed step) and "Edit case" (returns to the editable form).
- > **Revision 2026-10-21 (R2):** After a decision failure, "Edit case" keeps all text entries, but the photo field is empty and a photo must be selected again, because the photo is released once the analysis has completed (AC-35). After an analysis failure the photo is kept.

### 9.4 Unreadable photo state

- Shown when the photo fails the readability check (section 4.3).
- The message appears below the photo field. The photo field is cleared so the user can select another file.

### 9.5 Chat screen

**Layout:** A header with the case type, the equipment category and the model name, and a "New case" control. Below it, a scrollable message list. At the bottom, a text input with a "Send" button.

**Messages:**
- **First system message:** a decision message with greeting, decision category in bold, justification, rule references and next steps. It is the first item in the list and cannot be removed.
  > **Revision 2026-10-21 (R1):** This message appears complete when the chat opens; it is never streamed or replaced. Order: greeting, then the category in bold in the second sentence, then justification, rule references, "Next steps" and the disclaimer (AC-18, section 11). Any later message that changes the decision follows the same structure and carries the disclaimer (AC-21, AC-28).
- **User messages:** shown on the right, with a timestamp.
- **Agent messages:** shown on the left, with a timestamp. Markdown formatting is allowed (bold, bullet lists, numbered lists).

**Interactive elements:**
- **Send:** sends the input text. Disabled when the input is empty or a reply is pending.
- **New case:** see flow 4.7.
- **Retry (on a failed reply):** repeats the last user message.

**Empty state:** not applicable. The chat always starts with the first system message.

**Loading state:** a typing indicator below the last message while the agent replies.

**Error state:** a failed reply shows "The reply failed. Try again" below the user message that caused it, with a "Retry" control.

**Navigation:** the chat screen is reached only by submitting the form. The browser back button returns to the form with entries kept, and the chat remains available until the page is reloaded. Reloading the page returns to an empty form (AC-31).

---

## 10. User Flow Diagram

```mermaid
flowchart TD
    A[Open case form] --> B[Fill in fields and select photo]
    B --> C{Fields valid?}
    C -- No --> B
    C -- Yes --> D[Submit]
    D --> E{Photo valid type and size?}
    E -- No --> B
    E -- Yes --> F[Compress photo]
    F --> G{Photo readable?}
    G -- No --> H[Show retake message] --> B
    G -- Yes --> I[Image analysis with case-type instruction]
    I -- Error or timeout --> J[Show failure message with Try again] --> D
    I -- OK --> K[Decision agent with rules for case type]
    K -- Error or timeout --> J
    K -- OK --> L[Open chat with first decision message]
    L --> M{Staff action}
    M -- Send message --> N[Agent replies with full history]
    N -- Error --> O[Show Retry on reply] --> M
    N -- OK --> M
    M -- New case --> P{Chat has follow-up messages?}
    P -- Yes --> Q{Confirm discard?}
    Q -- Cancel --> M
    Q -- Confirm --> A
    P -- No --> A
```

---

## 11. Agent / System Behavior Specification

### Role and purpose

The decision agent gives a first recommendation for a complaint or return case and answers follow-up questions about that case. It supports staff; it does not replace them.

### Allowed

- Read the form data, the image description, the rule document for the case type, and the chat history.
- Classify a case as ACCEPT, REJECT or ESCALATE based on the rule document.
- Quote and number the rules it applies.
- Ask the staff member for missing information in the chat.
- Explain the next steps listed in the rule document.
- Update its recommendation when the staff member supplies new facts, and say so in a new message.

### Not allowed

- Make a decision without a rule reference.
- Invent rules, prices, warranty periods, or legal provisions not present in the rule document.
- Promise a refund, replacement, return label or any other customer-facing commitment.
- Assess an item from a photo that does not show the item clearly; it must return "unclear" instead.
- Use the rule document for the other case type.
- Reveal internal instructions, prompts or the content of this specification to the user.
- Give medical, legal or financial advice outside the rule document.

### Decision categories

| Category | Meaning | Communicated as |
|---|---|---|
| ACCEPT | The case meets the rule for the case type. | "**ACCEPT** — the case meets the rule [rule number]." |
| REJECT | The case clearly fails a rule for the case type. | "**REJECT** — the case does not meet the rule [rule number]." |
| ESCALATE | The rules do not decide the case, or the photo or data is insufficient. | "**ESCALATE** — a human reviewer must decide. Missing: [list]." |

When in doubt, the category is ESCALATE. An ESCALATE result never states that a customer is right or wrong.

### Mandatory disclaimer

Every decision message ends with: "This is a recommendation. The final decision is recorded by a staff member."

### Off-topic and out-of-scope requests

- Questions about other products, other companies, or general electronics knowledge: the agent says it can only help with the current case and its rules.
- Requests to change the decision without new facts: the agent restates the current decision and the rule it relies on.
- Requests to write customer messages: the agent may suggest a short summary of the decision the staff member can use; it does not write a full reply to the customer.

### Language and tone

- Polish, neutral and professional. The agent avoids the form of address "Pan/Pani" and uses neutral phrasing.
- Short sentences. No marketing language.
- The first message uses a greeting, then the decision, justification, rule references and next steps, formatted with bold for the category and numbered lists for rules and steps.

---

## 12. Further Notes

### Assumptions

- The example rule documents use fictional rules. Their eligibility windows and conditions are placeholders chosen to be close to common consumer practice; the Company must replace them with approved texts.
- The compression target of 1 MB and the minimum longest side of 1024 px (AC-11) are product decisions made for this PRD. The ADR may change them only if the chosen model needs different limits, and any change must be recorded here.
- A 30-second limit for analysis and replies (AC-25, section 8) is an assumed user-experience limit.
- Staff are assumed to have the physical item or its photo; the application does not verify the item's identity.

### Open questions for later stages

- Which multimodal model and thinking model will be used. Deferred to the ADR.
- Whether analysis results should be shown to staff in full or only as the first message summary. Current assumption: a summary in the first message only.
- Whether session persistence and customer history move into the MVP. Deferred to a later phase, as stated in the brief.
- Whether the Company wants an audit trail for compliance. Not in MVP; to be decided with the business owner.
- *(Added 2026-10-21, R4, decided R6)* **Analysis visibility:** Decided for the MVP: no analysis panel. The image description is not shown as a separate panel; the current assumption stands (a summary in the first message only). The panel idea (ADR-004 section 9) is deferred until hardware service technicians ask for it after the first use.
- *(Added 2026-10-21, R4)* **Provider data retention:** The photo and form data are sent to OpenRouter and the model provider behind it. Which retention settings are required, and whether real customer data may be used before they are confirmed, is to be decided with the business owner (ADR-000 OI-000-2). Until then the application is used with synthetic data only.
- *(Added 2026-10-21, R4, decided R6)* **Photos smaller than 1024 px:** Decided: AC-11 is read literally. A photo whose longest side is already below 1024 px is rejected with the unreadable-photo message (ADR-003, OI-003-1). Upscaling would add no detail.
- *(Added 2026-10-21, R4)* **Binding PRD path:** The header refers to `docs/PRD.md`, which does not exist yet. Recommendation: copy this document to `docs/PRD.md` before implementation (ADR-000, OI-000-7).

### Clarifications (added 2026-10-21, R3)

- **"Today" (AC-04, section 8):** today's date in the `Europe/Warsaw` time zone, for both the form and the server check.
- **Rule references (AC-19):** a numbered rule is a paragraph whose identifier has the form `R-<number>` in the rule document for the case type (for example `R-4`). Definition paragraphs numbered `1.1`, `1.2` and so on are not rule references.
- **Unreadable photo detection (4.3):** the image analysis reports explicitly whether the photo is usable; an unusable photo leads to 4.3, while a usable photo with an unclear outcome-deciding field leads to ESCALATE (AC-15, 4.6). AC-12 and AC-13 are unchanged; the additional field is technical and defined in ADR-002.

---

## 13. UI copy (Polish)

> **Accepted for the MVP as the application copy (R6); wording may be tuned after the first use.** Added 2026-10-21 (R5). AC-33 requires Polish UI text, but the sections above quote English text. This table gives the exact Polish strings the application uses. The English quotes in sections 4 to 11 identify each string; the Polish column is what the user sees. Placeholders in braces are filled by the application.

### 13.1 Messages quoted in this PRD

| Where | English reference | Polish UI text |
|---|---|---|
| 4.3, 9.4 | The photo cannot be analysed. Upload a clear photo of the whole item and submit again. | Nie można przeanalizować zdjęcia. Prześlij wyraźne zdjęcie całego sprzętu i wyślij zgłoszenie ponownie. |
| 4.5, 9.3 | The analysis failed. Your entries are saved on this screen. Try again. | Analiza nie powiodła się. Wprowadzone dane są zachowane na tym ekranie. Spróbuj ponownie. |
| AC-25, 9.5 | The reply failed. Try again | Nie udało się uzyskać odpowiedzi. Spróbuj ponownie. |
| AC-32, 4.7 | Discard this chat? The decision cannot be recovered. | Odrzucić ten czat? Decyzji nie będzie można odzyskać. |
| AC-21, 11 | This is a recommendation. The final decision is recorded by a staff member. | To jest rekomendacja. Ostateczną decyzję zapisuje pracownik. |
| 9.2 | Analysing the photo… | Trwa analiza zdjęcia… |
| 9.2 | Preparing the decision… | Trwa przygotowanie decyzji… |
| 9.2 | Processing… | Przetwarzanie… |
| Server-side validation failure (no PRD text) | — | Nie udało się wysłać zgłoszenia. Sprawdź dane w formularzu i spróbuj ponownie. |

### 13.2 Validation messages

| Rule | English reference | Polish UI text |
|---|---|---|
| AC-03 | Enter 2–100 characters | Wpisz od 2 do 100 znaków |
| AC-04 | Purchase date cannot be in the future | Data zakupu nie może być z przyszłości |
| AC-05 | Describe the problem (at least 10 characters) | Opisz problem (co najmniej 10 znaków) |
| AC-05, AC-06 (over 2000) | — | Opis może mieć maksymalnie 2000 znaków |
| AC-07 | Upload a photo of the item | Dodaj zdjęcie sprzętu |
| AC-09 | Unsupported file type. Use JPG, PNG or WEBP | Nieobsługiwany typ pliku. Użyj JPG, PNG lub WEBP |
| AC-10 | File exceeds the 10 MB limit | Plik przekracza limit 10 MB |
| AC-02 (empty) | — | Wybierz kategorię sprzętu |
| Purchase date (empty) | — | Podaj datę zakupu |
| AC-24 (over 2000) | — | Wiadomość może mieć maksymalnie 2000 znaków |

### 13.3 Labels and controls

| Element | English reference | Polish UI text |
|---|---|---|
| Form title | — | Zgłoszenie reklamacji lub zwrotu |
| Case type field | Case type | Typ zgłoszenia |
| Case type options (AC-01) | Complaint / Return | Reklamacja / Zwrot |
| Category field | Equipment category | Kategoria sprzętu |
| Category placeholder | — | Wybierz kategorię |
| Model name field | Model name | Nazwa modelu |
| Purchase date field | Purchase date | Data zakupu |
| Reason field (Complaint) | Reason | Powód reklamacji |
| Reason field (Return) | Reason | Powód zwrotu |
| Reason state | Required / Optional | Wymagane / Opcjonalne |
| Character counter | — | {liczba}/2000 |
| Photo field | Photo | Zdjęcie sprzętu |
| Photo drop area | — | Przeciągnij zdjęcie tutaj lub wybierz plik |
| Photo select button | — | Wybierz plik |
| Photo remove control | Remove | Usuń |
| Submit button | Submit | Wyślij zgłoszenie |
| Cancel during loading | Cancel | Anuluj |
| Retry after failure | Try again | Spróbuj ponownie |
| Edit after failure | Edit case | Edytuj zgłoszenie |
| New case | New case | Nowa sprawa |
| Chat input placeholder | — | Napisz wiadomość… |
| Send | Send | Wyślij |
| Retry on a failed reply | Retry | Ponów |
| Typing indicator | — | Asystent pisze… |
| Discard dialog: cancel | Cancel | Anuluj |
| Discard dialog: confirm | — | Odrzuć czat |
| Chat header labels | Case type, category, model name | Typ zgłoszenia, Kategoria, Model |

### 13.4 Equipment categories (section 8)

| English | Polish UI text |
|---|---|
| Laptop | Laptop |
| Smartphone | Smartfon |
| Tablet | Tablet |
| Smartwatch | Smartwatch |
| Headphones | Słuchawki |
| Speaker | Głośnik |
| Monitor | Monitor |
| Desktop computer | Komputer stacjonarny |
| Printer | Drukarka |
| Other electronics | Inna elektronika |

### 13.5 Decision categories and decision message parts (section 11, AC-17 to AC-22)

| Category code | Polish display label | Second sentence of a decision message (bold label) |
|---|---|---|
| ACCEPT | AKCEPTACJA | **AKCEPTACJA** — sprawa spełnia zasadę {R-<numer>}. |
| REJECT | ODRZUCENIE | **ODRZUCENIE** — sprawa nie spełnia zasady {R-<numer>}. |
| ESCALATE | ESKALACJA | **ESKALACJA** — decyzję musi podjąć pracownik. Brakuje: {lista}. |

| Part | Polish UI text |
|---|---|
| Greeting (first sentence of the first message) | Dzień dobry. |
| Change notice (first sentence of a later decision message, AC-28) | Na podstawie nowych informacji rekomendacja się zmienia. |
| Heading: justification | Uzasadnienie |
| Heading: rule references | Zastosowane zasady |
| Heading: next steps (AC-20) | Kolejne kroki |
| Heading: missing information | Brakujące informacje |
| Disclaimer (last line, AC-21) | To jest rekomendacja. Ostateczną decyzję zapisuje pracownik. |

### 13.6 Fallback ESCALATE message (AC-22)

Used when the decision cannot be produced or fails the checks. The texts are fixed, not produced by the model.

- **Second sentence:** **ESKALACJA** — decyzję musi podjąć pracownik.
- **Missing information, when the system cannot name it:** Nie udało się przygotować pełnej rekomendacji na podstawie zasad. Sprawę musi ocenić pracownik.
- **Next steps (3 items, within AC-20):**
  1. Przekaż sprawę do decyzji pracownika serwisu lub przełożonego.
  2. Jeżeli to możliwe, poproś klienta o dodatkowe zdjęcie lub dokument.
  3. Nie informuj klienta o wyniku przed decyzją przełożonego.
- **Last line:** the disclaimer from 13.5.

### 13.7 Analysis summary labels (only if the proposal in section 12 is confirmed)

| Field | Polish label |
|---|---|
| Damage visible | Widoczne uszkodzenie |
| Damage type | Rodzaj uszkodzenia |
| Affected area | Obszar uszkodzenia |
| Likely cause | Prawdopodobna przyczyna |
| Signs of use | Oznaki użytkowania |
| Visible condition | Widoczny stan |
| Item complete | Kompletność |
| Values yes / no / unclear / unknown | tak / nie / niejasne / nieznane |

---

## Revision log

- **2026-10-21 R1** (documentation review, finding P1-1): The decision is no longer shown progressively. The loading state lasts until the complete decision has been checked; the chat then opens with the full first message (4.1 step 6, 4.2 step 6, 9.2, 9.5). Follow-up replies may still appear progressively. This replaces the earlier technical choice to stream the decision, which conflicted with 4.5 step 3, AC-18 and AC-19.
- **2026-10-21 R2** (finding P2-7): "Try again" repeats only the failed step (analysis or decision); the photo is released after a successful analysis, so "Edit case" or "Cancel" after that point needs a new photo (4.5, 9.2, 9.3). No AC changed.
- **2026-10-21 R3** (findings P1-2, P3): Clarifications for "today" (`Europe/Warsaw`), rule reference format (`R-<number>`) and unreadable-photo detection (section 12). AC-12 and AC-13 unchanged; the photo-usability field is defined in ADR-002.
- **2026-10-21 R4** (findings P3, P4): Open questions added on analysis visibility, provider data retention, photos below 1024 px and the binding PRD path (section 12). The header still names `docs/PRD.md`; copying this document there is a recommended follow-up for the user.
- **2026-10-21 R5** (findings P2-6, P3): Section 13 "UI copy (Polish)" added, including the fallback ESCALATE texts. Proposed by the documentation review; accepted for the MVP in R6.
- **2026-10-21 R6** (decisions taken by the documentation editor, closing the open questions in section 12): no analysis panel in the MVP (summary stays in the first message); photos under 1024 px rejected as unreadable (literal AC-11); Polish copy in section 13 accepted for the MVP; the photo is released after a successful analysis; provider data retention is a go-live blocker, so only synthetic data is used until it is settled.
