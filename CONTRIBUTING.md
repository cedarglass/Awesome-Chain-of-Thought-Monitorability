# Contributing

Contributions should improve the accuracy, coverage, or organization of the survey.

## Adding a paper

Open an issue or pull request with the paper placed in the most relevant section. Use this format:

```markdown
- **[MM/DD/YYYY] Paper title** [arXiv](url) · [alphaXiv](url) · [Code](url)  
  **Authors:** Author One, Author Two  
  **Scope:** Core, adjacent, or commentary (omit only when obvious from the section).
  **TL;DR:** One or two factual sentences describing the paper's contribution to CoT monitorability.
```

Prefer an arXiv, OpenReview, proceedings, or publisher URL over a secondary summary. Keep descriptions neutral and avoid claims not supported by the paper.

Include, when known:

- authors and venue;
- initial publication or arXiv submission date;
- code, data, and project links;
- tags from the working taxonomy;
- a short reason the paper is in scope.

## Inclusion criteria and scope

- **Core:** CoT monitorability, CoT faithfulness, reasoning-trace oversight, or a direct CoT-monitoring failure is a central research object.
- **Adjacent:** The work studies a mechanism that changes the production, interpretation, robustness, or replacement of visible reasoning, such as evaluation awareness, role conditioning, latent reasoning, activation monitoring, or monitor-side failure.
- **Commentary:** A non-peer-reviewed work supplies a useful hypothesis or vocabulary and is clearly labeled as such.

Broad reasoning, personality, prompt-injection, or AI-control papers should not be added solely because they use chain of thought or monitoring. State the concrete claim about CoT evidence that the work changes.

## Updating the literature map

For a core paper, add a row to the property map and identify direct measurements separately from related implications. If the paper introduces a reusable dataset, benchmark, environment, or protocol, also add it to the asset table with its target, observation channel, property, pressure condition, and released artifact.

## Quality checks

- Verify the title, year, links, and summary against the primary source.
- Add a paper only once; cross-link it from another section if needed.
- Keep monitoring targets, failure mechanisms, trace properties, and monitor implementations conceptually separate.
- Do not rank papers or use promotional language.
- Explain taxonomy changes in the pull request description.
