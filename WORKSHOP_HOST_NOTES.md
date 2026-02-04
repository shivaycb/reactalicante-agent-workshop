# Workshop Host Notes: React Alicante AI Agent Workshop

## Workshop Overview

This hands-on workshop teaches participants how to build AI-powered features into a React application, progressing from basic LLM integration to a fully functional AI agent with RAG (Retrieval-Augmented Generation) capabilities.

**Duration**: 2-3 hours  
**Skill Level**: Intermediate (React experience required)  
**Prerequisites**: 
- Node.js and npm installed
- Basic understanding of React, TypeScript, and async/await
- Text editor or IDE

## Workshop Learning Objectives

By the end of this workshop, participants will:
1. ✅ Understand local vs. cloud-based LLM inference
2. ✅ Implement conversation management with LLMs
3. ✅ Build an AI agent with function calling capabilities
4. ✅ Implement RAG using vector embeddings and semantic search
5. ✅ Integrate AI agents into React applications

---

## Repository Setup

**Main Branch**: `main` - Starting point (blank webshop with no AI features)  
**Complete Branch**: `complete` - Finished implementation with all AI features

Each commit in the repository represents a workshop step that participants will implement.

---

## Step 0: Initial Setup (Before Workshop)

**Commits**: 
- `e1ac9d83` - "Initial commit"
- `0f6414e3` - "Initialize Workshop"

### Host Preparation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/shivaycb/reactalicante-agent-workshop.git
   cd reactalicante-agent-workshop
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Setup environment variables** (`.env` file):
   ```
   PORT=5173  # Optional
   VITE_GOOGLE_GENERATIVE_AI_API_KEY=your_api_key_here
   ```
   - Get API key from: https://aistudio.google.com/apikey

4. **Start the development server**:
   ```bash
   npm run dev
   ```

5. **Verify the application loads**:
   - Open http://localhost:5173
   - You should see a working webshop with products, cart, and FAQ

### Introduction (10 minutes)

**Welcome & Context**:
- This is an e-commerce demo application ("Swag Shop")
- Features: Product browsing, shopping cart, FAQ section
- Goal: Add AI-powered assistant capabilities

**Key Files to Highlight**:
- `src/App.tsx` - Main application
- `src/app/chat/Chat.tsx` - Chat interface (to be enhanced)
- `src/store/` - Data management (products, FAQ)
- `src/ai/` - AI components (to be built)
- `src/utils/agent/` - Agent framework utilities (pre-built)

