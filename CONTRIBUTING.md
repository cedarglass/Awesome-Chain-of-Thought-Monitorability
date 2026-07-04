# Contributing

Contributions should improve the accuracy, coverage, or organization of the survey.

## Adding a paper

Open an issue or pull request with the paper placed in the most relevant section. Use this format:

```markdown
- **[MM/DD/YYYY] Paper title** [arXiv](url) · [alphaXiv](url) · [Code](url)  
  **Authors:** Author One, Author Two  
  **TL;DR:** One or two factual sentences describing the paper's contribution to CoT monitorability.
```

Prefer an arXiv, OpenReview, proceedings, or publisher URL over a secondary summary. Keep descriptions neutral and avoid claims not supported by the paper.

Include, when known:

- authors and venue;
- initial publication or arXiv submission date;
- code, data, and project links;
- tags from the working taxonomy;
- a short reason the paper is in scope.

## Inclusion criteria

A paper belongs in the main list when CoT monitorability, CoT faithfulness, reasoning-trace oversight, or a direct failure mode is a central research object. Broad reasoning papers without a monitoring connection should not be added solely because they use chain of thought.

## Quality checks

- Verify the title, year, links, and summary against the primary source.
- Add a paper only once; cross-link it from another section if needed.
- Do not rank papers or use promotional language.
- Explain taxonomy changes in the pull request description.
