# Vision Delegation Workaround (Non-Vision Models)

## Problem

You use a primary model that doesn't support image inputs (e.g. **DeepSeek**, **MiniMax**, **Llama**, **Mistral**) but want to drop images into chat and have the agent understand them.

## Solution: Delegation to a Vision-Capable Subagent

Configure Hermes's `delegation` section to use a vision-capable model. When an image arrives, the primary model delegates image analysis to the subagent, which uses the `vision` toolset to describe it, then returns the description so the conversation continues normally.

## Config Setup

Edit `~/.hermes/config.yaml`:

```yaml
delegation:
  provider: openrouter
  model: anthropic/claude-sonnet-4
  base_url: ''
  api_key: ''
  inherit_mcp_toolsets: true
```

No `api_key` needed if the environment variable is already set (e.g. `OPENROUTER_API_KEY`).

### Recommended Vision Models (via OpenRouter)

| Model | ID | Notes |
|-------|-----|-------|
| **Claude Sonnet 4** | `anthropic/claude-sonnet-4` | Best all-rounder, excellent at screenshots & diagrams |
| **GPT-4o** | `openai/gpt-4o` | Fast, cheap, great vision |
| **Gemini 2.0 Flash** | `google/gemini-2.0-flash-001` | Very fast, free tier available |
| **GPT-4o Mini** | `openai/gpt-4o-mini` | Cheapest option with vision |
| **Claude 3.5 Sonnet** | `anthropic/claude-3.5-sonnet` | Reliable fallback if Sonnet 4 is overloaded |

### With Other Providers

```yaml
# Anthropic directly
delegation:
  provider: anthropic
  model: claude-sonnet-4-20250514

# OpenAI directly
delegation:
  provider: openai
  model: gpt-4o

# Google Gemini
delegation:
  provider: google
  model: gemini-2.0-flash-001
```

## How the Agent Decides (Decision Logic)

When the agent (non-vision model) receives a message containing an image file path:

1. **Image-only message** → Delegate: "Describe this image in detail"
2. **Text extraction needed** ("read this", "OCR this") → Try `tesseract` first (local, free, instant), fall back to delegation
3. **Visual content question** ("what is this?", "explain this diagram") → Delegate with the user's question as context
4. **Text + image** → Delegate with both: "The user asks: [question]. Analyze the image and answer."
5. **Multiple images** → One delegation call per image or batched together

## Pitfalls

- **Cost**: Each delegation call costs ~5-10¢ on vision models. Not an issue for occasional use but add up at scale.
- **Latency**: ~5-10 second delay while the subagent processes the image and returns a description.
- **Context bloat**: The subagent's description gets injected into the conversation, consuming tokens on the primary model too.
- **No streaming**: The primary model waits for the full delegation result before continuing.
- **Failover**: If the delegation model is unavailable, the agent should notify the user rather than silently continuing without the image analysis.
- **Model doesn't support vision**: Verify the chosen model actually has vision capabilities — not all Claude/GPT/Gemini variants do. Check provider docs.

## Verification

After setting up, test by sending an image containing text (e.g. a screenshot of code) and asking "what does this say?" If the delegation is working, the response will accurately describe the image.

## Fallback: Local OCR (Free Alternative)

For text-only images (screenshots of code, documents, signs), `tesseract-ocr` is a zero-cost alternative:

```bash
sudo apt install tesseract-ocr
tesseract /path/to/image.png -
```

The agent can use this as a first pass before falling back to delegation.