**Pre-built Utilities** (participants don't need to write these):
- `tool.ts` - Type definitions for agent tools
- `parseXmlFunctionCalls.ts` - Parses XML function calls from LLM responses
- `toolsToSystemPrompt.ts` - Generates system prompts with tool descriptions
- `generateSchemaDescription.ts` - Converts Zod schemas to descriptions
- `cosineSimilarity.ts` - Vector similarity calculation

---

## Step 1: Configure WebLLM for Local Inference

**Commit**: `24a662aa` - "webllm config"  
**Time**: 10 minutes  
**Difficulty**: Easy

### Overview
Set up configuration for running LLMs locally in the browser using WebLLM. This enables privacy-preserving, offline AI capabilities.

### What Participants Will Learn
- WebLLM basics and model configuration
- WebGPU requirements
- Model formats (quantization: q4f16)

### Implementation

**Create**: `src/utils/agent/webllm.ts`

```typescript
export const GEMMA_2_9B_CONFIG = {
  model: "https://uploads.nico.dev/mlc-llm-libs/gemma-2-9b-it_q4f16_MLC/",
  model_id: "gemma-2-9b-it_q4f16_MLC",
  model_lib:
    "https://uploads.nico.dev/mlc-llm-libs/gemma-2-9b-it_q4f16_MLC/lib/gemma-2-9b-it-q4f16_1-webgpu.wasm",
};

export const GEMMA_2_2B_ID = "gemma-2-2b-it-q4f16_1-MLC";
```

### Teaching Points

**What is WebLLM?**
- Runs LLMs directly in the browser using WebGPU
- No server required - completely client-side
- Great for privacy-sensitive applications
- Requires modern browser with WebGPU support (Chrome 113+, Edge 113+)

**Model Selection**:
- **Gemma 2 9B**: Larger, more capable (9 billion parameters)
- **Gemma 2 2B**: Smaller, faster (2 billion parameters)
- **q4f16**: Quantization format (4-bit weights, 16-bit float activations)
  - Reduces model size by ~75%
  - Maintains most performance
  - Enables running larger models in browser

**Model Hosting**:
- Custom URL points to pre-converted WebLLM models
- Includes both model weights and WASM runtime
- First load downloads ~5-6GB (cached afterward)

### Common Issues & Troubleshooting

❌ **WebGPU not available**:
- Check browser version (Chrome/Edge 113+)
- Enable flags: `chrome://flags/#enable-unsafe-webgpu`
- Some older GPUs not supported

❌ **Download fails**:
- Check network connection
- Verify URL accessibility
- Models are large (5-6GB) - may take time

❌ **Out of memory**:
- Use smaller model (2B instead of 9B)
- Close other tabs/applications
- Need GPU with at least 8GB VRAM for 9B model

### Demo Points
- Show the model URL in browser - it's just static files!
- Explain quantization impact on model size
- Discuss trade-offs: local (privacy, offline) vs cloud (power, speed)

---

## Step 2: Implement WebLLM Wrapper Class

**Commit**: `1e3f8a02` - "webllm"  
**Time**: 15 minutes  
**Difficulty**: Medium

### Overview
Create a wrapper class that manages the WebLLM engine and conversation state, providing a clean API for generating responses.

### What Participants Will Learn
- Singleton pattern for engine management
- Conversation state management
- Chat message format (system, user, assistant roles)
- Async initialization and generation

### Implementation

**Create**: `src/ai/llm/WebLLM.ts`

```typescript
import {
  ChatCompletionMessageParam,
  CreateMLCEngine,
  MLCEngine,
} from "@mlc-ai/web-llm";

import { GEMMA_2_9B_CONFIG } from "../../utils/agent/webllm.ts";

class WebLLM {
  private engine: MLCEngine | null = null;

  public createConversation = (systemPrompt: string) => {
    const messages: Array<ChatCompletionMessageParam> = [
      { role: "system", content: systemPrompt },
    ];

    console.log("-- SYSTEM PROMPT --");
    console.log(systemPrompt);

    return {
      generate: async (prompt: string, temperature: number = 1) => {
        messages.push({ role: "user", content: prompt });
        console.log("-- MESSAGES --");
        console.log(messages);

        if (!this.engine) {
          this.engine = await CreateMLCEngine(GEMMA_2_9B_CONFIG.model_id, {
            initProgressCallback: console.log,
            appConfig: {
              model_list: [GEMMA_2_9B_CONFIG],
            },
          });
        }

        await this.engine.chat.completions.create({
          messages,
          temperature,
        });
        const response = await this.engine.getMessage();

        messages.push({ role: "assistant", content: response });
        return response;
      },
    };
  };
}

export default WebLLM;
```

### Teaching Points

**Architecture Pattern**:
- Lazy initialization: Engine created on first `generate()` call
- Singleton engine: Only one instance per page
- Closure pattern: Messages array captured in `createConversation()`

**Message Roles**:
- **system**: Instructions for the AI's behavior (e.g., "You are a helpful assistant")
- **user**: Human's input/questions
- **assistant**: AI's responses
- Entire conversation history maintained for context

**Temperature Parameter**:
- Controls randomness in generation
- 0 = deterministic, focused
- 1 = creative, varied
- 2 = very random (rarely used)

**Initialization Flow**:
1. First call to `generate()` triggers model download
2. `initProgressCallback` logs download progress
3. Model compiled for WebGPU
4. Engine persists across generations

**Why Lazy Init?**
- Don't download/compile until actually needed
- User might not use AI features
- Better perceived performance

### Code Walkthrough

Walk through step-by-step:

1. **Class structure**: Single responsibility - manage LLM conversations
2. **Private engine**: Shared across all conversations (memory efficient)
3. **createConversation**: Factory method pattern
   - Returns object with `generate` method
   - Captures `messages` array in closure
   - Each conversation maintains its own history
4. **generate method**:
   - Adds user message to history
   - Creates/reuses engine
   - Gets response
   - Adds assistant response to history
   - Returns text

### Testing

**Test in browser console**:
```javascript
const llm = new WebLLM();
const conv = llm.createConversation("You are a helpful assistant.");
await conv.generate("Tell me a joke");
// Wait for model download on first call (~5 min)
// Second call will be fast
await conv.generate("Tell me another one");
```

### Common Issues & Troubleshooting

❌ **Engine initialization hangs**:
- Check browser console for errors
- Verify WebGPU support
- First init downloads large model - be patient

❌ **Messages not maintaining context**:
- Check that messages array is being updated
- Verify messages.push() calls happen in correct order

❌ **Memory leaks with multiple conversations**:
- Engine is shared (good!)
- Each conversation has own messages array (intended)
- Don't create new conversations unnecessarily

### Discussion Questions
- Why use closure instead of passing messages as parameter?
- What happens if user has multiple chat sessions?
- How would you add streaming responses?

---

## Step 3: Implement Cloud-Based Gemini LLM

**Commit**: `25ff0b56` - "add gemini llm"  
**Time**: 20 minutes  
**Difficulty**: Medium

### Overview
Create an alternative LLM implementation using Google's Gemini API for cloud-based inference. This demonstrates abstraction patterns and provides a fallback when local inference isn't available.

### What Participants Will Learn
- REST API integration with LLMs
- Environment variable management
- Interface compatibility/duck typing
- Cloud vs. local trade-offs

### Implementation

**Create**: `src/ai/llm/GeminiLlm.ts`

```typescript
interface MessagePart {
  role: "user" | "system" | "model";
  content: string;
}

interface Part {
  text: string;
}

interface Content {
  parts: Part[];
  role: string;
}

interface Candidate {
  content: Content;
  finishReason: string;
  avgLogprobs: number;
}

interface UsageMetadata {
  promptTokenCount: number;
  candidatesTokenCount: number;
  totalTokenCount: number;
}

export interface GeminiResponse {
  candidates: Candidate[];
  usageMetadata: UsageMetadata;
  modelVersion: string;
}

const API_KEY = import.meta.env.VITE_GOOGLE_GENERATIVE_AI_API_KEY;

class GeminiLlm {
  private postApi = async (
    messages: Array<MessagePart>,
    temperature: number = 1
  ): Promise<string> => {
    const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${API_KEY}`;

    const requestBody = {
      contents: messages.map((m) => ({
        role: m.role === "user" || m.role === "system" ? "user" : "model",
        parts: [{ text: m.content }],
      })),
      generationConfig: { temperature },
    };

    const resp = await fetch(url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(requestBody),
    });
    const body: GeminiResponse = await resp.json();

    return body.candidates[0].content.parts[0].text;
  };

  public createConversation = (systemPrompt: string) => {
    if (!API_KEY) {
      throw new Error("env VITE_GOOGLE_GENERATIVE_AI_API_KEY is required");
    }
    const messages: Array<MessagePart> = [
      { role: "system", content: systemPrompt },
    ];

    return {
      generate: async (
        prompt: string,
        callback: (text: string) => void = () => {}
      ) => {
        messages.push({ role: "user", content: prompt });
        const generated = await this.postApi(messages);
        messages.push({ role: "model", content: generated });
        callback(generated);
        return generated;
      },
    };
  };
}

