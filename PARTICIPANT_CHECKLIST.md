# Workshop Participant Checklist

Use this checklist to track your progress through the React Alicante AI Agent Workshop.

## Pre-Workshop Setup ✓

- [ ] Node.js installed (v18 or higher)
- [ ] npm or yarn installed
- [ ] Git installed
- [ ] Code editor/IDE ready (VS Code, WebStorm, etc.)
- [ ] Modern browser installed (Chrome 113+ or Edge 113+ for WebGPU)
- [ ] Repository cloned: `git clone https://github.com/shivaycb/reactalicante-agent-workshop.git`
- [ ] Dependencies installed: `npm install`
- [ ] Google Generative AI API key obtained from https://aistudio.google.com/apikey
- [ ] `.env` file created with API key
- [ ] Dev server runs successfully: `npm run dev`
- [ ] Application opens in browser at http://localhost:5173

---

## Workshop Steps

### Step 0: Understand the Starting Point ✓
- [ ] Explored the application UI (products, cart, FAQ)
- [ ] Reviewed project structure (`src/`, `public/`, config files)
- [ ] Located the chat component: `src/app/chat/Chat.tsx`
- [ ] Identified pre-built utilities in `src/utils/agent/`
- [ ] Understood workshop goals

**Notes:**
```
[Your notes here]
```

---

### Step 1: Configure WebLLM for Local Inference ✓
**Commit**: `24a662aa` - "webllm config"

- [ ] Created new file: `src/utils/agent/webllm.ts`
- [ ] Added `GEMMA_2_9B_CONFIG` export with model URL
- [ ] Added `GEMMA_2_2B_ID` export
- [ ] Understood quantization (q4f16) concept
- [ ] Learned about WebGPU requirements

**Key Learnings:**
- [ ] What WebLLM does
- [ ] Why models are large (~5-6GB)
- [ ] Local vs cloud trade-offs

**Notes:**
```
[Your notes here]
```

---

### Step 2: Implement WebLLM Wrapper Class ✓
**Commit**: `1e3f8a02` - "webllm"

- [ ] Created new file: `src/ai/llm/WebLLM.ts`
- [ ] Imported WebLLM dependencies
- [ ] Implemented `WebLLM` class with private `engine` property
- [ ] Implemented `createConversation()` method
- [ ] Implemented `generate()` method with lazy initialization
- [ ] Added message history management (system, user, assistant roles)
- [ ] Added console logging for debugging
- [ ] Tested in browser console (optional)

**Key Learnings:**
- [ ] Lazy initialization pattern
- [ ] Conversation state management
- [ ] Temperature parameter
- [ ] Message roles (system, user, assistant)

**Notes:**
```
[Your notes here]
```

---

### Step 3: Implement Cloud-Based Gemini LLM ✓
**Commit**: `25ff0b56` - "add gemini llm"

- [ ] Created new file: `src/ai/llm/GeminiLlm.ts`
- [ ] Defined TypeScript interfaces for Gemini API response
- [ ] Implemented `GeminiLlm` class
- [ ] Implemented `postApi()` private method
- [ ] Implemented `createConversation()` method
- [ ] Added API key validation
- [ ] Handled message format transformation (user/model roles)
- [ ] Tested with API key (optional)

**Key Learnings:**
- [ ] REST API integration with LLMs
- [ ] Environment variable usage in Vite
- [ ] Interface compatibility (duck typing)
- [ ] Cloud vs local LLM comparison

**Notes:**
```
[Your notes here]
```

---

### Step 4: Build AI Agent with RAG ✓
**Commit**: `afa86c8b` - "agent added"

#### Part A: Feature Extraction
- [ ] Created new file: `src/utils/vectorSearch/FeatureExtraction.ts`
- [ ] Imported Transformers.js dependencies
- [ ] Implemented `FeatureExtraction` class
- [ ] Implemented `extract()` method with lazy initialization
- [ ] Configured WebGPU device
- [ ] Added pooling and normalization

**Key Learnings:**
- [ ] What embeddings are
- [ ] all-MiniLM-L6-v2 model (384 dimensions)
- [ ] Transformers.js and ONNX Runtime
- [ ] WebGPU acceleration

#### Part B: FAQ Search
- [ ] Created new file: `src/ai/vectorSearch/findSimilarFAQs.ts`
- [ ] Implemented `findSimilarFAQs()` function
- [ ] Added query vector extraction
- [ ] Implemented similarity calculation loop
- [ ] Added sorting and top-N selection
- [ ] Tested FAQ search (optional)

**Key Learnings:**
- [ ] Vector search flow
- [ ] Cosine similarity
- [ ] Pre-computed embeddings
- [ ] RAG concept

#### Part C: Agent Class
- [ ] Created new file: `src/ai/agent/Agent.ts`
- [ ] Implemented `Agent` class with WebLLM
- [ ] Implemented `addTool()` method
- [ ] Implemented `processPrompt()` method
- [ ] Added agentic loop (think → act → observe)
- [ ] Added function call parsing
- [ ] Added tool execution with parallel processing
- [ ] Added render callback support
- [ ] Added max rounds limit
- [ ] Tested agent with simple tool (optional)

**Key Learnings:**
- [ ] Agentic AI pattern
- [ ] Function calling with XML
- [ ] Tool registration and execution
- [ ] Agent reasoning loop
- [ ] Tool interface (description, parameters, execute, examples)

