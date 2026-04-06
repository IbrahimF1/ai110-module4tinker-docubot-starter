# DocuBot Model Card

This model card is a short reflection on your DocuBot system. Fill it out after you have implemented retrieval and experimented with all three modes:

1. Naive LLM over full docs  
2. Retrieval only  
3. RAG (retrieval plus LLM)

Use clear, honest descriptions. It is fine if your system is imperfect.

---

## 1. System Overview

**What is DocuBot trying to do?**  
Describe the overall goal in 2 to 3 sentences.

> DocuBot answers developer questions about a codebase using project documentation. It retrieves relevant text sections and generates responses through three modes: naive LLM, retrieval only, and RAG.

**What inputs does DocuBot take?**  
For example: user question, docs in folder, environment variables.

> DocuBot takes a user question and loads documentation files from the docs/ folder. It optionally uses a GEMINI_API_KEY environment variable for LLM features.

**What outputs does DocuBot produce?**

> DocuBot returns retrieved text snippets with source filenames or generated answers. In retrieval only mode, it returns raw snippets. In RAG mode, it returns LLM-generated answers based on retrieved snippets.

---

## 2. Retrieval Design

**How does your retrieval system work?**  
Describe your choices for indexing and scoring.

- How do you turn documents into an index?
- How do you score relevance for a query?
- How do you choose top snippets?

> I build an inverted index that maps each lowercase word to the documents containing it. I split documents into paragraphs by newlines and score each section by counting matching query words. I return the top-k sections sorted by score in descending order.

**What tradeoffs did you make?**  
For example: speed vs precision, simplicity vs accuracy.

> I chose simplicity over accuracy by using basic word matching instead of semantic understanding. This approach runs fast but may miss relevant content that uses different terminology.

---

## 3. Use of the LLM (Gemini)

**When does DocuBot call the LLM and when does it not?**  
Briefly describe how each mode behaves.

- Naive LLM mode:
- Retrieval only mode:
- RAG mode:

> Naive LLM mode calls the LLM without any context. Retrieval only mode never calls the LLM and returns raw snippets. RAG mode calls the LLM with retrieved snippets as context.

**What instructions do you give the LLM to keep it grounded?**  
Summarize the rules from your prompt. For example: only use snippets, say "I do not know" when needed, cite files.

> I instruct the LLM to use only the provided snippets and refuse to guess when evidence is insufficient. I require the LLM to reply "I do not know based on the docs I have" when snippets lack information. I ask the LLM to mention which files it relied on when answering.

---

## 4. Experiments and Comparisons

Run the **same set of queries** in all three modes. Fill in the table with short notes.

You can reuse or adapt the queries from `dataset.py`.

| Query | Naive LLM: helpful or harmful? | Retrieval only: helpful or harmful? | RAG: helpful or harmful? | Notes |
|------|---------------------------------|--------------------------------------|---------------------------|-------|
| Example: Where is the auth token generated? | Harmful | Helpful | Helpful | Naive LLM hallucinates, retrieval finds correct line |
| Example: How do I connect to the database? | Harmful | Helpful | Helpful | Naive LLM invents steps, retrieval finds actual instructions |
| Example: Which endpoint lists all users? | Harmful | Helpful | Helpful | Naive LLM guesses incorrectly, retrieval finds correct endpoint |
| Example: How does a client refresh an access token? | Harmful | Helpful | Helpful | Naive LLM provides wrong method, retrieval finds correct endpoint |

**What patterns did you notice?**  

- When does naive LLM look impressive but untrustworthy?  
- When is retrieval only clearly better?  
- When is RAG clearly better than both?

> Naive LLM looks impressive when it provides detailed, confident answers but often hallucinates information not in the docs. Retrieval only is clearly better for factual questions where the answer exists in the documentation. RAG is clearly better when the question requires synthesis or explanation beyond simple fact retrieval.

---

## 5. Failure Cases and Guardrails

**Describe at least two concrete failure cases you observed.**  
For each one, say:

- What was the question?  
- What did the system do?  
- What should have happened instead?

> **Failure case 1:** Question: "What payment methods does the system support?" The system returned no results because the docs don't mention payment processing. It correctly refused to answer with "I do not know based on these docs."

> **Failure case 2:** Question: "How do I delete a user account?" The system retrieved snippets about user table fields but no deletion instructions. It returned irrelevant snippets instead of refusing to answer.

**When should DocuBot say "I do not know based on the docs I have"?**  
Give at least two specific situations.

> DocuBot should say "I do not know" when no retrieved snippets contain relevant information. It should also refuse when snippets mention related concepts but don't answer the specific question.

**What guardrails did you implement?**  
Examples: refusal rules, thresholds, limits on snippets, safe defaults.

> I implemented a guardrail that checks if retrieval returns any results before answering. I return "I do not know based on these docs" when no relevant snippets exist. I also skip empty sections during retrieval to avoid returning blank content.

---

## 6. Limitations and Future Improvements

**Current limitations**  
List at least three limitations of your DocuBot system.

1. Simple word matching fails to understand semantic meaning or context.
2. The system cannot handle multi-word phrases or technical terms that differ from exact matches.
3. Paragraph-level retrieval may miss relevant information spread across multiple sections.

**Future improvements**  
List two or three changes that would most improve reliability or usefulness.

1. Implement semantic search using embeddings to match meaning beyond exact words.
2. Add phrase matching and technical term recognition to improve precision.
3. Implement context windowing to combine related paragraphs for more comprehensive retrieval.

---

## 7. Responsible Use

**Where could this system cause real world harm if used carelessly?**  
Think about wrong answers, missing information, or over trusting the LLM.

> The system could cause harm if developers implement incorrect authentication methods based on hallucinated LLM responses. It could also lead to security vulnerabilities if it provides outdated or incomplete configuration instructions. Over-reliance on the system might prevent developers from reading actual documentation.

**What instructions would you give real developers who want to use DocuBot safely?**  
Write 2 to 4 short bullet points.

- Always verify critical information by reading the actual documentation files.
- Treat LLM-generated answers as suggestions rather than authoritative sources.
- Use retrieval only mode for factual questions where exact accuracy matters.
- Cross-check any configuration values, endpoints, or code examples in the original docs.
