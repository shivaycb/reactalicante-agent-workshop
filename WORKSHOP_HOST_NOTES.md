# React Alicante Workshop: Building AI Agents - Host Notes

## Workshop Overview
This hands-on workshop teaches participants how to build an AI-powered shopping assistant for a React e-commerce application. Participants will learn to integrate AI capabilities using both local (WebLLM) and cloud-based (Google Gemini) models, implement vector search for semantic matching, and create an agentic system with tool calling.

**Duration:** ~4 hours (4 hours 20 minutes with breaks)  
**Level:** Intermediate (React knowledge required)  
**Target Audience:** React developers interested in adding AI capabilities to their applications

---

## Prerequisites

### Required Knowledge
- React fundamentals (hooks, state management, components)
- TypeScript basics
- Understanding of async/await
- Basic familiarity with REST APIs

### Setup Requirements
- Node.js 18+ installed
- Modern browser with WebGPU support (Chrome/Edge recommended)
- Google Generative AI API Key (get from https://aistudio.google.com/apikey)
- Code editor (VS Code recommended)
- At least 4GB free disk space (for WebLLM models)

### Pre-Workshop Setup (Send to participants 1 day before)
```bash
# Clone the repository
git clone https://github.com/shivaycb/reactalicante-agent-workshop
cd reactalicante-agent-workshop

# Checkout the main branch (starting point)
git checkout main

# Install dependencies (this may take a few minutes)
npm install

# Create .env file
echo "VITE_GOOGLE_GENERATIVE_AI_API_KEY=your_api_key_here" > .env
echo "VITE_USE_WEBLLM=false" >> .env

# Start the dev server to verify setup
npm run dev
```

---

## Workshop Structure

### Introduction (15 minutes)
**Topics to Cover:**
- Overview of AI capabilities in modern web applications
- Local vs. Cloud AI models (tradeoffs)
- Introduction to the demo application (webshop)
- Workshop goals and what we'll build

**Demo:**
1. Show the complete application (`git checkout complete` and run `npm run dev`)
2. Demonstrate the AI chat assistant features:
   - Natural language product search
   - FAQ answering with semantic search
   - Product navigation via chat
3. Switch back to main branch for the workshop

---

## Step 1: Project Setup and Introduction (20 minutes)
**Commit:** `0f6414e` - "Initialize Workshop"

### Learning Objectives
- Understand the project structure
- Familiarize with the React webshop application
- Explore the base application without AI features

### What Changed in This Step
This commit sets up the initial React application with:
- React + TypeScript + Vite setup
- TailwindCSS for styling
- React Router for navigation
- Product catalog with images
- Basic chat UI (non-functional)
- FAQ data structure
- Store context providers

### Key Files to Review
```
src/
├── App.tsx              # Main application component
├── app/                 # Layout components
│   ├── chat/           # Chat UI (empty implementation)
│   ├── header/         # Navigation header
│   └── products/       # Product listing pages
├── store/              # Application state
│   ├── products.ts     # Product catalog
│   └── faq.ts          # FAQ data with vector embeddings
├── theme/              # Reusable UI components
└── utils/              # Utility functions (agent utils included but not used yet)
```

### Host Instructions

1. **Walk through the application structure** (5 min)
   - Open `src/App.tsx` and explain the routing setup
   - Show `src/store/products.ts` - emphasize the enum-based categories, colors, sizes
   - Open `src/store/faq.ts` - point out the pre-computed vector embeddings

2. **Run the application** (5 min)
   ```bash
   npm run dev
   ```
   - Navigate through the product pages
   - Try to use the chat (show it doesn't work yet)
   - View products by different categories

3. **Explain the AI architecture we'll build** (10 min)
   - Draw a diagram on whiteboard:
     ```
     User Question → Agent → LLM (Gemini/WebLLM) → Tool Execution → Response
                      ↓
                  Vector Search (FAQ matching)
                      ↓
                  Tool Definitions (Product search, Navigation)
     ```

### Talking Points
- "We're starting with a fully functional e-commerce site, but the chat is just UI"
- "Notice the FAQ data already has vector embeddings - we pre-computed these to save time"
- "The utils folder has agent infrastructure we'll use later"

---

## Step 2: WebLLM Configuration (15 minutes)
**Commit:** `24a662aa` - "webllm config"

### Learning Objectives
- Understand WebLLM and local AI inference
- Configure a custom model for WebLLM
- Learn about model quantization and optimization

### What Changed in This Step
**New File:** `src/utils/agent/webllm.ts`

This commit adds configuration for WebLLM to use a custom-hosted Gemma 2 9B model:

```typescript
export const GEMMA_2_9B_CONFIG = {
  model: "https://uploads.nico.dev/mlc-llm-libs/gemma-2-9b-it_q4f16_MLC/",
  model_id: "gemma-2-9b-it_q4f16_MLC",
  model_lib: "https://uploads.nico.dev/mlc-llm-libs/gemma-2-9b-it_q4f16_MLC/lib/gemma-2-9b-it-q4f16_1-webgpu.wasm",
};

export const GEMMA_2_2B_ID = "gemma-2-2b-it-q4f16_1-MLC";
```

### Host Instructions

1. **Explain WebLLM** (5 min)
   - Runs entirely in the browser using WebGPU
   - No API calls, no server needed
   - Privacy-friendly - data never leaves the device
   - Tradeoff: Requires powerful GPU, slower first load

2. **Discuss Model Quantization** (5 min)
   - Original Gemma 2 9B: ~18GB
   - Quantized (q4f16): ~5GB - can fit in browser memory
   - q4f16 = 4-bit quantization with float16 computation
   - Show the model file structure in the URL

3. **Live Coding** (5 min)
   - Create `src/utils/agent/webllm.ts`
   - Copy the configuration
   - Explain each field:
     - `model`: Base URL for model files
     - `model_id`: Unique identifier
     - `model_lib`: WASM library for WebGPU execution

### Talking Points
- "WebLLM uses MLC (Machine Learning Compilation) to run LLMs efficiently in browsers"
- "The model downloads once and caches in IndexedDB - ~5GB download"
- "Perfect for privacy-sensitive applications or offline-capable apps"
- "We're using a custom-hosted model for reliability in the workshop"

### Common Questions
- **Q: Why not use the default WebLLM models?**  
  A: Custom hosting ensures availability and specific version control for the workshop
  
- **Q: Can I use this in production?**  
  A: Yes! But consider the large download size and GPU requirements

---

## Step 3: WebLLM Implementation (30 minutes)
**Commit:** `1e3f8a02` - "webllm"

### Learning Objectives
- Implement a WebLLM wrapper class
- Understand conversation management
- Learn about the MLC Engine API

### What Changed in This Step
**New File:** `src/ai/llm/WebLLM.ts`

This commit implements the WebLLM wrapper that:
- Creates and manages an MLCEngine instance
- Implements a conversation pattern with system prompts
- Handles message history
- Provides a clean API for generating responses

### Key Code Structure

```typescript
class WebLLM {
  private engine: MLCEngine | null = null;

  public createConversation = (systemPrompt: string) => {
    const messages: Array<ChatCompletionMessageParam> = [
      { role: "system", content: systemPrompt },
    ];

    return {
      generate: async (prompt: string, temperature: number = 1) => {
        messages.push({ role: "user", content: prompt });
        
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
```

### Host Instructions

1. **Explain the Architecture** (5 min)
   - Singleton pattern for the engine (expensive to create)
   - Conversation pattern keeps message history
   - Lazy initialization of the engine

2. **Live Coding** (15 min)
   - Create `src/ai/llm/WebLLM.ts`
   - Start with the class skeleton
   - Implement `createConversation` method:
     ```typescript
     public createConversation = (systemPrompt: string) => {
       const messages = [{ role: "system", content: systemPrompt }];
       return { generate: async (prompt: string) => { /* ... */ } };
     }
     ```
   - Explain the conversation pattern: system → user → assistant → user → assistant
   - Add engine initialization with config
   - Implement chat completion and message history

3. **Test the Implementation** (10 min)
   - Open browser console
   - Create a test instance:
     ```javascript
     const llm = new WebLLM();
     const chat = llm.createConversation("You are a helpful shopping assistant.");
     const response = await chat.generate("Hello!");
     console.log(response);
     ```
   - Watch the model download progress (will take a few minutes first time)
   - Show the response

### Talking Points
- "The engine initialization is slow (~1-2 min first time), so we lazy-load it"
- "The conversation keeps full history - this gives the model context"
- "Temperature=0 means deterministic (same input → same output), higher values add creativity"
- "System prompts set the AI's behavior and personality"

### Common Issues
- **Model download fails:** Check browser WebGPU support, network connection
- **Out of memory:** Try the 2B model instead of 9B
- **Slow responses:** Normal for large models in browser, ~2-5 seconds per response

---

## Step 4: Google Gemini LLM Integration (30 minutes)
**Commit:** `25ff0b56` - "add gemini llm"

### Learning Objectives
- Integrate cloud-based AI (Google Gemini)
- Understand API-based LLM interaction
- Compare local vs. cloud approaches

### What Changed in This Step
**New File:** `src/ai/llm/GeminiLlm.ts`

This commit adds a Gemini LLM wrapper with the same interface as WebLLM, enabling easy switching between local and cloud models.

### Key Code Structure

```typescript
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
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(requestBody),
    });
    const body: GeminiResponse = await resp.json();
    return body.candidates[0].content.parts[0].text;
  };

  public createConversation = (systemPrompt: string) => {
    if (!API_KEY) {
      throw new Error("env VITE_GOOGLE_GENERATIVE_AI_API_KEY is required");
    }
    const messages = [{ role: "system", content: systemPrompt }];

    return {
      generate: async (prompt: string, callback: (text: string) => void = () => {}) => {
        messages.push({ role: "user", content: prompt });
        const generated = await this.postApi(messages);
        messages.push({ role: "model", content: generated });
        callback(generated);
        return generated;
      },
    };
  };
}
```

### Host Instructions

1. **Compare WebLLM vs Gemini** (5 min)
   Create a comparison table:
   
   | Aspect | WebLLM | Gemini |
   |--------|--------|--------|
   | Speed | 2-5s per response | <1s per response |
   | Privacy | Complete privacy | Data sent to Google |
   | Cost | Free | Free tier, then paid |
   | Setup | 5GB download | API key only |
   | Offline | Yes | No |
   | Quality | Good (9B model) | Excellent (large model) |

2. **Live Coding** (15 min)
   - Create `src/ai/llm/GeminiLlm.ts`
   - Implement the interface types for Gemini API response
   - Build the `postApi` method:
     - Show the API URL structure
     - Explain message format conversion (Gemini uses "user"/"model" roles)
     - Handle the response parsing
   - Implement `createConversation` with the same interface as WebLLM
   - Add API key validation

3. **Test Both Implementations** (10 min)
   - Test Gemini:
     ```javascript
     const gemini = new GeminiLlm();
     const chat = gemini.createConversation("You are a helpful shopping assistant.");
     const response = await chat.generate("What products do you have?");
     console.log(response);
     ```
   - Compare response times
   - Compare response quality

### Talking Points
- "Same interface pattern makes models interchangeable - principle of abstraction"
- "Gemini 1.5 Flash is optimized for speed and cost-efficiency"
- "The API key should never be committed to git - always use .env files"
- "In production, API keys should be on the backend, not exposed to clients"

### Common Questions
- **Q: Why expose the API key in the frontend?**  
  A: For workshop simplicity. In production, proxy through your backend.
  
- **Q: Which should I use in production?**  
  A: Depends on requirements. WebLLM for privacy/offline, Gemini for quality/speed.

### Best Practices Discussion
- Environment variables for API keys
- Backend proxy for production
- Rate limiting and cost management
- Fallback strategies when API is unavailable

---

## Step 5: Building the AI Agent (45 minutes)
**Commit:** `afa86c8b` - "agent added"

### Learning Objectives
- Understand agentic AI patterns
- Implement tool calling and function execution
- Build vector search for semantic FAQ matching
- Learn about feature extraction and embeddings

### What Changed in This Step
**New Files:**
- `src/ai/agent/Agent.ts` - Core agent implementation with tool calling
- `src/ai/vectorSearch/findSimilarFAQs.ts` - Semantic search implementation
- `src/utils/vectorSearch/FeatureExtraction.ts` - Embedding generation using Hugging Face transformers

This is the most complex step, introducing the agentic pattern where the LLM can call functions.

### Key Concepts

#### 1. Agent Architecture
```
User Query → Agent.processPrompt()
                ↓
           LLM generates response with XML function calls
                ↓
           parseXmlFunctionCalls() extracts function calls
                ↓
           Execute tool functions
                ↓
           Collect results
                ↓
           Pass results back to LLM as next prompt
                ↓
           Repeat until no more tool calls (max 5 rounds)
```

#### 2. Tool System
Tools are defined with:
- `description`: What the tool does
- `parameters`: Zod schema for type-safe parameters
- `execute`: Async function that performs the action
- `examples`: Few-shot examples for the LLM

#### 3. Vector Search
- Uses Hugging Face transformers.js in the browser
- Model: `Xenova/all-MiniLM-L6-v2` (small, fast, 384-dimensional embeddings)
- Converts text to vectors using WebGPU
- Finds similar FAQs using cosine similarity

### Host Instructions

#### Part A: Feature Extraction (15 min)

1. **Explain Embeddings** (5 min)
   - Text → Vector of numbers that captures semantic meaning
   - Similar meanings → Similar vectors
   - Enables semantic search (vs. keyword search)
   - Show example:
     ```
     "red shirt" → [0.12, -0.34, 0.56, ...]
     "crimson top" → [0.15, -0.31, 0.53, ...]  // Very similar!
     "blue pants" → [-0.42, 0.67, -0.23, ...]  // Very different
     ```

2. **Live Coding** (10 min)
   - Create `src/utils/vectorSearch/FeatureExtraction.ts`
   - Implement the pipeline initialization:
     ```typescript
     import { pipeline } from "@huggingface/transformers";
     
     class FeatureExtraction {
       private pipeline = null;
       
       public extract = async (text: string) => {
         if (!this.pipeline) {
           this.pipeline = await pipeline(
             "feature-extraction",
             "Xenova/all-MiniLM-L6-v2",
             { progress_callback: console.log, device: "webgpu" }
           );
         }
         const output = await this.pipeline(text, {
           pooling: "mean",
           normalize: true,
         });
         return output.tolist()[0];
       };
     }
     ```
   - Test it:
     ```javascript
     const extractor = new FeatureExtraction();
     const vector = await extractor.extract("red shirt");
     console.log(vector); // Array of 384 numbers
     ```

#### Part B: Semantic FAQ Search (10 min)

1. **Explain the Algorithm** (3 min)
   ```
   1. Convert user query to vector
   2. Calculate cosine similarity with all FAQ vectors
   3. Sort by similarity
   4. Return top N matches
   ```

2. **Live Coding** (7 min)
   - Create `src/ai/vectorSearch/findSimilarFAQs.ts`
   - Show the FAQ data structure (already has vectors)
   - Implement cosine similarity search:
     ```typescript
     const findSimilarFAQs = async (query: string, results: number = 4) => {
       const queryVector = await extractor.extract(query);
       const faqs = FAQ_WITH_ID.map(({ questions }) => questions).flat();

       const similarities = faqs.map((faq) => {
         const vector = faq.vectorRepresentationQuestion.allMiniLmL6v2;
         return { faq, similarity: cosineSimilarity(queryVector, vector) };
       });

       return similarities
         .sort((a, b) => b.similarity - a.similarity)
         .slice(0, results)
         .map(({ faq }) => faq);
     };
     ```
   - Test with queries like "shipping costs", "return policy"

#### Part C: Agent Implementation (20 min)

1. **Explain Agentic Pattern** (5 min)
   - Draw the loop diagram:
     ```
     User Prompt → LLM → Parse Response
                          ↓
                     Has function calls?
                    /              \
                  YES               NO
                   ↓                ↓
              Execute Tools    Return Answer
                   ↓
              Results → LLM (next round)
     ```

2. **Live Coding** (15 min)
   - Create `src/ai/agent/Agent.ts`
   - Start with the class structure:
     ```typescript
     class Agent {
       private llm = new WebLLM();
       private tools: Record<string, Tool> = {};

       public addTool = (name: string, tool: Tool) => {
         this.tools[name] = tool;
       };
     }
     ```
   
   - Implement `processPrompt` method:
     ```typescript
     public processPrompt = async (
       systemPrompt: string,
       userPrompt: string,
       maxRounds: number = 5,
       renderCallback?: (element: React.ReactElement) => void
     ) => {
       // 1. Create conversation with system prompt + tool descriptions
       const conversation = this.llm.createConversation(
         `${systemPrompt}\n\n${toolsToSystemPrompt(this.tools)}`
       );
       
       let nextPrompt = userPrompt;
       let finalAnswer = "";
       let round = 0;

       // 2. Agent loop
       while (nextPrompt && round < maxRounds) {
         const response = await conversation.generate(nextPrompt, 0);
         const parsed = parseXmlFunctionCalls(response);

         // 3. Execute tools
         const toolsToCall = parsed.functionCalls
           .filter((func) => func.name in this.tools)
           .map((func) => this.tools[func.name].execute(func.parameters));

         if (toolsToCall.length === 0) {
           finalAnswer = parsed.cleanText;
           break;
         }

         // 4. Collect results
         const functionResults = await Promise.all(toolsToCall);
         nextPrompt = functionResults
           .map((result) => {
             if (result.render && renderCallback) {
               renderCallback(result.render());
             }
             return result.nextPrompt;
           })
           .join("\n\n");
         
         round++;
       }

       return finalAnswer || "Unable to answer";
     };
     ```

3. **Explain the Utility Functions** (pause to review)
   - `toolsToSystemPrompt`: Converts tool definitions to text for the LLM
   - `parseXmlFunctionCalls`: Extracts `<function_call>` XML from LLM response
   - Point to the utils folder - already implemented

### Talking Points
- "This is a ReAct pattern - Reasoning and Acting in a loop"
- "The LLM decides which tools to call based on the descriptions and examples"
- "XML format is simpler than JSON for LLMs to generate correctly"
- "The render callback lets tools show UI feedback (like 'Searching...')"
- "Max rounds prevents infinite loops if the agent gets stuck"

### Common Questions
- **Q: Why not just use function calling APIs?**  
  A: WebLLM doesn't support native function calling, so we use XML prompting
  
- **Q: What if the LLM calls a non-existent tool?**  
  A: We filter with `func.name in this.tools` - unknown calls are ignored

---

## Step 6: System Prompt Refinement (10 minutes)
**Commit:** `84fc504e` - "fix system prompt"

### Learning Objectives
- Understand the importance of system prompts
- Learn prompt engineering techniques
- Debug agent behavior issues

### What Changed in This Step
**Modified File:** `src/ai/llm/WebLLM.ts`

This commit fixes a bug in the WebLLM implementation where the system prompt wasn't being logged correctly, making it harder to debug agent behavior.

```typescript
// Before:
console.log("-- SYSTEM PROMPT --");
console.log(messages);  // Wrong! Logs array instead of system prompt

// After:
console.log("-- SYSTEM PROMPT --");
console.log(systemPrompt);  // Correct!
```

### Host Instructions

1. **Explain the Bug** (3 min)
   - System prompts are critical for agent behavior
   - Logging helps debug why the agent isn't working as expected
   - The bug made it impossible to see what instructions the LLM was receiving

2. **Discuss System Prompt Engineering** (7 min)
   - Show example of a good system prompt:
     ```
     You are a helpful shopping assistant for an online store.
     
     Your responsibilities:
     - Answer customer questions about products
     - Help users find products using the available tools
     - Provide friendly and concise responses
     
     When a user asks about products, use the available tools to help them.
     Always provide specific product recommendations when relevant.
     
     Available tools:
     [Tool descriptions will be appended here]
     ```
   
   - Key principles:
     1. **Identity**: "You are a..."
     2. **Responsibilities**: What the agent should do
     3. **Constraints**: What it should NOT do
     4. **Tone**: Friendly, professional, concise
     5. **Instructions**: When to use tools

3. **Live Fix** (demo)
   - Show the before/after in `src/ai/llm/WebLLM.ts`
   - Emphasize: "Small fixes like this are crucial for debuggability"

### Talking Points
- "System prompts are like training a customer service representative"
- "The clearer your instructions, the better the agent performs"
- "Always log your prompts during development"
- "Iterate on prompts based on agent behavior - it's an empirical process"

---

## Step 7: Chat Integration (45 minutes)
**Commit:** `e3079587` - "agent added to chat"

### Learning Objectives
- Integrate the agent into a React component
- Handle async AI operations with proper UI feedback
- Implement tool calling with real application actions
- Create multiple tools for product search and FAQ

### What Changed in This Step
**Modified File:** `src/app/chat/Chat.tsx` (+299 lines, -7 lines)

This is the culmination of the workshop - connecting everything into a working chat interface with multiple tools.

### Key Features Added

1. **Two Tools Implemented:**
   - `openProductOverview`: Navigate to products page with filters
   - `searchFAQ`: Semantic search over FAQ database

2. **Agent Integration:**
   - Swappable LLM backend (WebLLM vs Gemini)
   - UI state management (thinking, response, callback elements)
   - Error handling
   - Loading states

### Host Instructions

#### Part A: Setup Agent with Tools (20 min)

1. **Explain the Architecture** (5 min)
   - Agent lives in React state (useMemo)
   - Tools are defined inline with full type safety
   - Each tool can:
     - Execute an action (navigate, search)
     - Return a prompt for the next iteration
     - Render UI feedback

2. **Live Coding - Product Overview Tool** (7 min)
   - Open `src/app/chat/Chat.tsx`
   - Add agent creation:
     ```typescript
     const agent = React.useMemo(() => {
       const useWebLLM = import.meta.env.VITE_USE_WEBLLM === 'true';
       const agent = useWebLLM ? new Agent() : new AiSdkAgent();

       // Add tools...
       return agent;
     }, []);
     ```
   
   - Implement the product search tool:
     ```typescript
     agent.addTool(
       "openProductOverview",
       tool({
         description: "open the product overview page with the given filters",
         parameters: z.object({
           categories: z.array(z.nativeEnum(Category)).optional(),
           colors: z.array(z.nativeEnum(Color)).optional(),
           sizes: z.array(z.nativeEnum(Size)).optional(),
         }),
         execute: async (data) => {
           const query = Object.entries(data)
             .filter(([, value]) => value)
             .map(([key, value]) => `${key}=${value}`)
             .join("&");
           navigate(`/products?${query}`);
           return {
             nextPrompt: `Tell the user you just opened the product overview with ${query}`,
             render: () => (
               <p className="flex items-center gap-3 rounded-lg border bg-white p-3">
                 <CheckIcon className="size-8 text-lime-700" />
                 <span>Open Product Overview with {query}</span>
               </p>
             ),
           };
         },
         examples: [
           {
             query: "Show me all the T-Shirts in L",
             parameters: { categories: [Category.CLOTHING], sizes: [Size.L] },
           },
           {
             query: "Show me all red and green products",
             parameters: { colors: [Color.RED, Color.GREEN] },
           },
         ],
       })
     );
     ```

3. **Live Coding - FAQ Search Tool** (8 min)
   ```typescript
   agent.addTool(
     "searchFAQ",
     tool({
       description: "search the FAQ for relevant information to answer the user's question",
       parameters: z.object({
         query: z.string().describe("The search query"),
         results: z.number().optional().describe("Number of results to return (default 4)"),
       }),
       execute: async ({ query, results = 4 }) => {
         const faqs = await findSimilarFAQs(query, results);
         const faqText = faqs
           .map((faq) => `Q: ${faq.question}\nA: ${faq.answer}`)
           .join("\n\n");
         
         return {
           nextPrompt: `Here are relevant FAQs:\n\n${faqText}\n\nUse this information to answer the user's question.`,
           render: () => (
             <div className="rounded-lg border bg-white p-3 shadow-md">
               <p className="font-semibold">Searching FAQs...</p>
               <ul className="mt-2 space-y-1">
                 {faqs.map((faq, i) => (
                   <li key={i} className="text-sm text-gray-600">
                     • {faq.question}
                   </li>
                 ))}
               </ul>
             </div>
           ),
         };
       },
       examples: [
         { query: "What is your return policy?", parameters: { query: "return policy" } },
         { query: "How much does shipping cost?", parameters: { query: "shipping costs" } },
       ],
     })
   );
   ```

#### Part B: Handle Chat Submission (15 min)

1. **Implement Form Handler** (10 min)
   ```typescript
   const handleSubmit = async (prompt: string) => {
     setThinking(true);
     setResponse("");
     setCallbackElements([]);

     try {
       const systemPrompt = `You are a helpful shopping assistant.
       
       Your responsibilities:
       - Help users find products using the openProductOverview tool
       - Answer questions about policies using the searchFAQ tool
       - Be friendly and concise
       
       When users ask about products (e.g., "show me red shirts"), use openProductOverview.
       When users ask about policies or info, use searchFAQ first.`;

       const answer = await agent.processPrompt(
         systemPrompt,
         prompt,
         5,
         (element) => {
           setCallbackElements((prev) => [...prev, element]);
         }
       );

       setResponse(answer);
     } catch (error) {
       console.error("Chat error:", error);
       setResponse("Sorry, I encountered an error. Please try again.");
     } finally {
       setThinking(false);
     }
   };
   ```

2. **Update UI States** (5 min)
   - Show the thinking loader
   - Display callback elements (tool execution feedback)
   - Render the final markdown response
   - Handle errors gracefully

#### Part C: Testing (10 min)

1. **Test Product Search**
   - "Show me all red products"
   - "I want to see hoodies in size M"
   - "Show me accessories"

2. **Test FAQ Search**
   - "What's your return policy?"
   - "How much is shipping?"
   - "Do you ship internationally?"

3. **Test Multi-Turn**
   - "Show me t-shirts" → (navigates) → "What sizes do you have?" → (reads page context)

4. **Debug Common Issues**
   - Agent doesn't call tools → Check system prompt, examples
   - Wrong parameters → Check Zod schema descriptions
   - Slow responses → Expected with WebLLM, try Gemini

### Talking Points
- "Tool examples are critical - they're few-shot learning for the LLM"
- "The render callbacks provide immediate user feedback while the agent thinks"
- "Zod schemas provide both runtime validation and LLM parameter hints"
- "We can swap LLMs with one environment variable - abstraction pays off"

### Common Issues & Solutions

| Issue | Likely Cause | Solution |
|-------|--------------|----------|
| Agent doesn't call tools | Unclear tool description | Add more detailed description and examples |
| Wrong parameters | Poor parameter descriptions | Enhance Zod schema descriptions |
| Infinite loop | Tool returns conflicting info | Add better nextPrompt instructions |
| Slow on first use | Model download | Expected, show progress, use smaller model |
| FAQ not relevant | Query transformation issue | Log the generated query, adjust examples |

---

## Conclusion & Next Steps (20 minutes)

### Wrap-Up Discussion
1. **What We Built:**
   - AI-powered shopping assistant
   - Local + Cloud LLM integration
   - Semantic search with vector embeddings
   - Agentic system with tool calling
   - Type-safe tool definitions with Zod

2. **Key Takeaways:**
   - AI in browsers is practical and powerful
   - Abstraction allows swapping LLMs easily
   - Tools extend LLM capabilities infinitely
   - System prompts are crucial for behavior
   - Type safety (Zod) prevents runtime errors

3. **Architecture Patterns Learned:**
   - Singleton pattern for expensive resources (LLM engine)
   - Conversation pattern for stateful interactions
   - ReAct pattern for agentic behavior
   - Strategy pattern for swappable LLMs
   - Render props for flexible UI feedback

### Extension Ideas for Participants
1. **More Tools:**
   - `addToCart` - Add products directly from chat
   - `searchProducts` - Full-text product search
   - `getProductDetails` - Show specific product info
   - `trackOrder` - Order status lookup

2. **Enhanced Features:**
   - Streaming responses (show text as it generates)
   - Conversation history (show past messages)
   - Voice input/output
   - Multi-modal (image search)

3. **Production Improvements:**
   - Backend API for Gemini (hide API key)
   - Caching for vector search
   - Error boundaries and fallbacks
   - Analytics (track tool usage)
   - A/B testing different prompts

### Resources to Share
- **WebLLM:** https://mlc.ai/web-llm/
- **Google AI Studio:** https://aistudio.google.com/
- **Hugging Face Transformers.js:** https://huggingface.co/docs/transformers.js/
- **Anthropic Prompt Engineering:** https://docs.anthropic.com/claude/docs/prompt-engineering
- **Vercel AI SDK:** https://sdk.vercel.ai/ (used in the AiSdkAgent alternative)

### Q&A Topics to Prepare For
1. **How do I deploy this?**
   - Frontend: Any static host (Vercel, Netlify, etc.)
   - Backend proxy: For API keys (Express, Next.js API routes)
   - Consider model caching for WebLLM

2. **What about costs?**
   - WebLLM: Free (user's compute)
   - Gemini: Free tier generous, then $0.50/1M tokens
   - Embeddings: Free with transformers.js in browser

3. **How do I improve accuracy?**
   - Better system prompts (iterate!)
   - More tool examples
   - Larger models (Gemini Pro vs Flash)
   - RAG (Retrieval Augmented Generation) with more context

4. **Can I use this with other frameworks?**
   - Yes! All AI logic is framework-agnostic
   - Vue, Angular, Svelte - just wrap in components
   - Even vanilla JS works

---

## Troubleshooting Guide

### Setup Issues

**Problem:** `npm install` fails  
**Solution:** 
- Check Node.js version (need 18+)
- Clear npm cache: `npm cache clean --force`
- Delete `node_modules` and `package-lock.json`, reinstall

**Problem:** WebGPU not supported  
**Solution:**
- Use Chrome/Edge (best support)
- Enable WebGPU flags if needed: `chrome://flags/#enable-unsafe-webgpu`
- Use Gemini instead (set `VITE_USE_WEBLLM=false`)

