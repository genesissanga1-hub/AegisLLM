# Genesis Tencen Sanga (GTS) - Quick Reference Guide

## The 5 Pillars of GTS

### 1. [31mShadow DOM Closed Mode[0m
**REQUIRED:** All UI components MUST use Shadow DOM with `mode: 'closed'`

```javascript
// CORRECT
const shadow = element.attachShadow({ mode: 'closed' });

// VIOLATION - Open mode
const shadow = element.attachShadow({ mode: 'open' });  // ❌
```

**Why:** Prevents external DOM access, CSS interference, and XSS attacks.

---

### 2. [31mNo Autocomplete/Autofill[0m
**REQUIRED:** Disable autocomplete on ALL forms and inputs

```html
<!-- CORRECT -->
<form autocomplete="off" autocorrect="off" spellcheck="false">
    <input type="text" autocomplete="off" autocorrect="off" 
           autocapitalize="off" spellcheck="false">
</form>

<!-- VIOLATION -->
<form autocomplete="on">  <!-- ❌ -->
```

**Why:** Prevents browser data leakage, form injection, and unpredictable inputs.

---

### 3. [31mStop Event Propagation[0m
**REQUIRED:** All event listeners MUST stop propagation

```javascript
// CORRECT
el.addEventListener('click', (e) => {
    e.stopImmediatePropagation();
    e.stopPropagation();
    e.preventDefault();
}, true);  // Capture phase

// VIOLATION - Allows bubbling
el.addEventListener('click', (e) => {  // ❌
    // Event continues to bubble
});
```

**Why:** Prevents event hijacking, maintains component isolation, stops information leakage.

---

### 4. [31mStrict Mode Everywhere[0m
**REQUIRED:** All scripts MUST execute in strict mode

```javascript
// CORRECT - Module level
'use strict';

// CORRECT - Function level
function myFunc() {
    'use strict';
    // code
}

// VIOLATION - No strict mode
function myFunc() {  // ❌
    // Not in strict mode
}
```

**Why:** Prevents silent errors, secure scoping, no `this` coercion, disallowed dangerous syntax.

---

### 5. [31mPipeline Isolation[0m
**REQUIRED:** All pipelines MUST operate in isolated states

```python
# CORRECT - Isolated pipeline
class IsolatedPipeline:
    async def process(self, input):
        # Fresh state for each request
        context = IsolationContext()
        return await self._execute(input, context)

# VIOLATION - Shared state
class SharedPipeline:  # ❌
    def __init__(self):
        self.shared_state = {}  # Shared across requests!
```

**Why:** Prevents cross-request contamination, state leakage, and race conditions.

---

## GTS Compliance Checklist

### [32m✓ Web Components[0m
- [ ] All custom elements use `attachShadow({ mode: 'closed' })`
- [ ] No open Shadow DOM anywhere
- [ ] Shadow DOM not accessible via `element.shadowRoot`
- [ ] Styles are encapsulated (no external CSS leakage)

### [32m✓ Forms & Inputs[0m
- [ ] All `<form>` elements have `autocomplete="off"`
- [ ] All `<input>`, `<textarea>`, `<select>` have `autocomplete="off"`
- [ ] All form controls have `autocorrect="off"`
- [ ] All form controls have `spellcheck="false"`
- [ ] No browser autofill suggestions appear

### [32m✓ Event Listeners[0m
- [ ] All event listeners call `stopImmediatePropagation()`
- [ ] All event listeners call `stopPropagation()`
- [ ] All event listeners call `preventDefault()` where applicable
- [ ] Listeners use capture phase (`true` as third argument)
- [ ] No event bubbling allowed

### [32m✓ Scripts[0m
- [ ] All scripts have `'use strict'` at the top
- [ ] All functions that need strict mode have it
- [ ] No implicit globals (would throw in strict mode)
- [ ] No `this` coercion issues
- [ ] No assignment to undeclared variables

### [32m✓ Pipelines (Python)[0m
- [ ] Each pipeline execution has unique isolation ID
- [ ] No shared mutable state between requests
- [ ] All input data is deep copied
- [ ] All output data is deep copied
- [ ] No references to original data maintained

### [32m✓ GenAI Processors[0m
- [ ] All processors deep copy input content
- [ ] All processors create new part instances
- [ ] No mutation of input parts
- [ ] Isolation metadata added to all parts
- [ ] Each processor execution independent

---

## GTS Code Templates

### Secure Web Component

```javascript
class SecureComponent extends HTMLElement {
    constructor() {
        super();
        this._shadow = this.attachShadow({ mode: 'closed' });  // Pillar 1
    }
    
    connectedCallback() {
        'use strict';  // Pillar 4
        
        this._shadow.innerHTML = `
            <form autocomplete="off" autocorrect="off">  // Pillar 2
                <input autocomplete="off" autocorrect="off">
                <button>Submit</button>
            </form>
        `;
        
        const form = this._shadow.querySelector('form');
        form.addEventListener('submit', (e) => {  // Pillar 3
            e.stopImmediatePropagation();
            e.stopPropagation();
            e.preventDefault();
            this._handleSubmit();
        }, true);
    }
    
    _handleSubmit() {
        'use strict';
        // Handle in isolation
    }
}
```

