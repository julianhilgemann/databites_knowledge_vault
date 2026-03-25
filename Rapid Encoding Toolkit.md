
### 1. The "Boardroom Defense" (Evolved Feynman Technique)

*   **The Adjustment:** Don't just teach a junior. Imagine you are defending your architecture to a skepticial **CTO** or **Head of Data**.
*   **Why:** You aren't applying for a Junior role anymore. You need to practice the language of *trade-offs* and *decisions*, not just definitions.
*   **The Method:**
    1.  Stand up.
    2.  **The Prompt:** "Why did you choose a [[Star Schema]] over one big table? Why did you use `SUMMARIZE` here instead of `GROUP BY` in [[SQL]]?"
    3.  **The Drill:** Explain the **Architecture**.
        *   *Bad:* "[[Star Schema|Star schema]] is better because it has facts and dims."
        *   *Good:* "I chose a [[Star Schema]] to optimize for the **[[Vertipaq|VertiPaq]] engine**, which compresses narrow columns better than wide [[Tables|tables]]. It also allows us to use standard **[[Time Intelligence]]** functions without complex [[DAX]] workarounds."

### 2. The "Notepad" Purge (Anti-Copilot Training)

*   **The Adjustment:** This is your "Detox." You must prove to yourself that you can write the "Master List" functions without AI.
*   **Why:** To cure your Imposter Syndrome. Once you realize you *can* write `CALCULATE` and `RANKX` blindly, you will stop fearing the whiteboard.
*   **The Method:**
    1.  Open **Notepad** (No Copilot, No Intellisense).
    2.  **Task 1 ([[DAX]]):** Write the "High Value Customer" measure using `VAR`, `CALCULATE`, and `FILTER`.
    3.  **Task 2 ([[SQL]]):** Write a CTE that creates a `ROW_NUMBER` to deduplicate data.
    4.  **The Rule:** If you get stuck, **do not prompt AI**. Look at the documentation documentation, fix it manually, delete it, and type it again.

### 3. The "Pseudo-Code" Pivot (Your Safety Net)

*   **The Adjustment:** This is a new tool specifically for practical application. It trains you to think in "Prompt Logic" but deliver it as "Engineering Logic."
*   **Why:** If you blank on syntax during a live coding test, you fall back to this. It shows you know *what* to do, even if you forgot the comma.
*   **The Method:**
    1.  Take a complex problem (e.g., "Rolling 12 Month Average").
    2.  **Step 1:** Write the logic in comments (Pseudo-code) first.
        ```dax
        // 1. Get the last date visible
        // 2. Create a range of dates: Current Date minus 12 months
        // 3. Calculate Sales over that range
        ```
    3.  **Step 2:** Fill in the code below it.
    4.  **The Benefit:** in a professional setting, if you write the comments first, your stakeholders see your process. If you mess up the syntax later, they usually forgive you because the logic was sound.

### 4. The "Full Stack" Canvas (Visualizing the Pipeline)

*   **The Adjustment:** Leverage your Figma/Design skills. Don't just draw the [[Tables|tables]]; draw the **Pipeline**.
*   **Why:** Your competitive advantage is that you do *everything* (Requirements $\to$ [[dbt]] $\to$ PBI $\to$ UI). Visualizing this cements your identity as a "Specialist" rather than just a "Report Builder."
*   **The Method:**
    1.  On a whiteboard or tablet (Obsidian Canvas/Excalidraw).
    2.  Draw the flow: **Raw Data $\to$ Lakehouse $\to$ [[dbt]] (Star Schema) $\to$ [[TMDL]] (Semantic Model) $\to$ PBIP (Report) $\to$ App**.
    3.  Annotate *where* the logic lives. (e.g., "I do SCD Type 2 logic *here* in [[dbt]], so the [[DAX]] *here* is simple").

### 5. The "Hybrid Sprint" ([[Simulation]])

*   **The Adjustment:** Simulate a real work session, but strictly time-box the AI usage to "Reviewer Mode."
*   **Why:** Proves that AI makes you faster, but you are the pilot.
*   **The Method:**
    1.  Set timer for 30 mins.
    2.  **Prompt:** "Build a YoY calculation that handles [[[[Context]] Transition|[[[[Context]] Transition|[[[[Context]] Transition|context transition]]]]]] correctly."
    3.  **Phase 1 (20 mins):** Write it manually in Notepad. Struggle with it.
    4.  **Phase 2 (5 mins):** Paste it into ChatGPT. Prompt: *"Act as a Senior Principal Engineer. Code Review this DAX. Do not rewrite it, just point out potential performance flaws or context risks."*
    5.  **Phase 3 (5 mins):** Fix your own code based on the feedback.
    6.  *This mimics the explanation you will give to stakeholders: "I write the logic, then use AI to audit for edge cases."*

### 6. Anki "Senior Vocab" Deck

*   **The Adjustment:** Focus purely on the high-level [[Vocabulary|vocabulary]] that justifies your 2 years of experience.
*   **The Cards:**
    *   *Front:* "Why use [[TMDL]]?" $\to$ *Back:* "To enable [[git|Git]] merge conflict resolution and code-first bulk editing."
    *   *Front:* "Difference between DirectQuery and Direct Lake?" $\to$ *Back:* "DirectQuery translates DAX to [[SQL]] (Slow). Direct Lake reads Parquet columns into RAM (Fast)."
    *   *Front:* "What is Context Transition?" $\to$ *Back:* "When `CALCULATE` forces a Row Context to become a Filter Context."