### Runtime Issues

**Problem:** Model download fails  
**Solution:**
- Check browser console for specific error
- Try smaller model (GEMMA_2_2B_ID)
- Check disk space (need 5GB+)
- Clear IndexedDB cache

**Problem:** API key not working  
**Solution:**
- Verify key in .env file
- Restart dev server after .env changes
- Check key is valid at https://aistudio.google.com/apikey
- Ensure no extra spaces in .env file

**Problem:** Tools not being called  
**Solution:**
- Check browser console for logs
- Verify system prompt includes tool descriptions
- Add more examples to tool definition
- Try simpler, more direct queries

**Problem:** Slow responses (>30 seconds)  
**Solution:**
- Normal for WebLLM on first run (model loading)
- Switch to Gemini for faster responses
- Check if browser is throttling (dev tools open)
- Try smaller model

---

## Workshop Timing Breakdown

| Section | Duration | Cumulative |
|---------|----------|------------|
| Introduction | 15 min | 0:15 |
| Step 1: Project Setup | 20 min | 0:35 |
| Step 2: WebLLM Config | 15 min | 0:50 |
| **Break** | **10 min** | **1:00** |
| Step 3: WebLLM Implementation | 30 min | 1:30 |
| Step 4: Gemini Integration | 30 min | 2:00 |
| **Break** | **10 min** | **2:10** |
| Step 5: Agent & Vector Search | 45 min | 2:55 |
| Step 6: Prompt Refinement | 10 min | 3:05 |
| **Break** | **10 min** | **3:15** |
| Step 7: Chat Integration | 45 min | 4:00 |
| Conclusion & Q&A | 20 min | 4:20 |

