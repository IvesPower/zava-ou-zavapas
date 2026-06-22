<!--
Review Panel - recommendation comment template (advisory regime).

RENDERING RULES (the orchestrator follows these literally):
- The panel is ADVISORY. NEVER render "Verdict", "APPROVE", "REJECT",
  "blocked", "merge gate", or any equivalent.
- Sections are SKIPPED (not rendered as empty placeholders) when their
  source field is empty or missing.
- The per-persona table renders ALL three panelists.
- Follow-ups are CAPPED at 5 items.
-->

## Review Panel: {{ synthesizer.stance }}

> {{ synthesizer.headline }}

{{ synthesizer.arbitration }}

### Panel summary

| Persona | B | R | N | Takeaway |
|---|---|---|---|---|
{{#each panelists }}
| {{ persona | humanize }} | {{ count_blocking }} | {{ count_recommended }} | {{ count_nits }} | {{ summary }} |
{{/each}}

> B = blocking-severity findings, R = recommended, N = nits.
> Counts are signal strength, not gates. The maintainer ships.

{{#if synthesizer.follow_ups.length }}
### Follow-ups

{{#each synthesizer.follow_ups[:5] }}
{{ @index_plus_1 }}. **[{{ from_persona | humanize }}]** {{ summary }}
{{/each}}
{{/if}}

### Recommendation

{{ synthesizer.recommendation_prose }}

---

<details>
<summary>Full per-persona findings</summary>

{{#each panelists }}
#### {{ persona | humanize }}

{{#if findings.length }}
{{#each findings }}
- **[{{ severity }}]** {{ title }}{{#if file }} at `{{ file }}{{#if line }}:{{ line }}{{/if}}`{{/if}}
  {{ detail }}
{{/each}}
{{else}}
No findings.
{{/if}}

{{/each}}
</details>

<sub>This panel is advisory. It does not block merge. Re-apply the
`panel-review` label after addressing feedback to re-run.</sub>