export default GeminiLlm;
```

### Teaching Points

**API Integration**:
- Uses Google's Generative AI REST API
- Gemini 1.5 Flash: Fast, cost-effective model
- Stateless API: Must send full conversation each time
- Rate limits apply (check Google AI Studio dashboard)

**Environment Variables in Vite**:
- Must prefix with `VITE_` to be accessible
- Available at `import.meta.env.VITE_*`
- Never commit API keys to git!
- Add `.env` to `.gitignore`

**Message Format Transformation**:
- Gemini uses different role names: "user" and "model" (not "assistant")
- System messages treated as user messages (Gemini doesn't have system role)
- Parts array structure: `{ parts: [{ text: "..." }] }`

**Duck Typing / Interface Compatibility**:
- Same public API as WebLLM: `createConversation()` returns object with `generate()`
- Can swap implementations without changing calling code
- This is the power of abstraction!

**Callback Pattern**:
- Optional callback parameter for streaming UI updates
- Could be enhanced for true streaming with Server-Sent Events
- Useful for showing incremental responses

### Comparison: WebLLM vs Gemini

| Feature | WebLLM | Gemini |
|---------|---------|---------|
| **Privacy** | ✅ Complete (local) | ❌ Data sent to Google |
| **Offline** | ✅ After initial download | ❌ Requires internet |
| **Speed** | ⚡ Fast (depends on GPU) | ⚡ Very fast (Google servers) |
| **Cost** | 💰 Free | 💰 Pay per token |
| **Setup** | ⚙️ Complex (large download) | ⚙️ Simple (API key) |
| **Reliability** | ❓ Depends on device | ✅ Very reliable |
| **Model Size** | 📦 Limited by device | 📦 Any model available |

### Testing

**Quick test** (requires API key in `.env`):
```javascript
const llm = new GeminiLlm();
const conv = llm.createConversation("You are a pirate. Speak like one!");
const response = await conv.generate("What's the weather today?");
console.log(response); // Should respond in pirate speak!
```

### Common Issues & Troubleshooting

❌ **API key error**:
- Verify `.env` file exists in project root
- Check variable name: `VITE_GOOGLE_GENERATIVE_AI_API_KEY`
- Restart dev server after adding `.env`
- Verify API key is valid at https://aistudio.google.com

❌ **CORS errors**:
- Gemini API allows browser requests
- Check browser console for details
- Verify URL is correct

❌ **Rate limiting**:
- Gemini free tier has limits
- Check quotas in Google AI Studio
- Add exponential backoff for production

❌ **Parsing errors**:
- Check response structure matches interface
- Handle cases where `candidates` is empty
- Add error handling for API failures

### Discussion Questions
- When would you choose WebLLM over Gemini? Vice versa?
- How would you implement automatic fallback (try WebLLM, fall back to Gemini)?
- What about other LLM providers (OpenAI, Anthropic, etc.)?
- How would you add cost tracking?

### Extension Ideas
- Add response streaming
- Implement retry logic
- Add token counting
- Create provider abstraction interface
- Add caching layer

---

## Step 4: Build AI Agent with RAG

**Commit**: `afa86c8b` - "agent added"  
**Time**: 30 minutes  
**Difficulty**: Hard

### Overview
This is the most complex step. Implement an AI agent with function calling capabilities and RAG using vector embeddings. Three files work together to enable intelligent FAQ search.

### What Participants Will Learn
- Agentic AI patterns (reasoning loop)
- Function calling / tool use
- Vector embeddings for semantic search
- RAG (Retrieval-Augmented Generation)
- WebGPU-based embeddings with Transformers.js

### Architecture Overview

```
Agent Flow:
1. User query → Agent
2. Agent adds tools to system prompt
3. LLM generates response (may include function calls)
4. Agent parses XML function calls
5. Agent executes tools
6. Tool results added to prompt
7. Loop back to step 3 (max rounds: 5)
8. Final answer returned
```

### Implementation - Part A: Feature Extraction

**Create**: `src/utils/vectorSearch/FeatureExtraction.ts`

```typescript
import { FeatureExtractionPipeline, pipeline } from "@huggingface/transformers";