**Total: ~4 hours 20 minutes (with breaks)**  
**Adjust:** Can shorten by skipping WebLLM (use only Gemini) to save ~45 min

---

## Pre-Workshop Checklist for Hosts

### 1 Week Before
- [ ] Send setup instructions to participants
- [ ] Verify all participants have Node.js 18+
- [ ] Share Google AI Studio link for API keys
- [ ] Test workshop on multiple browsers
- [ ] Prepare slides for introduction

### 1 Day Before
- [ ] Test complete workshop flow
- [ ] Verify all commits are accessible
- [ ] Check internet connection at venue
- [ ] Have backup internet (hotspot)
- [ ] Download models locally as backup

### Day Of
- [ ] Arrive 30 min early for setup
- [ ] Test projector/screen share
- [ ] Have workshop repo open in VS Code
- [ ] Open browser tabs: GitHub, Google AI Studio, docs
- [ ] Test WebGPU on presentation machine
- [ ] Have complete branch ready to demo
- [ ] Print troubleshooting guide

### During Workshop
- [ ] Take breaks every hour
- [ ] Monitor chat/questions continuously
- [ ] Walk around to help stuck participants
- [ ] Take photos (with permission)
- [ ] Note common issues for future workshops

### After Workshop
- [ ] Share completed code repository
- [ ] Send resource links
- [ ] Send feedback survey
- [ ] Answer follow-up questions
- [ ] Update workshop notes based on learnings

