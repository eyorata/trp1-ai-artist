# TRP 1 – AI Content Generation Submission

## Environment Setup Documentation

### APIs Configured

The following APIs were configured during the environment setup:

- **Google Gemini API**
  - Used for Lyria music generation
  - Intended for Veo video generation
  - API key configured in `.env` as `GEMINI_API_KEY`

- **AIMLAPI**
  - Intended for MiniMax vocal music generation
  - API key configured in `.env` as `AIMLAPI_KEY`

KlingAI credentials were not required and were not used.

---

### Issues Encountered During Setup

- Music generation initially failed due to a missing required `--prompt` argument
- MiniMax vocal generation failed with a `403 Forbidden` error
- Veo video generation failed due to missing or unsupported SDK methods

---

### How Issues Were Resolved

- The missing prompt issue was resolved by explicitly providing a descriptive
  `--prompt`, even when using presets
- The MiniMax error was identified as an external provider requirement
  (AIMLAPI account verification), not a local configuration problem
- The Veo issue was investigated at the code and dependency level and documented
  as a limitation caused by reliance on non-public Google GenAI SDK APIs

---

## Codebase Understanding

### Architecture Description

The project is structured as a modular AI content generation framework:

- **`src/ai_content/core/`**
  - CLI entry points
  - Provider registry
  - Shared exception and result handling

- **`src/ai_content/providers/`**
  - Individual provider implementations (Lyria, MiniMax, Veo)
  - Providers self-register using a registry/decorator pattern

- **`src/ai_content/pipelines/`**
  - Orchestrates prompt handling, preset resolution, and provider execution

- **`src/ai_content/presets/`**
  - Defines reusable presets for music styles and video formats

---

### Provider System Insights

- Providers are dynamically registered and resolved at runtime
- Each provider declares its supported capabilities (e.g., vocals, image-to-video)
- SDK-specific logic is fully encapsulated within providers, isolating failures

Key observations:

- **Lyria** supports instrumental music only
- **MiniMax** supports vocal music and lyrics input
- **Veo** supports text-to-video and image-to-video in theory, but depends on
  non-public Google GenAI SDK APIs

---

### Pipeline Orchestration

1. The CLI parses user arguments and validates required options
2. Presets are applied to generate default parameters
3. The provider is selected via the provider registry
4. The pipeline calls the provider’s `generate()` method
5. Generated content is saved to the `exports/` directory and returned as a
   `GenerationResult` object

---

### Audio Generation – Lyria (Instrumental)

**Prompt rationale:**  
The prompt was chosen to complement the jazz preset while steering the output
toward a relaxed, cinematic feel.

**Command executed:**
```bash
uv run ai-content music \
  --provider lyria \
  --style jazz \
  --prompt "Smooth nostalgic jazz with warm brass and relaxed swing" \
  --duration 30
Result:

Generation succeeded

Audio file saved to exports/

Duration: ~30 seconds

Format: WAV
---
Video Generation – Veo (Attempted)
Command executed:

bash
Copy code
uv run ai-content video \
  --provider veo \
  --style nature \
  --prompt "Cinematic peaceful forest landscape with soft lighting, gentle camera movement, and natural atmosphere" \
  --duration 5
Result:

Generation failed due to missing SDK methods

Observed errors:

types.GenerateVideoConfig not found

client.aio.models.generate_video not implemented

This confirms that the Veo provider relies on non-public or internal Google
GenAI SDK APIs that are not available in the public Python ecosystem.
---
Vocal Music Generation – MiniMax (Attempted)
Command executed:

bash
Copy code
uv run ai-content music \
  --provider minimax \
  --prompt "Emotional cinematic vocal track" \
  --lyrics lyrics.txt
Result:

Request was constructed and submitted correctly

Provider returned 403 Forbidden

Error message indicated that AIMLAPI account verification is required
---
Challenges & Solutions
What Didn’t Work on First Try
Music generation without a prompt failed

MiniMax generation failed due to provider authorization restrictions

Veo video generation failed due to unsupported or unavailable SDK features

Troubleshooting Process
Carefully reviewed CLI error messages

Inspected provider source code (especially veo.py)

Tested execution inside an isolated virtual environment

Verified installed dependencies and compared them against public SDKs

Workarounds Discovered
Providing explicit prompts resolves Lyria generation issues

MiniMax failures must be resolved at the account verification level

Veo cannot be used without access to non-public Google APIs; documenting the
limitation is the correct and transparent approach

Insights & Learnings
What Surprised Me
Presets do not eliminate the need for prompts

The Veo provider appears to target an internal or experimental Google SDK

The provider abstraction cleanly isolates failures without crashing the CLI

What I Would Improve
Add runtime validation for provider capability availability

Improve error messaging for unsupported or non-public providers

Add fallback or mock providers for video generation

Comparison to Other AI Tools
Compared to other AI generation tools, this framework is more transparent and
developer-oriented. While it is less plug-and-play, it provides clearer insight
into provider behavior, orchestration logic, and failure modes.

Links
YouTube Video: Not available (video generation blocked by SDK limitation)

GitHub Repository: https://github.com/eyorata/trp1-ai-artist