class FeatureExtraction {
  private pipeline: FeatureExtractionPipeline | null = null;

  public extract = async (text: string): Promise<Array<number>> => {
    if (!this.pipeline) {
      this.pipeline = await pipeline(
        "feature-extraction",
        "Xenova/all-MiniLM-L6-v2",
        {
          progress_callback: console.log,
          device: "webgpu",
        }
      );
    }
    const output = await this.pipeline(text, {
      pooling: "mean",
      normalize: true,
    });

    return output.tolist()[0];
  };
}

export default FeatureExtraction;
```

**Teaching Points**:

**What are Embeddings?**
- Convert text to fixed-size number arrays (vectors)
- Similar text → similar vectors
- Enable semantic search (meaning-based, not keyword-based)
- Example dimensions: 384 for all-MiniLM-L6-v2

**Model: all-MiniLM-L6-v2**
- Sentence embedding model
- 384-dimensional vectors
- Trained on semantic similarity tasks
- Fast and accurate
- Runs in browser via WebGPU!

**Transformers.js**
- Hugging Face models compiled for JavaScript
- Uses ONNX Runtime Web
- WebGPU acceleration
- First load downloads model (~80MB)

**Pipeline Pattern**:
- Singleton pattern for model
- Lazy initialization
- Pooling: "mean" averages token embeddings
- Normalize: Unit length vectors (important for cosine similarity)

### Implementation - Part B: FAQ Search

**Create**: `src/ai/vectorSearch/findSimilarFAQs.ts`

```typescript
import { FAQ_WITH_ID } from "../../store/faq.ts";
import FeatureExtraction from "../../utils/vectorSearch/FeatureExtraction.ts";
import cosineSimilarity from "../../utils/vectorSearch/cosineSimilarity.ts";

const extractor = new FeatureExtraction();

const findSimilarFAQs = async (query: string, results: number = 4) => {
  const queryVector = await extractor.extract(query);
  const faqs = FAQ_WITH_ID.map(({ questions }) => questions).flat();

  const similarities = faqs.map((faq) => {
    const vector = faq.vectorRepresentationQuestion.allMiniLmL6v2;
    if (vector.length !== queryVector.length) {
      throw new Error("Vector lengths do not match");
    }
    return { faq, similarity: cosineSimilarity(queryVector, vector) };
  });

  return similarities
    .sort((a, b) => b.similarity - a.similarity)
    .slice(0, results)
    .map(({ faq }) => faq);
};

export default findSimilarFAQs;
```

**Teaching Points**:

**Vector Search Flow**:
1. Convert user query to vector (embedding)
2. Compare against pre-computed FAQ vectors
3. Calculate similarity scores (cosine similarity)
4. Sort by similarity (highest first)
5. Return top N results

**Cosine Similarity**:
- Measures angle between vectors
- Range: -1 (opposite) to 1 (identical)
- Normalized vectors: Only direction matters, not magnitude
- Formula: dot(A, B) / (||A|| * ||B||)
- Pre-computed in `cosineSimilarity.ts` utility

**Pre-computed Embeddings**:
- FAQs already have vectors in `faq.ts`
- Generated offline to save time
- Same model must be used for consistency
- Real apps: Store in vector database (Pinecone, Weaviate, etc.)

**Why This Works (RAG)**:
- LLMs don't "know" your specific FAQ data
- Retrieval gets relevant information
- Augmentation adds it to LLM context
- Generation produces informed answer
- RAG = Retrieval-Augmented Generation

### Implementation - Part C: Agent Class

**Create**: `src/ai/agent/Agent.ts`

```typescript
import React from "react";

import parseXmlFunctionCalls from "../../utils/agent/parseXmlFunctionCalls.ts";
import { Tool } from "../../utils/agent/tool.ts";
import toolsToSystemPrompt from "../../utils/agent/toolsToSystemPrompt.ts";
import WebLLM from "../llm/WebLLM.ts";

class Agent {
  private llm = new WebLLM();

  private tools: Record<string, Tool> = {};

  public addTool = (name: string, tool: Tool) => {
    this.tools[name] = tool;
  };

