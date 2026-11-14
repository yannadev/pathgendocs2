# Local RAG feedback

Pathgen uses developer-authored JSON to retrieve approved mathematics guidance and a small local Qwen model to phrase feedback. This happens only when tracked practice ends. The student sees the actual first-try score, retry corrections, a strength if supported, and one next focus. Django owns grading, mastery, focus selection and links.

## Knowledge handoff

The content team creates and reviews the JSON outside Pathgen. Put the approved file at `content/lesson_knowledge.json`. Keep a separate `lesson_knowledge.example.json` with this schema:

```json
{
  "schema_version": 1,
  "content_version": "example-1",
  "entries": [
    {
      "skill_id": "INT_SUB_NEG",
      "lesson_code": "integer_subtraction",
      "section_id": "skill-int-sub-neg",
      "title": "Adding the opposite",
      "explanation": "To subtract an integer, add its opposite.",
      "worked_example": "3 - (-5) = 3 + 5 = 8."
    }
  ]
}
```

Require one entry per included skill and allow partial delivery. Validate schema, types, known lesson/skill pairing, unique skill IDs, nonempty short text, and section anchors. Construct the review URL from a known named route and a validated anchor. Test anchors against delivered pages. The v1 default is structured retrieval by focus skill ID. The app reads the file and supplies the matching content at runtime. If the professor confirms that numeric embedding similarity is required, specify the precomputed vector schema, embedding model, compatible query vector, and validation separately before changing retrieval.

## Runtime sequence

1. Finalize the session and persist authoritative result counts. Pick one focus skill using [special_algo.md](special_algo.md). Save basic feedback and a review link if the destination exists.
2. Retrieve the focus entry by skill ID. If content is absent, keep the deterministic feedback. If available, send a short prompt with the approved explanation/example and minimum result facts. Omit student names, credentials and unrelated records. Ask for brief Grade 7 feedback about the observed strength, if any, and the one next focus.
3. Call the private llama.cpp service with a bounded timeout and short output cap. Treat the model output as plain text. Validate the output before saving, and use the server's score and review link as separate UI fields.
4. Show fallback feedback if knowledge is missing, invalid, or inference fails, times out, or returns unusable text. Finish grading and mastery independently. Ensure repeated results loads reuse the saved feedback. **V1 implementation default:** once the results appear, submit a CSRF-protected request to generate feedback for the student's own completed session. Claim the job once using a conditional database write. Perform inference outside that write, with a short timeout and the fallback already visible. A failed or abandoned claim leaves the fallback available. Concurrent requests reuse the saved status and result.

## Model and hosting

Use Qwen2.5-0.5B-Instruct GGUF `Q4_K_M`, filename `qwen2.5-0.5b-instruct-q4_k_m.gguf`, size 491,400,032 bytes. Verified local source: `C:/Users/jhadz/Downloads/path.dev/model/rag_model/qwen2.5-0.5b-instruct-q4_k_m.gguf`.

SHA-256: `74a4da8c9fdbcd15bd1f6d01d621410d31c6fc00986f5eb687824e7b93d7a9db`. Use the [official file and checksum](https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/blob/main/qwen2.5-0.5b-instruct-q4_k_m.gguf). Fetch from the official Qwen repository into a verified local cache or production image. Pin the source revision or checksum and pin the llama.cpp version. Django calls the local inference server, on the Compose network in development and private Railway networking in production.

Verify a real response with example knowledge, a no-knowledge result, bad JSON, model timeout and concurrent finishes. Verify the final delivered knowledge when available. The 491 MB is the binary size. Measure runtime RAM and latency on the target environment. See [tech_stack.md](tech_stack.md) for deployment.
