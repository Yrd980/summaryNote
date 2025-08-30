## Project Overview

### Project Name
[Owncast](https://github.com/owncast/owncast)

### Description
Owncast is an open source, self-hosted, decentralized, single-user live video streaming and chat server. It allows you to run your own live streams, offering complete ownership over your content, interface, moderation, and audience. It aims to provide a similar experience to large mainstream platforms, but with full control for the user.

### Key Features
- Self-hosted live streaming
- Chat functionality for real-time interactions
- Fully decentralized
- Complete control over content and audience

---

## Project Structure
```
owncast/                                    # Root directory - Self-hosted live streaming platform
├── main.go                                # Main entry point - Initializes core services, database, and web server
├── go.mod                                 # Go module definition with dependencies
├── go.sum                                 # Go module checksums
├── Dockerfile                             # Container configuration for deployment
├── Earthfile                              # Earthly build configuration
├── package-lock.json                      # Node.js dependency lock file
├── renovate.json                          # Automated dependency update configuration
├── lefthook.yml                           # Git hooks configuration
├── sqlc.yaml                              # SQL code generation configuration
├── openapi.yaml                           # OpenAPI specification for API documentation
├── redocly.yaml                           # API documentation generator config
├── crowdin.yml                            # Localization platform configuration
│
├── config/                                # Configuration management
│   ├── config.go                          # Core configuration structures and defaults
│   ├── configUtils.go                     # Configuration utility functions
│   ├── constants.go                       # Application constants
│   ├── defaults.go                        # Default configuration values
│   ├── updaterConfig_enabled.go           # Auto-updater configuration
│   └── verifyInstall.go                   # Installation verification logic
│
├── core/                                  # Core application logic
│   ├── core.go                            # Core service initialization and management
│   ├── cache/                             # Caching layer implementation
│   ├── chat/                              # Real-time chat system
│   ├── data/                              # Data layer and persistence setup
│   ├── offlineState.go                    # Offline state management
│   ├── playlist/                          # Media playlist management
│   ├── rtmp/                              # RTMP server for receiving streams
│   ├── stats.go                           # Statistics collection
│   ├── status.go                          # Stream status management
│   ├── storage.go                         # Storage abstraction layer
│   ├── storageproviders/                  # Storage provider implementations (S3, local)
│   ├── streamState.go                     # Stream state management
│   ├── transcoder/                        # Video transcoding pipeline
│   └── webhooks/                          # Webhook system for external integrations
│
├── webserver/                             # HTTP server and API endpoints
│   ├── handlers/                          # HTTP request handlers
│   ├── router/                            # URL routing configuration
│   └── utils/                             # Web server utilities
│
├── web/                                   # Frontend React/Next.js application
│   ├── package.json                       # Frontend dependencies and scripts
│   ├── next.config.js                     # Next.js configuration
│   ├── tsconfig.json                      # TypeScript configuration
│   ├── components/                        # React components (215+ files)
│   ├── pages/                             # Next.js page components
│   ├── styles/                            # CSS/SCSS styling
│   ├── utils/                             # Frontend utility functions
│   ├── services/                          # API service layer
│   ├── interfaces/                        # TypeScript type definitions
│   ├── i18n/                              # Internationalization files
│   ├── public/                            # Static assets
│   ├── stories/                           # Storybook component stories
│   └── tests/                             # Frontend test files
│
├── models/                                # Data models and structures
│   ├── auth.go                            # Authentication models
│   ├── broadcaster.go                     # Broadcaster information models
│   ├── chatAccessScopes.go                # Chat access control models
│   ├── client.go                          # Client connection models
│   ├── configEntry.go                     # Configuration entry models
│   ├── currentBroadcast.go                # Current broadcast state models
│   ├── eventType.go                       # Event type definitions
│   ├── externalAction.go                  # External action models
│   ├── externalAPIUser.go                 # External API user models
│   ├── federatedActivity.go               # ActivityPub federation models
│   ├── follower.go                        # Follower relationship models
│   ├── notification.go                    # Notification models
│   ├── playlist.go                        # Playlist models
│   ├── socialHandle.go                    # Social media handle models
│   ├── stats.go                           # Statistics models
│   ├── status.go                          # Status models
│   ├── storageProvider.go                 # Storage provider models
│   ├── streamHealth.go                    # Stream health monitoring models
│   ├── streamOutputVariant.go             # Stream output variant models
│   ├── user.go                            # User account models
│   ├── viewer.go                          # Viewer models
│   └── webhook.go                         # Webhook models
│
├── persistence/                           # Data persistence layer
│   ├── authrepository/                    # Authentication data access
│   ├── chatmessagerepository/             # Chat message persistence
│   ├── configrepository/                  # Configuration persistence
│   ├── userrepository/                    # User data persistence
│   ├── webhookrepository/                 # Webhook data persistence
│   └── tables/                            # Database table definitions
│
├── db/                                    # Database layer
│   ├── db.go                              # Database connection and setup
│   ├── models.go                          # Database model definitions
│   ├── query.sql                          # SQL queries
│   ├── query.sql.go                       # Generated Go code from SQL
│   ├── schema.sql                         # Database schema
│   └── README.md                          # Database documentation
│
├── activitypub/                           # ActivityPub federation support
│   ├── activitypub.go                     # Main ActivityPub service
│   ├── apmodels/                          # ActivityPub data models
│   ├── controllers/                       # ActivityPub API controllers
│   ├── crypto/                            # Cryptographic operations
│   ├── inbox/                             # Incoming federation handling
│   ├── outbox/                            # Outgoing federation handling
│   ├── persistence/                       # Federation data persistence
│   ├── requests/                          # Federation request handling
│   ├── resolvers/                         # Federation data resolution
│   ├── webfinger/                         # WebFinger protocol support
│   └── workerpool/                        # Background task processing
│
├── auth/                                  # Authentication system
│   ├── auth.go                            # Main authentication logic
│   ├── fediverse/                         # Fediverse authentication
│   ├── indieauth/                         # IndieAuth authentication
│   └── persistence.go                     # Auth data persistence
│
├── notifications/                         # Notification system
│   ├── notifications.go                   # Main notification service
│   ├── channels.go                        # Notification channel management
│   ├── browser/                           # Browser push notifications
│   └── discord/                           # Discord integration
│
├── metrics/                               # Monitoring and metrics
│   ├── metrics.go                         # Main metrics collection
│   ├── alerting.go                        # Alert system
│   ├── hardware.go                        # Hardware monitoring
│   ├── healthOverview.go                  # Health status overview
│   ├── playback.go                        # Playback metrics
│   ├── prometheus.go                      # Prometheus metrics export
│   ├── timestampedValue.go                # Time-series data structures
│   └── viewers.go                         # Viewer analytics
│
├── utils/                                 # Utility functions
│   ├── utils.go                           # General utility functions
│   ├── accessTokens.go                    # Access token management
│   ├── backup.go                          # Database backup utilities
│   ├── clientId.go                        # Client ID generation
│   ├── db.go                              # Database utilities
│   ├── emojiMigration.go                  # Emoji migration utilities
│   ├── hashing.go                         # Cryptographic hashing
│   ├── netutils.go                        # Network utilities
│   ├── nulltime.go                        # Nullable time handling
│   ├── performanceTimer.go                # Performance measurement
│   ├── phraseGenerator.go                 # Random phrase generation
│   ├── restendpointhelper.go              # REST API helper utilities
│   └── strings.go                         # String manipulation utilities
│
├── services/                              # External service integrations
│   └── geoip/                             # Geolocation service
│
├── yp/                                    # Yellow Pages directory service
│   ├── yp.go                              # Main YP service for instance discovery
│   ├── api.go                             # YP API endpoints
│   └── README.md                          # YP service documentation
│
├── static/                                # Static file serving
│   ├── static.go                          # Static file handler
│   ├── img/                               # Static images
│   ├── web/                               # Web assets (JS, CSS, fonts)
│   ├── metadata.html.tmpl                 # Metadata template
│   └── offline-v2.ts                      # Offline functionality
│
├── data/                                  # Runtime data storage
│   ├── emoji/                             # Custom emoji storage
│   ├── hls/                               # HLS stream segments
│   ├── logs/                              # Application logs
│   ├── metrics/                           # Metrics data
│   ├── owncast.db                         # SQLite database
│   └── tmp/                               # Temporary files
│
├── build/                                 # Build and deployment scripts
│   ├── develop/                           # Development environment setup
│   ├── gen-api.sh                         # API generation script
│   └── web/                               # Frontend build scripts
│
├── test/                                  # Testing infrastructure
│   ├── automated/                         # Automated test suites
│   ├── fixture/                           # Test data fixtures
│   ├── load/                              # Load testing scripts
│   ├── fakeChat.js                        # Chat simulation for testing
│   ├── ocTestStream.sh                    # Stream testing script
│   ├── populateContent.sh                 # Content population script
│   ├── test-local.sh                      # Local testing script
│   └── userColorsTest.js                  # User color testing
│
├── contrib/                               # Community contributions
│   ├── README.md                          # Contribution guidelines
│   ├── owncast_for_windows.md             # Windows setup guide
│   ├── owncast-sample.service             # Systemd service example
│   ├── owncast-systemd-service.md         # Systemd service documentation
│   └── varnish/                           # Varnish cache configuration
│
├── docs/
```


## **Project Understanding**

### **Architecture Map**
```
[main.go] → [core.Start()] → [RTMP Server + Transcoder + Chat + Web]
    ↓              ↓                    ↓
[Config]    [State Manager]      [Async Workers]
    ↓              ↓                    ↓
[Database]  [Stream State]       [HLS Generation]
    ↓              ↓                    ↓
[Storage]   [ActivityPub]        [Web Interface]
```

### **1. Lifecycle Tracing**

#### **Init Phase**
- **Entry Point**: `main.go` → `core.Start()`
- **Configuration**: Database setup, emoji migration, temp directory creation
- **Dependencies**: RTMP server, transcoder, chat system, webhooks, notifications
- **State Initialization**: Creates offline video stream state

#### **Running Phase**
- **RTMP Server**: Listens for incoming streams, validates stream keys
- **Transcoder**: Converts RTMP to HLS segments using FFmpeg
- **Chat System**: WebSocket-based real-time communication
- **Web Interface**: Next.js frontend with admin controls
- **ActivityPub**: Federation with other social platforms

#### **Queue Management**
- **Worker Pools**: ActivityPub inbox/outbox processing
- **Async Tasks**: Stream notifications, cleanup timers, thumbnail generation
- **Resource Pooling**: HLS segment cleanup, storage management

#### **Shutdown Phase**
- **Graceful Disconnect**: RTMP connection cleanup, transcoder shutdown
- **State Persistence**: Stats saving, offline content transition
- **Resource Cleanup**: Timer cancellation, file cleanup

### **2. State Machine Analysis**

#### **Key States**
```
OFFLINE → [RTMP Connect] → STREAMING → [RTMP Disconnect] → OFFLINE
   ↓           ↓              ↓              ↓
[Offline]  [Validation]   [Transcode]   [Cleanup]
[Content]  [Stream Key]   [HLS Gen]     [Timer Start]
```

#### **State Transitions**
- **OFFLINE → STREAMING**: RTMP connection + valid stream key
- **STREAMING → OFFLINE**: RTMP disconnect or transcoder failure
- **Recovery**: Automatic offline content restoration after 5 minutes

#### **State Dependencies**
- **Stream State**: `_stats.StreamConnected`, `_currentBroadcast`
- **Timers**: Online cleanup (1 min), offline cleanup (5 min)
- **External State**: ActivityPub federation, notifications

### **3. Business-Critical Modules**

#### **A. RTMP Stream Ingestion** (`core/rtmp/`)
```go
function rtmp_handler(connection, stream_key):
    validate_stream_key(stream_key) -> access_granted
    if access_granted:
        create_io_pipe() -> rtmp_pipe
        spawn_transcoder(rtmp_pipe) -> async_task
        set_stream_connected() -> state_transition
    else:
        reject_connection()
```

**Role**: Primary stream input, authentication, connection management
**State Dependencies**: Stream connection status, broadcaster info
**Async Behavior**: Spawns transcoder goroutine, manages connection lifecycle

#### **B. Video Transcoder** (`core/transcoder/`)
```go
function transcoder_pipeline(rtmp_input, output_settings):
    validate_input(rtmp_input) -> input_ready
    spawn_ffmpeg_process(input_ready, output_settings) -> hls_segments
    generate_playlists(hls_segments) -> streaming_content
    handle_completion() -> cleanup_routine
```

**Role**: Video processing, HLS generation, quality variants
**State Dependencies**: Stream output settings, latency levels
**Async Behavior**: FFmpeg subprocess, segment generation, thumbnail creation

#### **C. Chat System** (`core/chat/`)
```go
function chat_server(websocket_connections):
    accept_connections() -> client_pool
    broadcast_messages(client_pool) -> real_time_chat
    handle_user_events() -> state_updates
    manage_connections() -> cleanup_disconnected
```

**Role**: Real-time communication, user management, moderation
**State Dependencies**: User authentication, chat permissions
**Async Behavior**: WebSocket management, message broadcasting

#### **D. ActivityPub Federation** (`activitypub/`)
```go
function federation_manager(followers, content):
    generate_activities(content) -> federated_messages
    distribute_to_followers(federated_messages) -> outbound_queue
    process_incoming_activities() -> inbox_processing
    manage_crypto_keys() -> signature_verification
```

**Role**: Social media integration, content distribution
**State Dependencies**: Follower count, federation settings
**Async Behavior**: Worker pools, outbound message queuing

#### **E. Web Interface** (`web/`)
```go
function admin_interface(stream_status, config):
    render_stream_controls(stream_status) -> admin_panel
    handle_configuration_changes() -> settings_updates
    display_analytics() -> viewer_stats
    manage_content() -> stream_management
```

**Role**: Admin controls, viewer interface, configuration
**State Dependencies**: Stream status, user permissions
**Async Behavior**: Real-time updates via WebSocket

### **4. Async Task Analysis**

#### **Task Creation Points**
1. **RTMP Connection**: `go rtmp.Start(setStreamAsConnected, setBroadcaster)`
2. **Transcoder**: `go _transcoder.Start(true)` in stream connection
3. **Cleanup Timers**: `go func()` for online/offline cleanup
4. **Notifications**: `go func()` for delayed live notifications
5. **ActivityPub**: Worker pools for inbox/outbox processing

#### **Coordination Patterns**
- **Callback Functions**: `TranscoderCompleted` for stream lifecycle
- **Context Cancellation**: `_onlineTimerCancelFunc` for notification timers
- **Shared State**: Global variables with mutex protection in chat system
- **Event-Driven**: WebSocket events trigger chat state updates

#### **Resource Contention**
- **Stream State**: Single transcoder instance, global `_stats`
- **Chat Connections**: Mutex-protected client map
- **File System**: HLS segment cleanup, thumbnail generation
- **Database**: Concurrent read/write operations

### **5. Actionable Experiments**

#### **Experiment 1: State Transition Logging**
```go
// Add to core/streamState.go:setStreamAsConnected()
log.Infof("STATE_TRANSITION: OFFLINE → STREAMING | Time: %v | Broadcaster: %s", 
    time.Now(), broadcaster.Name)
```
**Purpose**: Observe state machine behavior during stream lifecycle
**Risk**: Low - only adds logging

#### **Experiment 2: Async Task Monitoring**
```go
// Add to core/core.go:Start()
go func() {
    ticker := time.NewTicker(30 * time.Second)
    for range ticker.C {
        log.Infof("ASYNC_STATUS: Goroutines: %d | Stream: %v | Chat: %d", 
            runtime.NumGoroutine(), IsStreamConnected(), len(chat.GetClients()))
    }
}()
```
**Purpose**: Monitor concurrent task patterns and resource usage
**Risk**: Low - diagnostic information only

#### **Experiment 3: Federation Behavior Toggle**
```go
// Modify activitypub/activitypub.go:SendLive()
if configRepository.GetFederationEnabled() {
    log.Infof("FEDERATION_DEBUG: Sending live to %d followers", followerCount)
    // existing code...
}
```
**Purpose**: Understand ActivityPub message distribution patterns
**Risk**: Low - enhanced logging for federation events

### **Key Differentiating Features**

1. **Self-Hosted Live Streaming**: Complete control over streaming infrastructure
2. **ActivityPub Federation**: Social media integration without corporate platforms
3. **Real-Time Chat**: WebSocket-based viewer interaction
4. **HLS Generation**: Adaptive bitrate streaming with multiple quality variants
5. **Admin Controls**: Comprehensive web-based stream management

### **Investigation Roadmap**

1. **Start with RTMP flow**: Follow stream from connection to HLS output
2. **Trace state transitions**: Monitor `setStreamAsConnected` → `SetStreamAsDisconnected`
3. **Examine async patterns**: Focus on transcoder goroutines and cleanup timers
4. **Understand federation**: Follow ActivityPub message flow to external platforms
5. **Validate web interface**: Test admin controls and real-time updates

This architecture represents a sophisticated live streaming platform that combines traditional streaming technology with modern web standards and social media federation, making it unique in the self-hosted streaming space.
