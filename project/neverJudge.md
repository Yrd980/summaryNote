## Deep Project Understanding: Never-Judge IDE Analysis

### Architecture Map (Text Diagram)

```
[main.cpp] → [IDEMainWindow] → [CodeEditWidget, LSP, Terminal, FileTree]
     ↓              ↓                        ↓
[QApplication]  [Event Loop]          [Async Workers (QCoro)]
     ↓              ↓                        ↓
[Qt Framework] [Signals/Slots]      [LSP Servers, Shell Processes]
     ↓              ↓                        ↓
[OS]          [State Persistence]    [External APIs (clangd, etc.)]
```

### Core Module Profiles

#### 1. IDEMainWindow

**Role**: Orchestrates IDE interface, managing layout and inter-component communication.  
**State Dependencies**: Project state, widget visibility, current file context.  
**Call Graph**: Upstream - QApplication, menu actions; Downstream - FileTreeWidget, CodeTabWidget, TerminalWidget, etc.  
**Async Behavior**: Delegates to children; no direct spawning.  
**I/O Footprint**: File system (project loading), no network.

**Pseudocode**:

```
function orchestrate_ide_interface(user_action):
    validate(user_action) -> check_project_loaded;
    state_check -> read_current_file_context;
    business_rule_application -> route_action_to_component;
    state_transition -> update_widget_visibility;
    side_effects -> emit_signals_to_children;
    async_task_spawn -> delegate_async_to_lsp_or_terminal;
    return updated_ui_state.
```

#### 2. CodeEditWidget

**Role**: Advanced code editing with syntax highlighting and LSP autocompletion.  
**State Dependencies**: File content, cursor, modification flag, LSP state.  
**Call Graph**: Upstream - CodeTabWidget, IDEMainWindow; Downstream - LanguageServer, Highlighter.  
**Async Behavior**: Spawns QCoro tasks for LSP; mutex for synchronization.  
**I/O Footprint**: File system (read/write files), network (LSP pipes).

**Pseudocode**:

```
function handle_code_edit(input_event):
    validate(input_event) -> check_file_loaded;
    state_check -> read_cursor_and_content;
    business_rule_application -> apply_syntax_or_completion;
    state_transition -> update_modification_flag;
    side_effects -> trigger_lsp_request;
    async_task_spawn -> co_await lsp_completion_or_definition;
    return updated_editor_state.
```

#### 3. LanguageServer

**Role**: Manages LSP protocol for intelligent code assistance.  
**State Dependencies**: Server process, request queues, language configs.  
**Call Graph**: Upstream - CodeEditWidget; Downstream - QProcess, parsers.  
**Async Behavior**: All ops as QCoro tasks; mutex for safety.  
**I/O Footprint**: Network (stdin/stdout to LSP servers).

**Pseudocode**:

```
function process_lsp_request(request):
    validate(request) -> check_server_initialized;
    state_check -> read_language_config;
    business_rule_application -> format_lsp_payload;
    state_transition -> update_request_queue;
    side_effects -> send_to_server_process;
    async_task_spawn -> co_await wait_response;
    return parsed_lsp_response.
```

#### 4. TerminalWidget

**Role**: Integrates terminal for code execution and commands.  
**State Dependencies**: Project root, command history, shell state.  
**Call Graph**: Upstream - IDEMainWindow; Downstream - QTermWidget, Command.  
**Async Behavior**: Delegates to QTermWidget; responds to signals.  
**I/O Footprint**: File system (directory changes), external processes.

**Pseudocode**:

```
function execute_terminal_command(command):
    validate(command) -> check_project_context;
    state_check -> read_current_directory;
    business_rule_application -> format_shell_command;
    state_transition -> update_shell_state;
    side_effects -> send_text_to_shell;
    async_task_spawn -> await_shell_completion;
    return execution_result.
```

#### 5. FileTreeWidget

**Role**: Displays project structure with file operations.  
**State Dependencies**: QFileSystemModel, selected index, shortcuts.  
**Call Graph**: Upstream - IDEMainWindow; Downstream - QFileSystemModel, CodeTabWidget.  
**Async Behavior**: Synchronous file ops.  
**I/O Footprint**: File system (traversal, operations).

**Pseudocode**:

```
function handle_file_operation(file_path, operation):
    validate(file_path) -> check_file_exists;
    state_check -> read_file_metadata;
    business_rule_application -> apply_operation_logic;
    state_transition -> update_model_index;
    side_effects -> emit_operation_signal;
    async_task_spawn -> none;
    return operation_result.
```

### Experimental Validation

Propose 3 minimal interventions:

1. **Strategic Logging in LSP**: Add qDebug for server creation/start and request sending to observe transitions.
2. **LSP Timeout Tweak**: Increase from 3s to 5s to test response handling.
3. **AI Chat Debouncing**: Add flag to prevent rapid requests, validating async behavior.

### Step-by-Step Investigation Roadmap

1. Trace entry from main.cpp to Qt event loop; map init, running, shutdown phases.
2. Map states (init, idle, editing, LSP, running, shutdown) and transitions via handlers.
3. Profile 5 critical modules for roles, dependencies, async, I/O.
4. Analyze async flows: QCoro in LSP, signals/slots coordination, mutex contention, error logging.
5. Validate with experiments: Apply logging/tweaks, observe logs/behavior, revert.
6. Synthesize into architecture map and profiles for ongoing development insights.
