Project Overview

  M3-Agent is a multimodal AI agent framework with long-term memory capabilities that can process visual and auditory inputs to build and update its
  memory over time, similar to human cognition.

  Key Components:

  Core Functionality:

- Memorization: Processes video/audio streams to generate episodic and semantic memory stored as multimodal graphs
- Control: Executes instructions through iterative reasoning and memory retrieval
- Long-term Memory: Entity-centric, multimodal format enabling consistent environment understanding

  Architecture:

- m3_agent/: Main module with control logic and memorization pipelines
- mmagent/: Core processing modules (face detection, voice processing, memory management, retrieval)
- configs/: Configuration files for API, memory, and processing settings
- data/: Storage for videos, clips, annotations, and memory graphs

  M3-Bench Dataset:

- M3-Bench-robot: 100 real-world first-person robot videos
- M3-Bench-web: 920 web-sourced videos across diverse scenarios
- Designed to evaluate long-term memory and reasoning capabilities

  Key Features:

- Real-time multimodal input processing (vision + audio)
- Entity-centric memory organization
- Cross-modal reasoning capabilities
- Reinforcement learning training approach
- Performance improvements over GPT-4o and Gemini-1.5-pro baselines

  The project is research-focused, advancing multimodal agents toward human-like memory and practical applications as personal assistants.


● M3-Agent Real-time Multimodal Input Processing Loop

  Based on the code analysis, here's the agent loop for real-time multimodal input processing:

  1. Input Segmentation (memorization_memory_graphs.py:82-96)

# Video is segmented into 30-second clips

  clips = glob.glob(sample["clip_path"] + "/*")
  for clip_path in clips:
      clip_id = int(clip_path.split["/"](-1).split["."](0))
      base64_video, base64_frames, base64_audio = process_video_clip(clip_path)

  2. Multimodal Processing Pipeline (memorization_memory_graphs.py:33-67)

  def process_segment(video_graph, base64_video, base64_frames, base64_audio, clip_id, sample, clip_path):
      # Face Processing
      id2faces = process_faces(video_graph, base64_frames, ...)

      # Voice Processing
      id2voices = process_voices(video_graph, base64_audio, base64_video, ...)

      # Memory Generation
      episodic_memories, semantic_memories = generate_memories(
          base64_frames, id2faces, id2voices, clip_path
      )

      # Memory Integration
      process_memories(video_graph, episodic_memories, clip_id, type="episodic")
      process_memories(video_graph, semantic_memories, clip_id, type="semantic")

  3. VideoGraph Memory Structure (videograph.py:36-62)

- Nodes: Face embeddings, voice embeddings, episodic/semantic memories
- Edges: Relationships between entities with similarity weights
- Clustering: Semantic nodes clustered by cosine similarity (threshold=0.9)
- Entity Mapping: Character ID to name mappings for consistent reference

  4. Control Loop for Query Processing (control.py:100-132)

  def consumer(data):
      if not data["finish"]:
          # Parse agent response for Action: [Answer/Search]
          action = match_result.group(1)  # "Answer" or "Search"
          content = match_result.group(2)  # Answer or search query

          if action == "Answer":
              data["response"] = content
              data["finish"] = True
          else:
              # Memory retrieval
              mem_node = load_video_graph(data["mem_path"])
              memories, current_clips, _ = search(
                  mem_node, content, data["current_clips"],
                  threshold=0.5, topk=processing_config["topk"]
              )
              # Update conversation with retrieved knowledge
              data["conversations"].append({"role": "user", "content": search_result})

  5. Memory Retrieval Mechanism (retrieve.py:76-100)

  def retrieve_from_videograph(video_graph, query, topk=5, mode='max', threshold=0):
      # Query translation using character mappings
      queries = back_translate(video_graph, [query])

      # Embedding-based similarity search
      query_embeddings = parallel_get_embedding(model, queries)[0]

      # Retrieve relevant memory nodes by cosine similarity
      # Returns top-k memories with scores above threshold

  Complete Processing Loop:

  1. Input: 30-second video/audio segments
  2. Extraction: Face detection, voice diarization, frame sampling
  3. Memory Generation: LLM generates episodic/semantic memories
  4. Graph Update: Add nodes/edges, cluster similar entities
  5. Query Processing: Iterative search-answer loop with memory retrieval
  6. Response: Final answer after sufficient knowledge accumulation

  The system maintains persistent memory across clips while processing new inputs in real-time, enabling continuous learning and context-aware
  responses.
