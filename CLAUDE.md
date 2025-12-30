# Agent Instructions

> This file is mirrored across CLAUDE.md, AGENTS.md, and GEMINI.md so the same instructions load in any AI environment.

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**
- Basically just SOPs written in Markdown, live in `directives/`
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**
- This is you. Your job: intelligent routing.
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `directives/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**
- Deterministic Python scripts in `execution/`
- Environment variables, api tokens, etc are stored in `.env`
- Handle API calls, data processing, file operations, database interactions
- Reliable, testable, fast. Use scripts instead of manual work.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Operating Principles

**1. Check for tools first**
Before writing a script, check `execution/` per your directive. Only create new scripts if none exist.

**2. Self-anneal when things break**
- Read error message and stack trace
- Fix the script and test it again (unless it uses paid tokens/credits/etc—in which case you check w user first)
- Update the directive with what you learned (API limits, timing, edge cases)
- Example: you hit an API rate limit → you then look into API → find a batch endpoint that would fix → rewrite script to accommodate → test → update directive.

**3. Update directives as you learn**
Directives are living documents. When you discover API constraints, better approaches, common errors, or timing expectations—update the directive. But don't create or overwrite directives without asking unless explicitly told to. Directives are your instruction set and must be preserved (and improved upon over time, not extemporaneously used and then discarded).

## Self-annealing loop

Errors are learning opportunities. When something breaks:
1. Fix it
2. Update the tool
3. Test tool, make sure it works
4. Update directive to include new flow
5. System is now stronger

---

## How to Write Directives

Directives are **SOPs (Standard Operating Procedures)** written in Markdown. They tell the agent what to do, which tools to use, and how to handle edge cases. Write them like instructions for a mid-level employee.

### Directive Structure

Use this template for new directives:

```markdown
# Directive Title

Brief 1-2 sentence description of what this does.

## When to Use
- Bullet points describing when this directive applies
- Specific use cases or triggers

## Inputs

| Parameter | Required | Description |
|-----------|----------|-------------|
| `--param` | Yes/No   | What it does |

## Execution

### Step 1: [Action Name]
```bash
python3 execution/script_name.py --arg value
```

Explain what this step does and any context needed.

### Step 2: [Next Action]
...continue with numbered steps...

## Output

- What the user receives (e.g., Google Sheet URL, document link)
- What format/columns/fields are included
- Where deliverables are stored

## Edge Cases
- **Scenario**: What can go wrong
  - → How to handle it

## Learnings
- (Updated as you discover things)
- Date: What was learned
```

### Directive Style Rules

1. **Use YAML frontmatter** for simple directives (optional):
   ```markdown
   ---
   description: Brief description of what this does
   ---
   ```

2. **Reference execution scripts explicitly**:
   ```markdown
   ## Tool
   **Script:** `execution/script_name.py`
   **Usage:** What it does in one line
   ```

3. **Show exact CLI commands**:
   ```bash
   python3 execution/gmaps_lead_pipeline.py --search "plumbers in Austin TX" --limit 50
   ```

4. **Use tables for parameters and outputs**:
   | Parameter | Default | Description |
   |-----------|---------|-------------|

5. **Document costs if applicable**:
   | Component | Cost per unit |
   |-----------|---------------|
   | Apify scrape | ~$0.01-0.02 |
   | Claude Haiku | ~$0.002 |

6. **Include Learnings section** — update with discoveries:
   ```markdown
   ## Learnings
   - Claude Haiku is sufficient for extraction and costs 10x less than Sonnet
   - ~10-15% of business websites return 403/503 errors - this is normal
   - 50 leads takes ~3-4 minutes with 3 workers
   ```

7. **Distinguish intermediates from deliverables**:
   - Intermediate: `.tmp/leads_timestamp.json` (never show to user)
   - Deliverable: Google Sheet URL (what user actually receives)

8. **Be explicit about multi-step workflows**: Use numbered steps with clear transitions like "Pass the output from Step 1 to Step 2"

9. **Include troubleshooting section** for common errors:
   ```markdown
   ## Troubleshooting

   ### "No leads found"
   - Check search query is valid
   - Include location in query

   ### 403 Forbidden errors
   - ~10-15% of sites block scrapers
   - Handled gracefully, lead still saved with partial data
   ```

---

## How to Write Execution Scripts

Execution scripts are **deterministic Python tools** that do the actual work. They should be reliable, testable, and standalone.

### Script Structure

Use this template for new execution scripts:

```python
#!/usr/bin/env python3
"""
script_name.py

Brief description of what this script does.

Usage:
    python3 execution/script_name.py --arg1 value --arg2 value

Inputs:
    --arg1: Description of argument
    --arg2: Description of argument

Outputs:
    - What the script produces (JSON file, Google Sheet, etc.)
"""

