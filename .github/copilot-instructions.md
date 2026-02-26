# Copilot Instructions for LangChain Lab

## Project Overview
This is a dual-component project:
1. **Learning Lab** (`lab-chains-in-langchain.ipynb`) - Interactive exercises teaching LangChain chain patterns
2. **Streaming API** (`main.py`) - FastAPI server demonstrating async agent streaming with conversation memory

## Key Architecture Patterns

### LangChain Chains (Lab Exercises)
- **LLMChain**: Single-prompt language model wrapper with template variables
- **SequentialChain**: Chains multiple LLMChains where outputs feed into inputs (complex dependency graphs)
- **SimpleSequentialChain**: Simplified sequential execution (output of chain i → input of chain i+1)
- **MultiPromptChain**: Router-based chain selecting specialized prompts by content type (e.g., physics vs. math questions)

All chains use:
- `ChatPromptTemplate.from_template()` for prompt definitions
- `ChatOpenAI(temperature=X)` for LLM initialization (0=deterministic, 0.9=creative)
- `output_key` parameter in SequentialChain to name intermediate outputs

### API Streaming Pattern (main.py)
The `/chat` endpoint implements **token-by-token streaming**:
1. Custom `AsyncCallbackHandler` intercepts LLM tokens in real-time
2. Detects "Final Answer" phase to extract action inputs
3. Yields tokens asynchronously to client via `StreamingResponse`
4. Uses `ConversationBufferWindowMemory(k=5)` to maintain recent chat history

Critical: `streaming=True` flag required in ChatOpenAI initialization and callback assignment happens per-request in `run_call()`.

## Critical Developer Workflows

### Run the Streaming API
```bash
python main.py
# Server starts at localhost:8000
# Health check: GET /health
# Chat endpoint: POST /chat with JSON {"text": "your query"}
```

### Execute Lab Exercises
Open `lab-chains-in-langchain.ipynb` in Jupyter/VS Code and run cells sequentially. Cells include:
- Data loading from `data/Data.csv` 
- Placeholder prompts (marked with comments) requiring student implementation
- Product description and review analysis examples

## Environment & Dependencies

**Required API Keys** (set in `.env`):
- `OPENAI_API_KEY` - Required for ChatOpenAI
- `PINECONE_API_KEY` - Referenced but unused in current code
- `HUGGINGFACEHUB_API_TOKEN` - Referenced but unused

**Key Dependencies**:
- `langchain`, `langchain_openai`, `langchain_community` - Chain/prompt/agent abstractions
- `fastapi`, `uvicorn` - API server
- `pydantic` - Request validation (Query model)
- `pandas` - Data loading in notebook

## Project-Specific Conventions

### Memory Management
- Always use `return_messages=True` in ConversationBufferWindowMemory for multi-turn agent conversations
- `memory_key="chat_history"` is the standard key name for agent compatibility

### Callback Architecture
- Inherit from `AsyncIteratorCallbackHandler` for async streaming implementations
- Override `on_llm_new_token()` to intercept tokens; `on_llm_end()` for completion logic
- Assign callbacks per-request (not globally) for stateless API design

### Prompt Design
- Use `{variable_name}` syntax in PromptTemplate strings
- Sequential chains require unique `output_key` names (e.g., "english_review", "summary")
- Router chains need `destinations_str` listing all available specialty prompts

## File Organization
```
├── main.py               # FastAPI streaming server
├── lab-chains-in-langchain.ipynb  # Interactive exercises (incomplete, students fill placeholders)
├── data/
│   └── Data.csv         # Review/product data for exercises
├── .env                 # API keys (not in repo)
└── .github/
    └── copilot-instructions.md  # This file
```

## When Helping Students/Contributors

1. **Lab Exercises**: Guide on chain composition logic, not just syntax. E.g., "What should the input to chain_two be?" not "Use LLMChain()"
2. **API Development**: Focus on the callback pattern—explain why `AsyncCallbackHandler` intercepts tokens and why final_answer flag matters
3. **Debugging Chains**: Use `verbose=True` on chains to see intermediate steps and routing decisions
4. **Memory Issues**: If agents lose context, check `k=5` window size and `return_messages=True` setting
