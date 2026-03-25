
## 1. Data Modeling (The Bible)

**Book:** ***The Data Warehouse Toolkit***
**Author:** Ralph [[Kimball]] & Margy Ross
**Why:** This is the source of truth. If a hiring manager mentions "[[Star Schema]]," they are referencing this book. It defines the [[Vocabulary|vocabulary]] of the industry.

*   **⚡ Focus Chapters:**
    *   **Chapter 1:** *Data Warehousing, Business Intelligence, and Dimensional Modeling Primer* (Understand the "Why").
    *   **Chapter 2:** *[[Kimball]] Dimensional Modeling Techniques Overview* (The Definitions: Grain, Fact, Dimension, SCD, Bus Matrix).
    *   **Chapter 6:** *Order Management* (The classic "Sales" example you will likely use on a whiteboard).

## 2. Power BI & [[DAX]] (The Engine)

**Book:** ***The Definitive Guide to [[DAX]]***
**Author:** Marco Russo & Alberto Ferrari (The "SQLBI" Guys)
**Why:** They wrote the documentation that Microsoft uses. Owning this book signals you are serious about the *engine*, not just the tool.

*   **⚡ Focus Chapters:**
    *   **Chapter 4:** *Understanding Evaluation Contexts* (Row vs. Filter [[Context]]—Crucial).
    *   **Chapter 5:** *Understanding CALCULATE and CALCULATETABLE* ([[Context Transition]]).
    *   **Chapter 15:** *Advanced [[DAX]] Concepts* (Specifically the section on *Expanded [[Tables]]*—the advanced "Specialist" concept).

## 3. Data Visualization (The Design)

**Book:** ***Storytelling with Data***
**Author:** Cole Nussbaumer Knaflic
**Why:** It moves you from "making charts" to "communicating insights." It creates the "Clean UI" [[Vocabulary|vocabulary]] (clutter, preattentive attributes) that impresses stakeholders.

*   **⚡ Focus Chapters:**
    *   **Chapter 3:** *Clutter is your enemy* (Data-Ink Ratio, removing borders/gridlines).
    *   **Chapter 4:** *Focus your audience’s attention* (Preattentive attributes: Size, Color, Position).

## 4. [[SQL]] (The Foundation)

**Book:** ***T-[[SQL]] Fundamentals***
**Author:** Itzik Ben-Gan
**Why:** Ben-Gan teaches [[SQL]] as **Set Theory**, not just code. This "Set-based thinking" is the perfect mental bridge to understanding DAX.

*   **⚡ Focus Chapters:**
    *   **Chapter 1:** *Background to T-SQL Querying* (Specifically: **Logical Query Processing Order**—knowing that `FROM` runs before `SELECT`).
    *   **Chapter 8:** *Window Functions* (For `RANK`, `LAG`, `LEAD` upstream calculations).

## 5. Modern Architecture ([[dbt]] & Engineering)

**Book:** ***Fundamentals of Data Engineering***
**Author:** Joe Reis & Matt Housley
**Why:** It explains the "Modern Data Stack" (ELT vs ETL). Even if you don't code Python, understanding the lifecycle (Generation → Ingestion → Transformation → Serving) proves you are an Architect.

*   **⚡ Focus Chapters:**
    *   **Chapter 7:** *Transformation* (The role of [[dbt]], modeling in the warehouse, and data quality).

---

### 💡 The "One-Liner" Drop
If you want to sound like an insider in practice, drop this line:

> *"I treat **[[Kimball]]'s** Toolkit as the blueprint for the architecture, and **Russo & Ferrari's** Definitive Guide as the manual for the engine. I don't guess with the model; I follow the standard."*