import os
import sys
import json
import argparse
from datetime import datetime
from dotenv import load_dotenv

# Load environment variables
load_dotenv()


def main_function(param1, param2):
    """
    Core logic for the script.
    
    Args:
        param1: Description
        param2: Description
        
    Returns:
        Result object or data
    """
    # Implementation here
    pass


def save_results(results, prefix="output"):
    """
    Save results to a JSON file in .tmp/
    """
    if not results:
        print("No results to save.")
        return None

    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    output_dir = ".tmp"
    os.makedirs(output_dir, exist_ok=True)

    filename = f"{output_dir}/{prefix}_{timestamp}.json"
    
    with open(filename, "w") as f:
        json.dump(results, f, indent=2)
        
    print(f"Results saved to {filename}")
    return filename


def main():
    parser = argparse.ArgumentParser(description="Script description")
    parser.add_argument("--arg1", required=True, help="What this arg does")
    parser.add_argument("--arg2", type=int, default=10, help="With default")
    parser.add_argument("--flag", action="store_true", help="Boolean flag")
    
    args = parser.parse_args()
    
    results = main_function(args.arg1, args.arg2)
    
    if results:
        print(f"Success: processed {len(results)} items")
        save_results(results)
    else:
        print("Error occurred.", file=sys.stderr)
        sys.exit(1)


if __name__ == "__main__":
    main()
