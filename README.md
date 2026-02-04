# React Alicante Workshop: Building AI Agents

A hands-on workshop teaching React developers how to build an AI-powered shopping assistant using WebLLM, Google Gemini, vector search, and agentic patterns.

## 📚 Workshop Materials

- **[WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md)** - Complete host guide with detailed instructions for each step
- **[WORKSHOP_QUICK_REFERENCE.md](./WORKSHOP_QUICK_REFERENCE.md)** - Quick reference with key concepts, commands, and templates

## 🌿 Branches

- `main`: Starting point - blank webshop, ready to implement AI features
- `complete`: Final solution - all AI features implemented

## 🎯 What You'll Build

An AI-powered e-commerce chat assistant that can:
- Search products using natural language
- Answer FAQs with semantic search
- Navigate product pages via chat commands
- Execute actions through tool calling

### Technologies Used
- **WebLLM** - Local AI inference in the browser
- **Google Gemini** - Cloud-based AI alternative
- **Transformers.js** - Vector embeddings for semantic search
- **Zod** - Type-safe tool definitions
- **React + TypeScript** - Modern web development

## 🚀 Getting Started

### Prerequisites
- Node.js 18+
- Modern browser with WebGPU support (Chrome/Edge recommended)
- Google Generative AI API Key ([Get one here](https://aistudio.google.com/apikey))

### Quick Setup

```bash
# Clone the repository
git clone https://github.com/shivaycb/reactalicante-agent-workshop
cd reactalicante-agent-workshop

# Checkout starting point
git checkout main

# Install dependencies
npm install

# Create .env file
echo "VITE_GOOGLE_GENERATIVE_AI_API_KEY=your_api_key_here" > .env
echo "VITE_USE_WEBLLM=false" >> .env

# Start dev server
npm run dev
```

### Environment Variables
Create a `.env` file in the root with:
- `VITE_GOOGLE_GENERATIVE_AI_API_KEY` - Your Google Generative AI API key (required)
- `VITE_USE_WEBLLM` - Set to `true` for WebLLM, `false` for Gemini (optional, defaults to false)
- `PORT` - Local dev server port (optional)

## 📁 Project Structure

```
reactalicante-agent-workshop/
├── public/             # Public assets (product images)
├── src/
│   ├── ai/            # AI components (built during workshop)
│   │   ├── agent/     # Agent implementation
│   │   ├── llm/       # LLM wrappers (WebLLM, Gemini)
│   │   └── vectorSearch/  # Semantic search
│   ├── app/           # Application pages
│   │   ├── chat/      # Chat interface
│   │   ├── header/    # Navigation
│   │   └── products/  # Product listings
│   ├── store/         # Application state
│   │   ├── products.ts   # Product catalog
│   │   └── faq.ts        # FAQ with embeddings
│   ├── theme/         # Reusable UI components
│   ├── utils/         # Utility functions
│   └── App.tsx        # Main application
├── WORKSHOP_HOST_NOTES.md      # Detailed host guide
└── WORKSHOP_QUICK_REFERENCE.md # Quick reference
```

## Dependencies

**React Alicante Workshop AI Agent** is a [React](https://react.dev/) Application that uses [tailwindcss](https://tailwindcss.com/) for styling.

### Key dependencies
*   **[React](https://react.dev/)**: For building the user interface.
*   **[React Router](https://reactrouter.com/)**: For client-side navigation.
*   **[Headless UI](https://headlessui.com/)**: For accessible UI components.
*   **[Heroicons](https://heroicons.com/)**: For icons.
*   **[Tailwind CSS](https://tailwindcss.com/)**: For rapid UI development and styling.
*   **[Nuqs](https://nuqs.vercel.app/)**: For managing URL query parameters.
*   **[Showdown](https://showdownjs.com/)**: For converting Markdown to HTML.

### Build process
*   **[Vite](https://vitejs.dev/)**: For fast development and building.

### AI Tasks
*   **[WebLLM](https://mlc.ai/web-llm/)**: For integrating large language models.
*   **[Hugging Face Transformers](https://huggingface.co/docs/transformers/index)**: For natural language processing (feature-extraction).
*   **[AI SDK Google](https://ai.google.dev/)**: For interacting with Google's AI services.
*   **[AI](https://sdk.vercel.ai/docs)**: For building AI-powered features.

### Developer Experience
*   **[Zod](https://zod.dev/)**: For data validation and type safety, especially for structured output of AI models
*   **[TypeScript](https://www.typescriptlang.org/)**: For static typing.
*   **[ESLint](https://eslint.org/)**: For code quality.
*   **[Prettier](https://prettier.io/)**: For code formatting.

## 🎓 Workshop Steps

The workshop follows a progression of commits on the `main` branch:

1. **Initialize Workshop** - Set up React webshop application
2. **WebLLM Config** - Configure local AI model (Gemma 2)
3. **WebLLM Implementation** - Build browser-based LLM wrapper
4. **Gemini Integration** - Add cloud AI alternative
5. **Agent & Vector Search** - Implement tool calling and semantic search
6. **System Prompt Fix** - Refine prompt engineering
7. **Chat Integration** - Complete the AI assistant

Each step builds upon the previous one. See [WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md) for detailed instructions.

## 🏃 Running the Complete Solution

Want to see the finished product?

```bash
git checkout complete
npm install
npm run dev
```

Then open the chat and try:
- "Show me all red products"
- "What's your return policy?"
- "I want hoodies in size M"

## 🤝 For Workshop Hosts

**New to hosting?** Start with [WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md) for:
- Complete setup instructions
- Detailed teaching guide for each step
- Timing breakdown (~4 hours)
- Troubleshooting tips
- Common questions and answers

**Need a quick reference?** Check [WORKSHOP_QUICK_REFERENCE.md](./WORKSHOP_QUICK_REFERENCE.md) for:
- One-page overview
- Key commands and templates
- Architecture diagrams
- Testing queries

## 📖 Learn More

- [WebLLM Documentation](https://mlc.ai/web-llm/)
- [Google AI Studio](https://aistudio.google.com/)
- [Hugging Face Transformers.js](https://huggingface.co/docs/transformers.js/)
- [Prompt Engineering Guide](https://docs.anthropic.com/claude/docs/prompt-engineering)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Credits

Workshop created by [Shivay Lamba](https://github.com/shivaycb) for React Alicante 2025.

Special thanks to the teams behind WebLLM, Google Gemini, and Hugging Face Transformers.js for making AI accessible in the browser.