### Isolated Python Pipeline

```python
import copy
import uuid
from typing import Any, Dict

class IsolatedPipeline:
    def __init__(self):
        self._lock = asyncio.Lock()
        self._counter = 0
    
    async def process(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        async with self._lock:
            self._counter += 1
            isolation_id = f"iso-{self._counter}-{uuid.uuid4()}"
        
        # Deep copy input (Pillar 5)
        isolated_input = copy.deepcopy(input_data)
        
        # Process in isolation
        result = {}
        for key, value in isolated_input.items():
            result[key] = self._process_value(value, isolation_id)
        
        # Add isolation metadata
        result['_gts'] = {
            'isolation_id': isolation_id,
            'compliant': True
        }
        
        # Deep copy output
        return copy.deepcopy(result)
```

### GTS-Compliant GenAI Processor

```python
from genai_processors import processor
from genai_processors import content_api
import copy
import uuid

class GTSProcessor(processor.Processor):
    async def call(
        self, 
        content: content_api.ProcessorStream
    ):
        # Create isolation ID
        isolation_id = str(uuid.uuid4())
        
        # Deep copy input
        gathered = await content.gather()
        isolated = copy.deepcopy(gathered)
        
        # Process each part
        for part in isolated:
            yield content_api.ProcessorPart(
                text=part.text,
                role=part.role,
                metadata={
                    **part.metadata,
                    '_gts_isolation_id': isolation_id,
                    '_gts_compliant': True
                }
            )
```

---

## GTS Verification Commands

### Check for Violations

```bash
# Find open Shadow DOM (VIOLATION)
grep -r "attachShadow.*mode.*['\"]open['\"]" src/

# Find autocomplete enabled (VIOLATION)
grep -r "autocomplete.*['\"]on['\"]" src/

# Find missing strict mode (WARNING)
# Requires more sophisticated analysis
```

### Runtime Verification

```javascript
// Verify Shadow DOM is closed
function verifyShadowDOMClosed(element) {
    const shadow = element.shadowRoot;
    if (shadow && shadow.mode !== 'closed') {
        console.error('GTS VIOLATION: Shadow DOM not closed', element);
        return false;
    }
    return true;
}

// Verify autocomplete is disabled
function verifyAutocompleteDisabled(form) {
    if (form.autocomplete !== 'off') {
        console.error('GTS VIOLATION: Autocomplete enabled', form);
        return false;
    }
    return true;
}

// Verify strict mode
function verifyStrictMode() {
    try {
        (function() { 'use strict'; return this; })();
        return false; // Not in strict mode
    } catch (e) {
        return true; // In strict mode
    }
}
```

---

## GTS Security Benefits

| Threat | Mitigation | Effectiveness |
|--------|------------|---------------|
| DOM-based XSS | Shadow DOM closed mode | [32m✓✓✓ High[0m |
| Form Injection | Autocomplete disabled | [32m✓✓✓ High[0m |
| Event Hijacking | Stop propagation | [32m✓✓✓ High[0m |
| Scope Pollution | Strict mode | [32m✓✓✓ High[0m |
| State Leakage | Pipeline isolation | [32m✓✓✓ High[0m |
| Reference Sharing | Deep copy | [32m✓✓✓ High[0m |

---

## GTS Leadership

**Founding Leader:** genesissanga1  
**Authority:** Absolute & Exclusive  
**Methodology:** Genesis Tencen Sanga (GTS)  
**Status:** Active & Enforced

**All GTS implementations MUST follow this guide.**

---

## Quick Decision Tree

```
Is this a web component?
    ├── Yes → Use Shadow DOM with mode='closed'
    └── No → Continue

Is this a form or input?
    ├── Yes → Set autocomplete='off'
    └── No → Continue

Is this an event listener?
    ├── Yes → Call stopImmediatePropagation(), stopPropagation(), preventDefault()
    └── No → Continue

Is this a script?
    ├── Yes → Add 'use strict' at the top
    └── No → Continue

Is this a pipeline?
    ├── Yes → Ensure complete isolation, deep copy all data
    └── No → Continue
```

---

## Remember: The 5 GTS Rules

1. **[31mCLOSED[0m** - Shadow DOM mode must be closed
2. **[31mOFF[0m** - Autocomplete must be off
3. **[31mSTOP[0m** - Event propagation must stop
4. **[31mSTRICT[0m** - Mode must be strict
5. **[31mISOLATE[0m** - Pipelines must be isolated

**Follow all 5, or it's not GTS-compliant!**

---

**Version:** 1.0.0  
**Author:** genesissanga1  
**License:** Apache 2.0