```

### Script Style Rules

1. **Shebang and docstring** — Every script starts with:
   ```python
   #!/usr/bin/env python3
   """
   script_name.py
   
   Description, usage, inputs, outputs.
   """
   ```

2. **Load dotenv early**:
   ```python
   from dotenv import load_dotenv
   load_dotenv()
   ```

3. **Use argparse for CLI arguments**:
   ```python
   parser = argparse.ArgumentParser(description="...")
   parser.add_argument("--query", required=True, help="...")
   parser.add_argument("--limit", type=int, default=25)
   parser.add_argument("--no-email-filter", action="store_true")
   ```

4. **Check for required environment variables**:
   ```python
   api_token = os.getenv("APIFY_API_TOKEN")
   if not api_token:
       print("Error: APIFY_API_TOKEN not found in .env", file=sys.stderr)
       return None
   ```

5. **Save intermediates to `.tmp/`** with timestamps:
   ```python
   filename = f".tmp/{prefix}_{timestamp}.json"
   ```

6. **Print progress and debug info**:
   ```python
   print(f"Starting scrape for '{query}' in '{location}' (Limit: {max_items})...")
   print(f"Debug: run_input = {json.dumps(run_input, indent=2)}")
   ```

7. **Return JSON to stdout for chaining**:
   ```python
   print(json.dumps(response, indent=2))
   ```

8. **Use proper error handling**:
   ```python
   try:
       result = api_call()
   except requests.exceptions.HTTPError as e:
       if e.response.status_code == 429:
           retry_after = int(e.response.headers.get('Retry-After', 60))
           time.sleep(retry_after)
   ```

9. **Implement retry logic with exponential backoff**:
   ```python
   max_retries = 3
   for attempt in range(max_retries):
       try:
           response = requests.post(url, json=payload, timeout=30)
           response.raise_for_status()
           return response.json()
       except Exception as e:
           if attempt == max_retries - 1:
               raise
           time.sleep(2 ** attempt)  # 1s, 2s, 4s
   ```

10. **Support both stdin and file input** for JSON processing:
    ```python
    if len(sys.argv) > 1:
        with open(sys.argv[1], 'r') as f:
            data = json.load(f)
    else:
        data = json.loads(sys.stdin.read())
    ```

11. **Use dataclasses for structured config**:
    ```python
    from dataclasses import dataclass, field
    
    @dataclass
    class Config:
        client_email: str
        project_title: str
        tokens: List[Dict[str, str]] = field(default_factory=list)
    ```

12. **Handle Google Sheets auth properly**:
    ```python
    # Check token.json first (OAuth), then fall back to service account
    if os.path.exists('token.json'):
        creds = Credentials.from_authorized_user_file('token.json', SCOPES)
    elif os.path.exists('credentials.json'):
        flow = InstalledAppFlow.from_client_secrets_file('credentials.json', SCOPES)
        creds = flow.run_local_server(port=0)
    ```

13. **Use batch operations for large datasets**:
    ```python
    if len(all_data) > 1000:
        chunk_size = 1000
        for i in range(0, len(all_data), chunk_size):
            chunk = all_data[i:i + chunk_size]
            worksheet.update(values=chunk, range_name=f"A{i+1}:...")
    ```

14. **Exit with proper codes**:
    ```python
    if __name__ == "__main__":
        sys.exit(main())  # main() returns 0 on success, 1 on failure
    ```

### Common Patterns

**Google Sheets integration:**
- `read_sheet.py` — Read data from sheet to JSON
- `update_sheet.py` — Upload JSON to sheet (creates if needed)
- `append_to_sheet.py` — Add rows to existing sheet

**API scraping:**
- Check for API token in env
- Use Apify client for managed scraping
- Handle rate limits with retry logic
- Save raw results to `.tmp/`

**LLM processing:**
- Use Claude Haiku for extraction (cheaper)
- Use Opus 4.5 for complex reasoning
- Always set timeout and max tokens

---

## File Organization

**Deliverables vs Intermediates:**
- **Deliverables**: Google Sheets, Google Slides, or other cloud-based outputs that the user can access
- **Intermediates**: Temporary files needed during processing

**Directory structure:**
- `.tmp/` - All intermediate files (dossiers, scraped data, temp exports). Never commit, always regenerated.
- `execution/` - Python scripts (the deterministic tools)
- `directives/` - SOPs in Markdown (the instruction set)
- `.env` - Environment variables and API keys
- `credentials.json`, `token.json` - Google OAuth credentials (required files, in `.gitignore`)

**Key principle:** Local files are only for processing. Deliverables live in cloud services (Google Sheets, Slides, etc.) where the user can access them. Everything in `.tmp/` can be deleted and regenerated.

## Cloud Webhooks (Modal)

The system supports event-driven execution via Modal webhooks. Each webhook maps to exactly one directive with scoped tool access.

**When user says "add a webhook that...":**
1. Read `directives/add_webhook.md` for complete instructions
2. Create the directive file in `directives/`
3. Add entry to `execution/webhooks.json`
4. Deploy: `modal deploy execution/modal_webhook.py`
5. Test the endpoint

**Key files:**
- `execution/webhooks.json` - Webhook slug → directive mapping
- `execution/modal_webhook.py` - Modal app (do not modify unless necessary)
- `directives/add_webhook.md` - Complete setup guide

**Endpoints:**
- `https://nick-90891--claude-orchestrator-list-webhooks.modal.run` - List webhooks
- `https://nick-90891--claude-orchestrator-directive.modal.run?slug={slug}` - Execute directive
- `https://nick-90891--claude-orchestrator-test-email.modal.run` - Test email

**Available tools for webhooks:** `send_email`, `read_sheet`, `update_sheet`

**All webhook activity streams to Slack in real-time.**

## Summary

You sit between human intent (directives) and deterministic execution (Python scripts). Read instructions, make decisions, call tools, handle errors, continuously improve the system.

Be pragmatic. Be reliable. Self-anneal.

Also, use Opus-4.5 for everything while building. It came out a few days ago and is an order of magnitude better than Sonnet and other models. If you can't find it, look it up first.