  public processPrompt = async (
    systemPrompt: string,
    userPrompt: string,
    maxRounds: number = 5,
    renderCallback?: (element: React.ReactElement) => void
  ) => {
    const conversation = this.llm.createConversation(
      `${systemPrompt}\n\n${toolsToSystemPrompt(this.tools)}`
    );
    let nextPrompt: string = userPrompt;
    let finalAnswer: string = "";
    let round = 0;

    while (nextPrompt && round < maxRounds) {
      const response = await conversation.generate(nextPrompt, 0);
      const parsed = parseXmlFunctionCalls(response);

      const toolsToCall = parsed.functionCalls
        .filter((func) => func.name in this.tools)
        .map((func) => this.tools[func.name].execute(func.parameters));

      if (toolsToCall.length === 0) {
        nextPrompt = "";
        finalAnswer = parsed.cleanText;
        continue;
      }

      const functionResults = await Promise.all(toolsToCall);
      nextPrompt = functionResults
        .map((result) => {
          if (result.render && renderCallback) {
            renderCallback(result.render());
          }
          return result.nextPrompt;
        })
        .filter(Boolean)
        .join("\n\n");
    }

    return (
      finalAnswer ||
      "Tell the user that you have not been able to answer the question"
    );
  };
}

export default Agent;
```

**Teaching Points**:

**Agent Loop Pattern**:
- Think → Act → Observe → Repeat
- Each round: LLM sees results from previous actions
- Max rounds prevents infinite loops
- Temperature=0 for deterministic tool use

**Tool Registration**:
- Dynamic tool registration via `addTool()`
- Tools defined elsewhere, agent agnostic
- Flexible: Add/remove tools without changing agent

**Function Call Parsing**:
- LLM outputs XML-formatted function calls
- `parseXmlFunctionCalls()` extracts: function name, parameters
- Returns both cleaned text and function calls
- XML chosen for reliability (easy to parse, works with all models)

**Tool Execution**:
- Filter to valid tools only
- Execute in parallel (`Promise.all`)
- Each tool returns:
  - `nextPrompt`: Context for next LLM call
  - `render`: Optional React component for UI

**Loop Termination**:
- No function calls → final answer ready
- Max rounds reached → return best effort
- `nextPrompt` empty → done

**System Prompt Enhancement**:
- Original system prompt + tool descriptions
- `toolsToSystemPrompt()` generates formatted instructions
- LLM learns what tools available and when to use them

### Testing the Components

**Test embedding extraction**:
```javascript
const extractor = new FeatureExtraction();
const vec1 = await extractor.extract("How do I return a product?");
const vec2 = await extractor.extract("What's the return policy?");
const vec3 = await extractor.extract("I love pizza!");

console.log(vec1.length); // 384
console.log(cosineSimilarity(vec1, vec2)); // High (~0.8)
console.log(cosineSimilarity(vec1, vec3)); // Low (~0.2)
```

**Test FAQ search**:
```javascript
const results = await findSimilarFAQs("How do I return items?");
console.log(results); // Should show return policy FAQs
```

**Test agent** (next step will integrate into UI):
```javascript
const agent = new Agent();
agent.addTool("time", tool({
  description: "Get current time",
  parameters: z.object({}),
  execute: async () => ({
    nextPrompt: `Current time: ${new Date().toISOString()}`
  }),
  examples: [{ query: "what time is it?", parameters: {} }]
}));

