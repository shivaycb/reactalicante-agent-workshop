# Workshop Quick Reference Guide

This is a condensed reference for the React Alicante AI Agent Workshop. For complete host notes, see [WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md).

## Workshop Flow

### Step 1: Initialize Workshop (20 min)
- **Commit:** `0f6414e` - "Initialize Workshop"
- **Goal:** Setup React webshop application
- **Key Files:** `src/App.tsx`, `src/store/products.ts`, `src/store/faq.ts`
- **Action:** Explore the base application structure

### Step 2: WebLLM Config (15 min)
- **Commit:** `24a662aa` - "webllm config"  
- **Goal:** Configure local AI model
- **New File:** `src/utils/agent/webllm.ts`
- **Action:** Add Gemma 2 9B model configuration

### Step 3: WebLLM Implementation (30 min)
- **Commit:** `1e3f8a02` - "webllm"
- **Goal:** Implement WebLLM wrapper
- **New File:** `src/ai/llm/WebLLM.ts`
- **Action:** Create LLM conversation interface

### Step 4: Gemini LLM (30 min)
- **Commit:** `25ff0b56` - "add gemini llm"
- **Goal:** Add cloud-based AI alternative
- **New File:** `src/ai/llm/GeminiLlm.ts`
- **Action:** Implement Gemini API integration

### Step 5: Agent & Vector Search (45 min)
- **Commit:** `afa86c8b` - "agent added"
- **Goal:** Build agentic system with tools
- **New Files:**
  - `src/ai/agent/Agent.ts`
  - `src/ai/vectorSearch/findSimilarFAQs.ts`
  - `src/utils/vectorSearch/FeatureExtraction.ts`
- **Action:** Implement tool calling and semantic search

### Step 6: System Prompt Fix (10 min)
- **Commit:** `84fc504e` - "fix system prompt"
- **Goal:** Improve debugging and prompt engineering
- **Modified:** `src/ai/llm/WebLLM.ts`
- **Action:** Fix system prompt logging

### Step 7: Chat Integration (45 min)
- **Commit:** `e3079587` - "agent added to chat"
- **Goal:** Complete the AI assistant
- **Modified:** `src/app/chat/Chat.tsx` (+299 lines)
- **Action:** Integrate agent with UI and add tools

## Key Technologies

| Technology | Purpose | When to Use |
|------------|---------|-------------|
| WebLLM | Local AI inference | Privacy, offline, free |
| Google Gemini | Cloud AI | Speed, quality, scale |
| Transformers.js | Vector embeddings | Semantic search in browser |
| Zod | Type-safe schemas | Tool parameter validation |

## Common Commands

```bash
# Development
npm run dev              # Start dev server (http://localhost:5173)
npm run build            # Build for production
npm run lint             # Run ESLint

# Git Navigation
git checkout main        # Starting point
git checkout complete    # Final solution
git log --oneline        # View commit history

# Environment Variables
VITE_GOOGLE_GENERATIVE_AI_API_KEY   # Required for Gemini
VITE_USE_WEBLLM=true|false          # Toggle LLM backend
```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         Chat UI                             │
│                    (src/app/chat/Chat.tsx)                  │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      Agent                                   │
│                 (src/ai/agent/Agent.ts)                      │
│                                                              │
│  ┌──────────┐  ┌──────────┐  ┌────────────────────────┐   │
│  │ System   │  │  Tools   │  │  Tool Execution Loop    │   │
│  │ Prompt   │  │Registry  │  │  (ReAct Pattern)        │   │
│  └──────────┘  └──────────┘  └────────────────────────┘   │
└──────────────┬──────────────────────┬───────────────────────┘
               │                      │
               ▼                      ▼