---

## Additional Notes for Hosts

### Pacing Tips
- **Go Slow:** AI concepts are new to many developers
- **Pause Often:** Ask "any questions?" frequently
- **Live Code:** Type everything, don't copy-paste (they learn from seeing)
- **Check Understanding:** Ask participants to explain concepts back
- **Be Patient:** Model downloads take time, plan for it

### Engagement Strategies
- **Pair Programming:** Have participants work in pairs
- **Show & Tell:** Have pairs demo their agents
- **Debug Together:** When someone has an issue, debug as a group
- **Celebrate Wins:** First successful agent call is exciting!

### Common Participant Questions
1. "Can I use ChatGPT API instead?" → Yes, similar pattern
2. "How do I make the agent remember past conversations?" → Store message history in React state
3. "Can this work on mobile?" → WebLLM no (needs WebGPU), Gemini yes
4. "Is this production-ready?" → With backend proxy and error handling, yes

### Backup Plans
- **No Internet:** Have models pre-downloaded, use WebLLM only
- **WebGPU Issues:** Switch entire workshop to Gemini
- **Time Running Short:** Skip WebLLM implementation, focus on Agent
- **Too Easy:** Add extension challenges (voice input, streaming)
- **Too Hard:** Provide more complete code snippets, reduce live coding

---

## Success Metrics

By the end of the workshop, participants should be able to:
- ✅ Explain the difference between local and cloud LLMs
- ✅ Implement a basic LLM wrapper in TypeScript
- ✅ Create semantic search using vector embeddings
- ✅ Build an agentic system with tool calling
- ✅ Define type-safe tools with Zod schemas
- ✅ Integrate AI into a React application
- ✅ Debug agent behavior using logs and prompts
- ✅ Extend the system with new tools independently

---

## Credits & Acknowledgments

**Workshop Created By:** Shivay Lamba (Couchbase)  
**Event:** React Alicante 2025  
**Technologies:** React, TypeScript, WebLLM, Google Gemini, Hugging Face Transformers.js, Zod

Special thanks to:
- MLC LLM team for WebLLM
- Google for Gemini API
- Hugging Face for Transformers.js
- React Alicante organizers

---

*These notes are a living document. After each workshop, update with learnings, common issues, and improvements.*
