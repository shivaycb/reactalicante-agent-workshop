# React Alicante Workshop AI Agent

A hands-on workshop for building AI-powered features in React, including local LLM integration, cloud APIs, AI agents with function calling, and RAG (Retrieval-Augmented Generation).

## Workshop Documentation

📚 **For Workshop Hosts**: See [WORKSHOP_HOST_NOTES.md](./WORKSHOP_HOST_NOTES.md) for comprehensive teaching materials, step-by-step guides, and troubleshooting tips.

⚡ **Quick Reference**: See [WORKSHOP_QUICK_REFERENCE.md](./WORKSHOP_QUICK_REFERENCE.md) for commit history, commands, and key concepts summary.

✅ **For Participants**: Use [PARTICIPANT_CHECKLIST.md](./PARTICIPANT_CHECKLIST.md) to track your progress through the workshop.

## Repository Structure

There are two branches:

- `main`: blank webshop, ready to implement AI features
- `complete`: all AI features implemented

Each commit in the history represents a workshop step with incremental AI feature additions.

## Getting Started

```
npm install
npm run dev
```

### Env
This project supports two env variables in a `.env` file in the root of the project:
- `PORT`: the local port on which the dev server will listen (optional)
- `VITE_GOOGLE_GENERATIVE_AI_API_KEY`: the API_KEY for the Google Generative AI Studio [https://aistudio.google.com/apikey](https://aistudio.google.com/apikey)

## Setup
```
- public      // public assets (mainly images)
- src         // main application logic
- - app       // layout components
- - store     // react context provider, data
- - theme     // UI components with limited logic
- - utils     // utility functions (some will be relevant later in the workshop)
- - App.tsx   // main App
- - index.css // set up tailwindcss
- - main.tsx  // main entry-point
- index.html  // set up vite
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

## Workshop Topics Covered

This workshop teaches participants to build a complete AI-powered shopping assistant:

1. **Local LLM Integration** - Run large language models in the browser using WebLLM and WebGPU
2. **Cloud LLM Integration** - Connect to Google's Gemini API for cloud-based inference
3. **AI Agents** - Build agents with function calling and tool execution capabilities
4. **RAG (Retrieval-Augmented Generation)** - Implement semantic search using vector embeddings
5. **React Integration** - Connect AI capabilities to a production-like React application

### Learning Outcomes

By the end of this workshop, you will:
- ✅ Understand the differences between local and cloud-based LLM inference
- ✅ Implement conversation management with LLMs
- ✅ Build AI agents with function calling (tool use)
- ✅ Create RAG systems using vector embeddings for semantic search
- ✅ Integrate AI agents into React applications

## License

MIT