# Workshop Quick Reference Guide

## Commit History & Workshop Steps

| Step | Commit SHA | Title | Files Changed | Difficulty | Time |
|------|-----------|-------|---------------|------------|------|
| 0 | `e1ac9d83` | Initial commit | .gitignore, LICENSE | - | - |
| 0 | `0f6414e3` | Initialize Workshop | Full React app structure | - | 10m |
| 1 | `24a662aa` | webllm config | `src/utils/agent/webllm.ts` | Easy | 10m |
| 2 | `1e3f8a02` | webllm | `src/ai/llm/WebLLM.ts` | Medium | 15m |
| 3 | `25ff0b56` | add gemini llm | `src/ai/llm/GeminiLlm.ts` | Medium | 20m |
| 4 | `afa86c8b` | agent added | `src/ai/agent/Agent.ts`, `src/ai/vectorSearch/findSimilarFAQs.ts`, `src/utils/vectorSearch/FeatureExtraction.ts` | Hard | 30m |
| 5 | `84fc504e` | fix system prompt | `src/ai/llm/WebLLM.ts` | Easy | 5m |
| 6 | `e3079587` | agent added to chat | `src/app/chat/Chat.tsx` | Medium-Hard | 30m |

**Total Time**: ~2h 30m (including breaks)

---

## Quick Commands

### Setup
```bash
# Clone and setup
git clone https://github.com/shivaycb/reactalicante-agent-workshop.git
cd reactalicante-agent-workshop
npm install

# Start dev server
npm run dev
```

### Jump to Specific Step
```bash
# View a specific commit
git show <commit-sha>

# Checkout to specific step (creates new branch)
git checkout <commit-sha> -b step-N

# Return to workshop branch
git checkout main
```

### View Changes Between Steps
```bash
# See what changed in a step
git diff <previous-commit>..<current-commit>

# Example: See step 2 changes
git diff 24a662aa..1e3f8a02
```

---

## Environment Variables

Create `.env` file in project root:
```env
PORT=5173
VITE_GOOGLE_GENERATIVE_AI_API_KEY=your_api_key_here
```

Get API key: https://aistudio.google.com/apikey

---

## Key Concepts Summary

### Step 1: WebLLM Config
- **File**: `src/utils/agent/webllm.ts`
- **Concept**: Model configuration for local inference
- **Key Points**: 
  - Gemma 2 9B model (quantized: q4f16)
  - WebGPU acceleration
  - ~5-6GB model download

### Step 2: WebLLM Class
- **File**: `src/ai/llm/WebLLM.ts`
- **Concept**: LLM wrapper for conversation management
- **Key Points**:
  - Lazy initialization
  - Message history (system, user, assistant)
  - Temperature parameter

### Step 3: Gemini LLM
- **File**: `src/ai/llm/GeminiLlm.ts`
- **Concept**: Cloud-based LLM integration
- **Key Points**:
  - REST API integration
  - Similar abstraction layer to WebLLM, but different `generate(...)` signature (callback vs temperature), so not directly interchangeable without adapter code
  - Requires API key

### Step 4: Agent + RAG
- **Files**: 
  - `src/ai/agent/Agent.ts`
  - `src/ai/vectorSearch/findSimilarFAQs.ts`
  - `src/utils/vectorSearch/FeatureExtraction.ts`
- **Concepts**: 
  - Agentic reasoning loop
  - Function calling (XML-based)
  - Vector embeddings (all-MiniLM-L6-v2)
  - Semantic search (cosine similarity)
  - RAG pattern
- **Key Points**:
  - Agent loop: Think → Act → Observe → Repeat
  - Tools with Zod schemas
  - 384-dimensional embeddings
  - Max 5 rounds

### Step 5: System Prompt Fix
- **File**: `src/ai/llm/WebLLM.ts`
- **Concept**: Bug fix (syntax error)
- **Key Points**: Always test with real LLM

### Step 6: Chat Integration
- **File**: `src/app/chat/Chat.tsx`
- **Concept**: Full React integration
- **Key Points**:
  - Two tools: faqSearch, navigateToPage
  - Loading states
  - Tool result rendering
  - Error handling

---

## Common Issues

### WebGPU Not Available
```javascript
// Check support
console.log(navigator.gpu !== undefined);

// Enable in Chrome: chrome://flags/#enable-unsafe-webgpu
```

### API Key Not Working
```bash
# Restart dev server after adding .env
npm run dev

# Verify in browser console
console.log(import.meta.env.VITE_GOOGLE_GENERATIVE_AI_API_KEY);
```

### Model Download Slow
- First download: 5-6GB (be patient!)
- Subsequent loads: cached
- Alternative: Use Gemini (cloud)

### Agent Not Using Tools
- Check system prompt includes tool descriptions
- Verify tool examples are clear
- Log LLM response to debug

---

## Testing Snippets

### Test WebLLM
```javascript
const llm = new WebLLM();
const conv = llm.createConversation("You are helpful.");
await conv.generate("Hello!");
```

### Test Gemini
```javascript
const llm = new GeminiLlm();
const conv = llm.createConversation("You are helpful.");
await conv.generate("Hello!");
```

### Test Embeddings
```javascript
const extractor = new FeatureExtraction();
const vec = await extractor.extract("test query");
console.log(vec.length); // Should be 384
```

