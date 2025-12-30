# Execution Scripts

> Layer 3: Doing the work

This folder contains **deterministic Python scripts** that handle the actual work:

- API calls
- Data processing
- File operations
- Database interactions
- External service integrations

## Key Principles

1. **Deterministic & Reliable**: Same inputs → same outputs, every time
2. **Testable**: Each script should be runnable standalone for testing
3. **Environment-aware**: Use `.env` for API keys and secrets
4. **Error-handling**: Scripts should fail gracefully with clear error messages

## Script Structure

Each script should follow this pattern:

```python
#!/usr/bin/env python3
"""
script_name.py

Description of what this script does.

Usage:
    python script_name.py [args]
    
Inputs:
    - arg1: description
    - arg2: description
    
Outputs:
    - What the script produces
"""

import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

def main():
    # Script logic here
    pass

if __name__ == "__main__":
    main()
```

## Available Scripts

| Script | Purpose |
|--------|---------|
| (none yet) | Add scripts as they're created |

## Adding New Scripts

1. Check if a similar script already exists
2. Follow the structure template above
3. Test the script standalone
4. Document it in the corresponding directive
5. Update this README
