# Research AW UML

`oppo-research` の Research AW を、Issue を起点に複数の Agent が並列実行し、RAG と Research Brief に統合する構造として定義する。

## System

```mermaid
flowchart TD
    I[Research Issue #12] --> AW[Research AW Dispatcher]
    AW --> BA[Book Agent]
    AW --> SA[Source Agent]
    AW --> OA[Ontology Agent]
    BA --> RA[RAG Agent]
    SA --> RA
    OA --> RA
    RA --> B[Research Brief Agent]
    B --> N[oppo-novel]
    B --> P[oppo-poc]
```

## Agent Sequence

```mermaid
sequenceDiagram
    participant I as Issue
    participant AW as AW Dispatcher
    participant B as Book Agent
    participant S as Source Agent
    participant O as Ontology Agent
    participant R as RAG Agent
    participant BR as Brief Agent

    I->>AW: Research Mission
    par Spawn Agents
        AW->>B: books.md inventory
        AW->>S: source/evidence inventory
        AW->>O: ontology mapping
    end
    B-->>R: source inventory
    S-->>R: evidence / claims
    O-->>R: entities / relations / dimensions
    R->>R: normalize + dedupe + provenance
    R->>BR: RAG dataset
    BR-->>I: Research Brief / status
```

## Domain Model

```mermaid
classDiagram
    class ResearchIssue {
      +number
      +mission
      +research_questions
      +status
    }

    class Source {
      +id
      +title
      +author
      +year
      +publisher
    }

    class Evidence {
      +id
      +level A/B/C/D
      +page
      +excerpt
      +status
    }

    class Claim {
      +id
      +text
      +type
      +status
    }

    class Entity {
      +id
      +type
      +name
    }

    class Relation {
      +subject
      +predicate
      +object
    }

    class RAGRecord {
      +id
      +source_id
      +type
      +evidence_level
      +dimensions
      +p2_relevance
    }

    class ResearchBrief {
      +timeline
      +media_matrix
      +human_needs
      +insights
      +open_questions
    }

    ResearchIssue --> Source : investigates
    Source --> Evidence : supports
    Evidence --> Claim : supports
    Claim --> Entity : describes
    Entity --> Relation : participates
    Claim --> RAGRecord : normalized as
    Evidence --> RAGRecord : provenance
    RAGRecord --> ResearchBrief : feeds
```

## GitHub Actions Deployment

```mermaid
flowchart LR
    D[workflow_dispatch] --> I[Issue #12]
    I --> J1[Book Agent]
    I --> J2[Source Agent]
    I --> J3[Ontology Agent]
    J1 --> J4[RAG Agent]
    J2 --> J4
    J3 --> J4
    J4 --> J5[Research Brief Agent]
```

Workflow:

`\.github/workflows/research-aw.yml`

The current implementation is intentionally lightweight. Agents establish the execution graph and workspace first; claim extraction, evidence verification, RAG generation, and Research Brief production are subsequent AW implementation steps.

## Architecture Rules

1. **Issue is the mission.**
2. **Agent is an execution role.**
3. **books.md is the source catalog.**
4. **Evidence keeps provenance.**
5. **Historical fact and hypothesis remain separate.**
6. **RAG is machine-readable research memory.**
7. **Research Brief is the handoff artifact.**
8. **Novel and PoC consume research; they do not own historical evidence.**

## Evidence Levels

| Level | Meaning |
|---|---|
| A | Primary / official / contemporaneous |
| B | Contemporaneous secondary source |
| C | Retrospective / first-person account |
| D | Hearsay / unverified |