const answer = await agent.processPrompt(
  "You are a helpful assistant.",
  "What time is it?"
);
console.log(answer);
```

### Common Issues & Troubleshooting

❌ **Embedding model fails to load**:
- Check WebGPU support
- First load downloads ~80MB
- Check network tab for progress
- Clear cache if corrupted

❌ **Vector dimension mismatch**:
- Ensure same model for query and pre-computed embeddings
- Check FAQ data has correct vectors
- Verify model outputs 384 dimensions

❌ **Agent infinite loop**:
- Check `maxRounds` setting
- Verify tool results include proper `nextPrompt`
- Debug by logging each round

❌ **Function calls not parsed**:
- Check LLM actually outputs XML format
- Verify XML structure matches expected format
- Log raw LLM response to debug

❌ **Poor semantic search results**:
- Embedding model may not suit domain
- Try different models or fine-tune
- Check FAQ coverage

### Discussion Questions
- Why XML for function calls instead of JSON?
- How would you add streaming to the agent loop?
- What happens if a tool execution fails?
- How would you implement parallel tool execution with dependencies?
- How does this compare to LangChain or other frameworks?

### Extension Ideas
- Add tool execution timeouts
- Implement tool call validation
- Add conversation memory across sessions
- Support nested/chained tool calls
- Add tool usage analytics

---

## Step 5: Fix System Prompt Handling

**Commit**: `84fc504e` - "fix system prompt"  
**Time**: 5 minutes  
**Difficulty**: Easy

### Overview
Minor fix to properly handle system prompts in WebLLM.

### Implementation

**Modify**: `src/ai/llm/WebLLM.ts`

Change line 9 from:
```typescript
{ role: "system", content:  },systemPrompt
```

To:
```typescript
{ role: "system", content: systemPrompt },
```

### Teaching Points
- Syntax error in original (formatting issue)
- System prompts are critical for agent behavior
- Always test with actual LLM to catch these issues

---

## Step 6: Integrate Agent into Chat UI

**Commit**: `e3079587` - "agent added to chat"  
**Time**: 30 minutes  
**Difficulty**: Medium-Hard

### Overview
The final step! Integrate the agent into the Chat UI component with two tools: FAQ search and page navigation. This brings everything together.

### What Participants Will Learn
- React integration with AI agents
- Tool implementation with Zod schemas
- UI rendering from tool results
- Navigation integration
- Loading states and UX

### Implementation

**Modify**: `src/app/chat/Chat.tsx`

Key additions:

1. **Import Dependencies**:
```typescript
import { z } from "zod";
import Agent from "../../ai/agent/Agent.ts";
import findSimilarFAQs from "../../ai/vectorSearch/findSimilarFAQs.ts";
import tool from "../../utils/agent/tool.ts";
```

2. **Create Agent with Tools**:
```typescript
const agent = React.useMemo(() => {
  const agent = new Agent();
  
  // Tool 1: FAQ Search
  agent.addTool("faqSearch", tool({
    description: "Search for similar FAQs to answer user questions about policies, shipping, returns, etc.",
    parameters: z.object({
      query: z.string().describe("The user's question to search FAQs for"),
    }),
    execute: async (args: { query: string }) => {
      const faqs = await findSimilarFAQs(args.query, 3);
      return {
        nextPrompt: `Found ${faqs.length} relevant FAQs:\n${faqs.map((faq, i) => 
          `${i + 1}. Q: ${faq.question}\n   A: ${faq.answer}`
        ).join('\n\n')}`,
        render: () => (
          <div className="space-y-2">
            <h4 className="font-semibold">Related FAQs:</h4>
            {faqs.map((faq, i) => (
              <div key={i} className="border-l-2 border-purple-400 pl-3">
                <p className="font-medium">{faq.question}</p>
                <p className="text-sm text-gray-600">{faq.answer}</p>
              </div>
            ))}
          </div>
        ),
      };
    },
    examples: [
      { query: "How do I return a product?", parameters: { query: "return policy" } },
      { query: "What's your shipping policy?", parameters: { query: "shipping" } },
    ],
  }));

  // Tool 2: Navigate to Page
  agent.addTool("navigateToPage", tool({
    description: "Navigate to a page in the app with specific filters",
    parameters: z.object({
      category: z.enum(["accessories", "clothes"]).optional(),
      color: z.enum(["red", "blue", "green", "yellow"]).optional(),
      size: z.enum(["S", "M", "L", "XL"]).optional(),
    }),
    execute: async (args) => {
      const params = new URLSearchParams();
      if (args.category) params.set("category", args.category);
      if (args.color) params.set("color", args.color);
      if (args.size) params.set("size", args.size);
      
      const path = `/products?${params.toString()}`;
      navigate(path);
      
      return {
        nextPrompt: `Navigated to ${path}`,
        render: () => (
          <div className="flex items-center gap-2 text-purple-700">
            <LinkIcon className="size-4" />
            <span>Navigated to products page</span>
          </div>
        ),
      };
    },
    examples: [
      { 
        query: "Show me red shirts", 
        parameters: { category: "clothes", color: "red" } 
      },
      { 
        query: "I want large blue accessories", 
        parameters: { category: "accessories", color: "blue", size: "L" } 
      },
    ],
  }));

  return agent;
}, [navigate]);
```

3. **Update Form Submit Handler**:
```typescript
<ChatForm
  chatOpen={chatOpen}
  onSubmit={async (prompt) => {
    if (!prompt) {
      setResponse("");
      setToolElements([]);
      return;
    }
    
    setThinking(true);
    setToolElements([]);
    
    try {
      const systemPrompt = `You are a helpful shopping assistant for an e-commerce store.
Help users find products, answer questions about policies, and navigate the site.
Use the available tools to provide accurate information.`;
      
      const answer = await agent.processPrompt(
        systemPrompt,
        prompt,
        5,
        (element) => {
          setToolElements((prev) => [...prev, element]);
        }
      );
      
      setResponse(answer);
    } catch (error) {
      console.error("Agent error:", error);
      setResponse("Sorry, I encountered an error. Please try again.");
    } finally {
      setThinking(false);
    }
  }}