┌──────────────────────┐   ┌────────────────────────┐
│   LLM Interface      │   │   Tools                │
│                      │   │                        │
│ ┌────────────────┐   │   │ ┌──────────────────┐  │
│ │  WebLLM (Local)│   │   │ │ openProductPage  │  │
│ │  - Gemma 2 9B  │   │   │ └──────────────────┘  │
│ └────────────────┘   │   │ ┌──────────────────┐  │
│ ┌────────────────┐   │   │ │   searchFAQ      │  │
│ │ Gemini (Cloud) │   │   │ │  (Vector Search) │  │
│ │  - 1.5 Flash   │   │   │ └──────────────────┘  │
│ └────────────────┘   │   └────────────────────────┘
└──────────────────────┘
```

## Tool Definition Template

```typescript
agent.addTool(
  "toolName",
  tool({
    description: "What this tool does",
    parameters: z.object({
      param1: z.string().describe("Description for LLM"),
      param2: z.array(z.enum(["A", "B"])).optional(),
    }),
    execute: async (data) => {
      // Perform action
      return {
        nextPrompt: "Tell the LLM what happened",
        render: () => <UIFeedback />
      };
    },
    examples: [
      {
        query: "Example user query",
        parameters: { param1: "value" }
      }
    ],
  })
);
```

## System Prompt Template

```typescript
const systemPrompt = `You are a [ROLE].

Your responsibilities:
- [Responsibility 1]
- [Responsibility 2]

When users ask about [X], use the [TOOL] tool.
Always [CONSTRAINT].

Be [TONE_ADJECTIVE] in your responses.`;
```

## Troubleshooting Quick Fix

| Problem | Quick Fix |
|---------|-----------|
| WebGPU not supported | Set `VITE_USE_WEBLLM=false` |
| Model download fails | Try smaller model or use Gemini |
| Tools not called | Add more examples to tool definition |
| Slow responses | Switch to Gemini (faster) |
| API key error | Check .env file, restart server |

## Testing Queries

### Product Search
- "Show me all red products"
- "I want hoodies in size M"
- "Find me accessories"

### FAQ Search  
- "What's your return policy?"
- "How much does shipping cost?"
- "Do you ship internationally?"

### Multi-Turn
- "Show me t-shirts" → "What colors are available?"
- "Search for caps" → "Do you have any in blue?"

## Key Concepts Explained

### Vector Embeddings
- Text → Array of numbers (e.g., 384 dimensions)
- Similar meanings → Similar vectors
- Enables semantic search (not just keywords)
- Model: `Xenova/all-MiniLM-L6-v2`

### ReAct Pattern (Reasoning + Acting)
```
1. User Query → LLM
2. LLM decides → Call Tool or Answer
3. If Tool → Execute → Get Result
4. Result → Back to LLM
5. Repeat until Answer (max 5 rounds)
```

### Tool Calling
- LLM generates XML: `<function_call name="tool">...</function_call>`
- Agent parses XML, extracts function name and parameters
- Validates parameters with Zod schema
- Executes function
- Returns result to LLM as next prompt

## Extension Ideas

### Easy
- Add more products to catalog
- Customize system prompt personality
- Add more FAQ entries

### Medium
- Add `addToCart` tool
- Implement conversation history UI
- Add streaming responses
- Create `getProductDetails` tool

### Hard
- Voice input/output with Web Speech API
- Multi-modal search (image upload)
- Conversation memory across sessions
- Custom embeddings for products

## Resources

- **WebLLM Docs:** https://mlc.ai/web-llm/
- **Gemini API:** https://ai.google.dev/
- **Transformers.js:** https://huggingface.co/docs/transformers.js/
- **Zod:** https://zod.dev/
- **Prompt Engineering:** https://docs.anthropic.com/claude/docs/prompt-engineering

## Workshop Success Criteria

By the end, participants should:
- ✅ Understand local vs cloud AI tradeoffs
- ✅ Implement LLM wrappers in TypeScript
- ✅ Build semantic search with embeddings
- ✅ Create agentic systems with tool calling
- ✅ Define type-safe tools with Zod
- ✅ Integrate AI into React apps
- ✅ Debug agent behavior
- ✅ Extend the system independently

---

**For detailed host instructions, timing, and troubleshooting, see [WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md)**
