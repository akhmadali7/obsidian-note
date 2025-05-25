
Protected Route - User Personalized Data
(alternatives related)
1. add alternatives -> alterantives created with pending status, not included at calculation yet because it hasnt valid score value
2. read alternatives
3. update alternatives
4. delete alternatives
5. archive / unarchived alternatives -> alternative archived and not included in calculation vice versa

(score related)
1. read all scores
2. read one score by alternative -> if alternative archived it wont read
3. update score by alternative -> alternatives updated with valid score value + status change to accepted, included in calcultion

```mermaid
flowchart TD
    A[User must be logged in] --> B{Actions}
    B --> C[Add alternatives]
    B --> D[Read alternatives]
    B --> E[Update alternatives]
    B --> F[Delete alternatives]
    B --> G[Archive / Unarchive alternatives]
    B --> H[Read all scores]
    B --> I[Read one score by alternative]
    B --> J[Update score by alternative]

    C --> K[Alternative created with pending status]
    K --> L[Not included in calculation yet - pending, no valid score]

    G --> M[Alternative archived]
    M --> N[Not included in calculation]
    G --> O[Alternative unarchived]
    O --> P[Included in calculation]

    I --> Q[Alternative archived?]
    Q -->|Yes| R[Cannot read score]
    Q -->|No| S[Read score by alternative]

    J --> T[Valid score value]
    T --> U[Alternative status changed to accepted]
    U --> V[Included in calculation]

    classDef actions fill:#f9f,stroke:#333,stroke-width:2px;
    class B,C,D,E,F,G,H,I,J actions;

```