/>
```

4. **Render Tool Results**:
```typescript
{toolElements.length > 0 && (
  <div className="space-y-2 mb-4">
    {toolElements.map((element, i) => (
      <div key={i} className="bg-purple-100 p-3 rounded">
        {element}
      </div>
    ))}
  </div>
)}
```

### Teaching Points

**Tool Design Principles**:
1. **Clear descriptions**: LLM needs to understand when to use tool
2. **Zod schemas**: Type-safe parameters with descriptions
3. **Examples**: Help LLM understand usage patterns
4. **Dual outputs**: `nextPrompt` for agent loop, `render` for UI

**FAQ Search Tool**:
- Semantic search using embeddings
- Returns context to LLM
- Renders FAQs in UI
- LLM synthesizes answer from retrieved FAQs

**Navigate Tool**:
- Uses React Router's `navigate()`
- Builds URL with query parameters
- Provides visual feedback
- Example of "action" tool (vs "information" tool like FAQ search)

**Agent Integration Pattern**:
- `useMemo` to avoid recreating agent
- Separate state for thinking, response, tool renders
- Error handling
- Loading states

**UX Considerations**:
- Show loading state ("thinking..")
- Render tool results as they execute
- Clear previous state on new query
- Error messages for failures

### Demo Flow

Walk through complete user interaction:

1. **User asks**: "What's your return policy?"
2. **Agent thinks**: Decides to use `faqSearch` tool
3. **Tool executes**: Finds relevant FAQ entries
4. **UI updates**: Renders FAQ cards
5. **Agent synthesizes**: Generates natural language answer from FAQs
6. **User sees**: Both FAQ cards and conversational response

Then demonstrate navigation:
1. **User asks**: "Show me red clothes"
2. **Agent thinks**: Decides to use `navigateToPage` tool
3. **Tool executes**: Navigates to `/products?category=clothes&color=red`
4. **UI updates**: Products page with filters applied
5. **Agent confirms**: "Here are the red clothing items"

### Testing Scenarios

Test these cases with participants:

**FAQ Questions**:
- "How do I return a product?"
- "What's your shipping policy?"
- "Do you ship internationally?"

**Navigation Requests**:
- "Show me red shirts"
- "I want large blue accessories"
- "Do you have yellow clothes?"

**Combined Queries**:
- "What's your return policy? And show me some hoodies"
- "Do you have green items in size M?"

**Edge Cases**:
- Empty query
- Nonsense query
- Query requiring multiple tools
- Query outside domain

### Common Issues & Troubleshooting

❌ **Agent doesn't use tools**:
- Check system prompt includes tool descriptions
- Verify tool examples are clear
- Try more explicit user queries
- Check console for LLM response

❌ **Tools not executing**:
- Verify tool names match exactly
- Check Zod schema validation
- Debug with logs in `execute` function

❌ **Navigation doesn't work**:
- Check `navigate` from useNavigate is in dependency array
- Verify route exists in router config
- Check query parameter format

❌ **UI not updating**:
- Check `renderCallback` is passed to `processPrompt`
- Verify state updates in callback
- Check React key props on rendered elements

❌ **Model too slow**:
- Switch to Gemini for faster responses
- Use smaller WebLLM model (2B vs 9B)
- Reduce max rounds

### Discussion Questions
- How would you add more tools (e.g., add to cart)?
- What about conversation history persistence?
- How to handle tool execution failures gracefully?
- How would you add tool chaining (one tool's output feeds another)?
- What's the trade-off between number of tools and agent reliability?

### Extension Ideas
- Add cart management tools
- Implement multi-step tool workflows
- Add conversation export
- Create tool usage analytics dashboard
- Add voice input/output
- Implement tool caching for repeated queries
- Add A/B testing for different system prompts

---

## Workshop Wrap-Up (15 minutes)

### What We Built

✅ **Local LLM Integration**: WebLLM with WebGPU  
✅ **Cloud LLM Integration**: Google Gemini API  
✅ **AI Agent**: Function calling with reasoning loop  
✅ **RAG System**: Vector embeddings + semantic search  
✅ **React Integration**: Full UI integration with tools  

### Key Takeaways

1. **LLMs are tools**: Choose local vs cloud based on requirements
2. **Agents extend LLMs**: Function calling enables action
3. **RAG solves knowledge gaps**: Retrieval + generation = grounded answers
4. **Abstractions matter**: Good interfaces enable flexibility
5. **UX is critical**: Loading states, error handling, visual feedback

### Production Considerations

**Performance**:
- WebLLM requires WebGPU-capable device
- First load is slow (model download)
- Consider hybrid: WebLLM for repeat users, Gemini for first-time
- Implement streaming for better perceived performance

**Reliability**:
- Tool execution can fail - add retries
- LLM might not follow instructions - validate outputs
- Rate limiting with cloud providers
- Fallback strategies

**Security**:
- Never expose API keys in client code (use backend proxy)
- Validate and sanitize tool inputs
- Limit tool capabilities (principle of least privilege)
- Monitor usage for abuse

**Cost**:
- Cloud LLMs charge per token
- Track usage and set budgets
- Cache common queries
- Use smaller models where possible

**Privacy**:
- Local LLMs = no data leaves device
- Cloud LLMs = data sent to provider
- Be transparent with users
- Consider data retention policies

### Next Steps for Participants

**Beginner**:
- Add more tools (weather, search, etc.)
- Customize system prompts
- Improve UI/UX
- Add conversation export

**Intermediate**:
- Implement tool caching
- Add streaming responses
- Create tool composition (chaining)
- Add analytics dashboard

**Advanced**:
- Build custom embeddings model for domain
- Implement multi-agent systems
- Add reinforcement learning from feedback
- Create evaluation framework

### Resources

**Documentation**:
- [WebLLM Docs](https://mlc.ai/web-llm/)
- [Transformers.js](https://huggingface.co/docs/transformers.js)
- [Google AI SDK](https://ai.google.dev/docs)
- [Vercel AI SDK](https://sdk.vercel.ai/docs)

**Further Learning**:
- [LangChain](https://js.langchain.com/) - Agent framework
- [LlamaIndex](https://www.llamaindex.ai/) - RAG framework
- [Pinecone](https://www.pinecone.io/) - Vector database
- [Weights & Biases](https://wandb.ai/) - Experiment tracking

**Community**:
- [Hugging Face Discord](https://huggingface.co/join/discord)
- [LocalAI Community](https://github.com/mudler/LocalAI)
- [AI Stack Devs](https://www.latent.space/)

---

## Appendix: Troubleshooting Guide

### WebGPU Issues

**Browser Support**:
```bash
# Check WebGPU availability
navigator.gpu !== undefined
```

**Enable in Chrome**:
1. Go to `chrome://flags`
2. Search "WebGPU"
3. Enable "Unsafe WebGPU"
4. Restart browser