### Test FAQ Search
```javascript
const results = await findSimilarFAQs("How do I return items?");
console.log(results);
```

### Test Agent
```javascript
const agent = new Agent();
agent.addTool("time", tool({
  description: "Get current time",
  parameters: z.object({}),
  execute: async () => ({
    nextPrompt: `Time: ${new Date().toISOString()}`
  }),
  examples: [{ query: "what time?", parameters: {} }]
}));

const answer = await agent.processPrompt(
  "You are helpful.",
  "What time is it?"
);
console.log(answer);
```

---

## Demo Queries

### FAQ Questions
- "What's your return policy?"
- "How long does shipping take?"
- "Do you ship internationally?"

### Navigation
- "Show me red shirts"
- "I want large blue accessories"
- "Show me all yellow items"

### Combined
- "What's your return policy and show me some hoodies"
- "Do you have green items in size M?"

---

## File Structure

```
src/
├── ai/
│   ├── agent/
│   │   └── Agent.ts              # Step 4: Agent with reasoning loop
│   ├── llm/
│   │   ├── WebLLM.ts             # Step 2: Local LLM wrapper
│   │   └── GeminiLlm.ts          # Step 3: Cloud LLM wrapper
│   └── vectorSearch/
│       └── findSimilarFAQs.ts    # Step 4: Semantic FAQ search
├── app/
│   └── chat/
│       └── Chat.tsx              # Step 6: UI integration
├── utils/
│   ├── agent/
│   │   ├── webllm.ts             # Step 1: Model config
│   │   ├── tool.ts               # Pre-built: Tool type definitions
│   │   ├── parseXmlFunctionCalls.ts  # Pre-built: Parse LLM output
│   │   └── toolsToSystemPrompt.ts    # Pre-built: Generate prompts
│   └── vectorSearch/
│       ├── FeatureExtraction.ts  # Step 4: Embedding extraction
│       └── cosineSimilarity.ts   # Pre-built: Vector similarity
└── store/
    ├── faq.ts                    # FAQ data with pre-computed vectors
    └── products.ts               # Product catalog
```

---

## Architecture Diagrams

### Agent Loop
```
User Query
    ↓
[Agent] Add tools to system prompt
    ↓
[LLM] Generate response (may include function calls)
    ↓
[Agent] Parse XML function calls
    ↓
[Agent] Execute tools in parallel
    ↓
[Agent] Add results to next prompt
    ↓
[LLM] Generate response... (repeat max 5 times)
    ↓
Final Answer
```

### RAG Flow
```
User Question
    ↓
[FeatureExtraction] Convert to vector (384-dim)
    ↓
[findSimilarFAQs] Compare with FAQ vectors
    ↓
[cosineSimilarity] Calculate similarity scores
    ↓
[findSimilarFAQs] Return top N FAQs
    ↓
[Agent] Add to LLM context
    ↓
[LLM] Generate answer using retrieved info
```

---

## Resources

### Documentation
- [WebLLM](https://mlc.ai/web-llm/)
- [Transformers.js](https://huggingface.co/docs/transformers.js)
- [Google Gemini](https://ai.google.dev/docs)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)

### Models
- [Gemma Models](https://ai.google.dev/gemma)
- [all-MiniLM-L6-v2](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)
- [WebLLM Model Library](https://github.com/mlc-ai/mlc-llm)

### Tools
- [Zod](https://zod.dev/) - Schema validation
- [React Router](https://reactrouter.com/) - Navigation
- [Tailwind CSS](https://tailwindcss.com/) - Styling

---

## Workshop Tips

### For Hosts
1. **Pre-cache models** on WiFi before workshop
2. **Have backup API keys** ready for Gemini
3. **Pair participants** (beginner + advanced)
4. **Use commits as checkpoints** for catching up
5. **Live code key sections**, provide utilities
6. **Keep it practical** - show it works first, explain theory after

### For Participants
1. **Don't fall behind** - checkout commits if needed
2. **Ask questions** - no question is too basic
3. **Experiment** - try different queries and tools
4. **Debug with console.log** - add logs everywhere
5. **Take notes** - document your learning
6. **Share discoveries** - help others when you figure things out

---

## Time Management

If running behind schedule:
1. **Skip Step 3** (Gemini) - focus on WebLLM
2. **Provide code for Steps 1 & 5** (config and fix)
3. **Demo Step 6** instead of coding it
4. **Shorten wrap-up** to 5 minutes

Critical steps you cannot skip:
- **Step 2**: WebLLM class (core concept)
- **Step 4**: Agent + RAG (core concept)
- **Step 6**: Integration (see it work)

---

## Post-Workshop

### What to Build Next
- Add more tools (weather, search, calculator)
- Implement conversation history
- Add voice input/output
- Create custom embeddings for your domain
- Build multi-agent systems
- Add tool usage analytics

### Share Your Work
- GitHub repo with your implementation
- Blog post about your experience
- Twitter/LinkedIn with demo video
- Help others in Discord/Slack

### Continue Learning
- [LangChain.js](https://js.langchain.com/)
- [LlamaIndex](https://www.llamaindex.ai/)
- [Pinecone](https://www.pinecone.io/)
- [Hugging Face Course](https://huggingface.co/learn)

---

**Happy Building! 🚀**