**Notes:**
```
[Your notes here]
```

---

### Step 5: Fix System Prompt Handling ✓
**Commit**: `84fc504e` - "fix system prompt"

- [ ] Modified `src/ai/llm/WebLLM.ts`
- [ ] Fixed syntax error on line 9
- [ ] Changed `{ role: "system", content:  },systemPrompt` to `{ role: "system", content: systemPrompt },`
- [ ] Understood importance of system prompts

**Key Learnings:**
- [ ] Always test with actual LLM
- [ ] System prompts control agent behavior

**Notes:**
```
[Your notes here]
```

---

### Step 6: Integrate Agent into Chat UI ✓
**Commit**: `e3079587` - "agent added to chat"

- [ ] Modified `src/app/chat/Chat.tsx`
- [ ] Imported Zod, Agent, tool, and findSimilarFAQs
- [ ] Created agent instance with `useMemo`
- [ ] Implemented `faqSearch` tool with:
  - [ ] Description
  - [ ] Zod schema parameters
  - [ ] Execute function
  - [ ] Render function
  - [ ] Examples
- [ ] Implemented `navigateToPage` tool with:
  - [ ] Description
  - [ ] Zod schema parameters (category, color, size)
  - [ ] Execute function with navigation
  - [ ] Render function
  - [ ] Examples
- [ ] Updated form submit handler to use agent
- [ ] Added thinking state management
- [ ] Added tool render elements state
- [ ] Added error handling
- [ ] Updated UI to display tool results
- [ ] Tested with various queries

**Test Queries Tried:**
- [ ] FAQ: "What's your return policy?"
- [ ] FAQ: "How long does shipping take?"
- [ ] Navigation: "Show me red shirts"
- [ ] Navigation: "I want large blue accessories"
- [ ] Combined: "What's your return policy and show me hoodies"
- [ ] Edge case: Empty query
- [ ] Edge case: Nonsense query

**Key Learnings:**
- [ ] Tool design principles
- [ ] Zod schema validation
- [ ] React integration patterns
- [ ] Loading states and UX
- [ ] Error handling
- [ ] Tool rendering in UI

**Notes:**
```
[Your notes here]
```

---

## Post-Workshop

### Understanding Check ✓
Rate your understanding (1-5, where 5 is "I can explain this to others"):

- [ ] LLM Integration (Local vs Cloud): ___/5
- [ ] AI Agents and Function Calling: ___/5
- [ ] RAG Systems: ___/5
- [ ] Vector Embeddings and Semantic Search: ___/5
- [ ] React Integration: ___/5

### Challenges Faced
List the most challenging parts:
```
1. 
2. 
3. 
```

### Favorite Concepts
What was most interesting:
```
1. 
2. 
3. 
```

### Next Steps ✓

**Immediate (This Week):**
- [ ] Review workshop notes
- [ ] Re-run the application
- [ ] Try adding a new tool (e.g., weather, calculator)
- [ ] Experiment with different system prompts
- [ ] Share workshop experience (blog, social media)

**Short-term (This Month):**
- [ ] Build a small project using these concepts
- [ ] Add conversation history to the chat
- [ ] Implement streaming responses
- [ ] Try different embedding models
- [ ] Create custom tools for your use case

**Long-term (This Quarter):**
- [ ] Explore LangChain.js or LlamaIndex
- [ ] Build a production-ready RAG system
- [ ] Experiment with multi-agent systems
- [ ] Contribute to open source AI projects
- [ ] Share learnings with the community

### Resources to Explore ✓

**Documentation:**
- [ ] [WebLLM Docs](https://mlc.ai/web-llm/)
- [ ] [Transformers.js](https://huggingface.co/docs/transformers.js)
- [ ] [Google Gemini](https://ai.google.dev/docs)
- [ ] [Vercel AI SDK](https://sdk.vercel.ai/docs)
- [ ] [Zod](https://zod.dev/)

**Learning Resources:**
- [ ] [Hugging Face Course](https://huggingface.co/learn)
- [ ] [LangChain Documentation](https://js.langchain.com/)
- [ ] [Vector Database Comparison](https://www.pinecone.io/)
- [ ] [RAG Best Practices](https://www.llamaindex.ai/)

**Community:**
- [ ] Join Hugging Face Discord
- [ ] Follow AI Stack Devs
- [ ] Participate in LocalAI community
- [ ] Share your project on Twitter/LinkedIn

### Project Ideas ✓

Ideas to build with these skills:
```
1. 
2. 
3. 
```

---

## Feedback for Workshop Host

**What went well:**
```
[Your feedback here]
```

**What could be improved:**
```
[Your feedback here]
```

**Additional topics you'd like covered:**
```
[Your feedback here]
```

**Would you recommend this workshop to others? (1-10):** ___/10

**Why or why not?**
```
[Your answer here]
```

---

## Additional Notes

Use this space for any additional notes, insights, or reminders:

```
[Your notes here]
```

---

**Workshop Completed!** 🎉

Date: _______________  
Time Spent: _______________  
Overall Rating: ___/5 ⭐

**Next Workshop Goal:** 
```
[Your goal here]
```

---

**Keep Learning, Keep Building! 🚀**

Remember: The best way to learn is to build something. Take what you learned and create!

Share your progress:
- GitHub: _______________
- Twitter: _______________
- Blog: _______________
- LinkedIn: _______________