**GPU Check**:
```javascript
const adapter = await navigator.gpu?.requestAdapter();
const info = await adapter?.requestAdapterInfo();
console.log(info);
```

### Model Loading Issues

**Clear Cache**:
```javascript
// In console
await caches.keys().then(names => 
  Promise.all(names.map(name => caches.delete(name)))
);
```

**Check Download Progress**:
```javascript
// Monitor Network tab in DevTools
// Look for large binary files (.wasm, .bin)
```

**Memory Issues**:
```javascript
// Check memory usage
performance.memory.usedJSHeapSize / 1024 / 1024 // MB
```

### API Issues

**Test API Key**:
```bash
curl "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=YOUR_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"contents":[{"parts":[{"text":"Hello"}]}]}'
```

**Rate Limit Handling**:
```typescript
async function withRetry(fn: () => Promise<any>, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
}
```

### Debugging Tips

**Log Everything**:
```typescript
// Add to Agent class
console.log("Round:", round);
console.log("Prompt:", nextPrompt);
console.log("Response:", response);
console.log("Parsed:", parsed);
console.log("Tools to call:", toolsToCall);
```

**Visualize Embeddings**:
```typescript
// Use t-SNE or PCA to visualize vector space
import { PCA } from 'ml-pca';

const vectors = [...]; // Your embeddings
const pca = new PCA(vectors);
const reduced = pca.predict(vectors, { nComponents: 2 });
// Plot reduced[i] for each embedding
```

**Test Tools Independently**:
```typescript
// Test each tool outside agent
const faqTool = tools["faqSearch"];
const result = await faqTool.execute({ query: "return policy" });
console.log(result);
```

---

## Time Management Cheat Sheet

| Step | Topic | Duration | Can Skip If Behind? |
|------|-------|----------|-------------------|
| 0 | Setup & Intro | 10 min | No |
| 1 | WebLLM Config | 10 min | Yes (provide code) |
| 2 | WebLLM Class | 15 min | Partial |
| 3 | Gemini LLM | 20 min | Yes (focus on WebLLM) |
| 4 | Agent + RAG | 30 min | No (core concept) |
| 5 | System Prompt Fix | 5 min | Yes (typo fix) |
| 6 | Chat Integration | 30 min | No (brings it together) |
| 7 | Wrap-up | 15 min | Shorten to 5 min |

**Total**: ~135 minutes (2h 15min)  
**With breaks**: ~150 minutes (2h 30min)

**If Running Short**: Skip Step 3 (Gemini), provide code for Steps 1 & 5.

---

## FAQ for Workshop Hosts

**Q: What if participants' GPUs don't support WebGPU?**  
A: Fall back to Gemini (cloud-based). Have backup API keys available, or use demo.

**Q: Should I have participants type all code or provide templates?**  
A: Hybrid approach: Type key sections (agent loop, tool definitions), provide utility functions.

**Q: What if the model download is too slow during workshop?**  
A: Pre-cache models on WiFi before workshop, or use Gemini exclusively.

**Q: How technical should I get with embeddings explanation?**  
A: Keep it practical. Show it works first, explain math only if asked.

**Q: What if someone asks about production deployment?**  
A: Great question! Discuss backend proxy for API keys, CDN for models, monitoring, rate limiting.

**Q: How do I handle different experience levels?**  
A: Pair programming! Match beginners with advanced. Have extension challenges ready.

**Q: Should I live code or use pre-built commits?**  
A: Live code with commits as checkpoints. Let participants checkout commits if they fall behind.

**Q: What's the most important concept to emphasize?**  
A: The agent loop (think → act → observe). It's the core of agentic AI.

---

## Post-Workshop Survey Questions

1. What was the most interesting concept you learned?
2. What was the most challenging part?
3. Rate your understanding of:
   - LLM integration (1-5)
   - AI agents (1-5)
   - RAG systems (1-5)
   - Vector embeddings (1-5)
4. What would you build with these skills?
5. What should we add/change for next workshop?
6. Would you recommend this workshop? (1-10)

---

## License

This workshop material is provided under the MIT License. Feel free to adapt, modify, and reuse for your own workshops!

---

**Version**: 1.0  
**Last Updated**: 2026-02-04  
**Author**: React Alicante Workshop Team  
**Contact**: [Add contact info]

---

## Acknowledgments

- **WebLLM Team**: For making browser-based LLMs possible
- **Hugging Face**: For Transformers.js and model hosting
- **Google**: For Gemini API
- **React Community**: For awesome libraries and tools
- **Workshop Participants**: For feedback and enthusiasm!

**Happy Teaching! 🎓✨**
