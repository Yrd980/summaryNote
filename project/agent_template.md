Agent Template: Terminal Frontend + Python Backend

     Architecture Overview

     - Frontend: Rich terminal interface with real-time streaming
     - Backend: Python-based agent engine with all core components
     - Communication: WebSocket/gRPC for real-time bidirectional communication

     Phase 1: Backend Core (Python)

     1. Project Structure
       - src/agent_template/ - Main package
       - Core modules: agent_loop.py, async_queue.py, stream_gen.py, etc.
       - Configuration management with YAML/JSON
       - Requirements.txt with asyncio, websockets, pydantic
     2. AgentLoop - Task scheduler and state management
       - Async task queue with priority scheduling
       - State machine for agent lifecycle
       - Event-driven architecture
     3. AsyncQueue - Communication pipeline
       - Async message queues with backpressure
       - Stream handling with flow control
       - Cancellation and timeout support

     Phase 2: Streaming & Intelligence

     4. StreamGen - Real-time response generation
       - Streaming API integration (OpenAI, Anthropic)
       - Token-by-token output processing
       - Real-time WebSocket streaming to frontend
     5. Message System
       - Session management with SQLite/PostgreSQL
       - Context queue with LRU eviction
       - Temporary cache with TTL
     6. Multi-model Support
       - Abstract model interface
       - Provider adapters (OpenAI, Anthropic, Ollama, etc.)
       - Model switching and fallback

     Phase 3: Advanced Features

     7. Compression & Optimization
       - Message compression using LZ4/zstd
       - Context optimization with summarization
       - History compression with key extraction
     8. Tool System
       - MCP protocol integration
       - Dynamic tool discovery
       - Subagent spawning and management
     9. State Management
       - Tool state persistence
       - Execution history tracking
       - Metrics collection and analysis

     Phase 4: Terminal Frontend

     10. Rich Terminal Interface
       - Python Rich/Textual for advanced TUI
       - Real-time streaming display
       - Interactive command palette
       - Progress bars and status indicators
     11. Communication Layer
       - WebSocket client for real-time updates
       - Command parsing and validation
       - Async input handling

     Phase 5: Integration

     12. API Layer
       - FastAPI/Flask for HTTP endpoints
       - WebSocket server for real-time communication
       - Configuration and health endpoints
     13. CLI & Deployment
       - Click-based CLI interface
       - Docker containerization
       - Environment configuration

     Tech Stack:
     - Backend: Python 3.11+, asyncio, FastAPI, SQLAlchemy
     - Frontend: Rich/Textual, websockets-client
     - Communication: WebSockets, JSON-RPC
     - Storage: SQLite/PostgreSQL, Redis (optional)
     - Packaging: Poetry/pip, Docker
