<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Serverless LLM Audio Summarizer — README</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial; line-height:1.6; color:#111; max-width:1000px; margin:32px auto; padding:0 20px; }
    header { border-left:6px solid #ff9900; padding:12px 20px; background:#fffaf0; margin-bottom:20px; border-radius:6px; }
    h1 { margin:0 0 8px; font-size:28px; }
    h2 { margin-top:28px; }
    pre { background:#f6f8fa; padding:12px; border-radius:6px; overflow:auto; }
    code { background:#f2f2f2; padding:2px 6px; border-radius:4px; font-family:monospace; }
    ul { margin-top:8px; }
    .badge { display:inline-block; margin-right:8px; padding:4px 8px; background:#eef; border-radius:999px; font-size:12px; }
    .note { background:#eef9ff; border-left:4px solid #7fbfff; padding:8px 12px; border-radius:4px; }
    .file-list code { display:block; margin:6px 0; }
    footer { margin-top:36px; color:#666; font-size:13px; }
    .commands { margin-top:8px; }
  </style>
</head>
<body>
  <header>
    <h1>Serverless LLM Audio Summarizer</h1>
    <div>Deploying a large-language-model based application into production using serverless technology (Amazon Bedrock, Titan, Transcribe, AWS Lambda).</div>
  </header>

  <section>
    <h2>Project Overview</h2>
    <p>
      This repository demonstrates a pattern for building a serverless application that pairs Automatic Speech Recognition (ASR) with a Large Language Model (LLM) to convert audio recordings into summaries. The system is designed to be event-driven — incoming audio triggers transcription and then summarization — and production-ready with logging for security, auditing, and compliance.
    </p>

    <p><strong>Key features:</strong></p>
    <ul>
      <li>Prompt and customize LLM responses using <code>Amazon Bedrock</code> (Titan model).</li>
      <li>Transcribe audio using <code>Amazon Transcribe</code>.</li>
      <li>Enable structured logging for LLM calls and ASR operations.</li>
      <li>Deploy the workflow as an event-driven serverless pipeline using <code>AWS Lambda</code>.</li>
      <li>Example notebooks that show how each piece is implemented and tested.</li>
    </ul>
  </section>

  <section>
    <h2>Files / Notebooks</h2>
    <div class="file-list">
      <code>1_first_generations_with_Amazon_Bedrock.ipynb</code>
      <p>Introductory notebook showing how to call Amazon Bedrock (Titan) to perform text generations and how to craft prompts and handle responses.</p>

      <code>2_Summarize_an_audio_file.ipynb</code>
      <p>End-to-end example that takes an audio file, transcribes it (Amazon Transcribe or other ASR), and summarizes the transcript using an LLM.</p>

      <code>3_Enable_logging.ipynb</code>
      <p>Shows how to enable structured logging for LLM and ASR calls; helpful for auditing, debugging, and compliance.</p>

      <code>4_Deploy_an_AWS_Lambda_function.ipynb</code>
      <p>Deploys the summarizer as an AWS Lambda function, packages dependencies, and demonstrates how to configure the function for event-driven processing.</p>

      <code>5_Event-driven_generation.ipynb</code>
      <p>Builds an event-driven flow: detects new customer inquiries (uploaded audio), triggers transcription, and invokes the LLM to generate a summary. Demonstrates integration points and best practices.</p>
    </div>
  </section>

  <section>
    <h2>Architecture (high-level)</h2>
    <p>
      The pattern used in this repo follows an event-driven serverless flow:
    </p>
    <ol>
      <li><strong>Audio Upload</strong> (S3 or similar) — a user uploads an audio file.</li>
      <li><strong>Event Trigger</strong> — S3 event (or API Gateway) triggers a Lambda function.</li>
      <li><strong>Transcription</strong> — Lambda calls <code>Amazon Transcribe</code> (or another ASR) to generate a transcript.</li>
      <li><strong>LLM Summarization</strong> — Lambda calls <code>Amazon Bedrock</code> (Titan) to summarize the transcript and optionally structure the output.</li>
      <li><strong>Logging & Storage</strong> — Results and logs are saved (CloudWatch, S3, or a DB) for auditing and later retrieval/analytics.</li>
    </ol>

    <div class="note">
      <strong>Notes:</strong> The LLM (Titan via Bedrock) is used for summarization and prompt-based customization. You should design prompts carefully (examples in notebooks) and ensure you do not send PII to models without appropriate governance.
    </div>
  </section>

  <section>
    <h2>Prerequisites</h2>
    <ul>
      <li>An AWS account with permissions to use: Bedrock, Transcribe, Lambda, S3, IAM, CloudWatch, and optionally EventBridge / API Gateway.</li>
      <li>Python 3.9+ environment for notebooks.</li>
      <li>AWS CLI configured locally (<code>aws configure</code>).</li>
      <li>Required Python libraries (see <code>requirements</code> below).</li>
    </ul>
  </section>

  <section>
    <h2>Example Requirements</h2>
    <p>Create a <code>requirements.txt</code> similar to:</p>
    <pre><code>boto3
requests
botocore
jupyter
ipython
pandas
tqdm
# Add your Bedrock SDK / client libs if you have a specific SDK
</code></pre>
  </section>

  <section>
    <h2>Environment & IAM</h2>
    <p>Set environment variables or use AWS profiles for access. Minimal IAM permissions include:</p>
    <ul>
      <li><code>transcribe:StartTranscriptionJob</code>, <code>transcribe:GetTranscriptionJob</code></li>
      <li><code>bedrock:InvokeModel</code> (or equivalent Bedrock actions)</li>
      <li><code>lambda:CreateFunction</code>, <code>lambda:InvokeFunction</code></li>
      <li><code>s3:PutObject</code>, <code>s3:GetObject</code></li>
      <li><code>logs:CreateLogGroup</code>, <code>logs:CreateLogStream</code>, <code>logs:PutLogEvents</code></li>
    </ul>
    <p class="note">Grant least privilege required and use role assumptions for Lambda instead of embedding permanent credentials in code.</p>
  </section>

  <section>
    <h2>How to run the notebooks (local)</h2>
    <ol>
      <li>Clone the repo and create a virtual environment:
        <pre class="commands"><code>python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab  # or jupyter notebook
</code></pre>
      </li>
      <li>Open each notebook in order (1 → 5) to follow the progression from simple LLM generations to a full event-driven deployment.</li>
      <li>Provide AWS credentials via environment, profile, or IAM role. Notebooks contain placeholders for credentials and resource names.</li>
    </ol>
  </section>

  <section>
    <h2>Lambda deployment (summary)</h2>
    <p>Example steps to build and deploy the Lambda handler that runs transcription + summarization:</p>
    <ol>
      <li>Package your handler and dependencies into a Lambda deployment package or container image.</li>
      <li>Create an IAM Role for Lambda with the permissions listed above.</li>
      <li>Create the Lambda function, configure the timeout/memory to account for transcription and LLM latency.</li>
      <li>Configure an S3 event (or API Gateway) to trigger the Lambda when audio is uploaded.</li>
      <li>Ensure logs flow to CloudWatch and consider storing structured outputs in S3 or a DB for retrieval.</li>
    </ol>

    <pre><code># Example (AWS CLI skeleton)
zip -r function.zip lambda_function.py  # include dependencies or use Layers
aws lambda create-function \
  --function-name audio-summarizer \
  --zip-file fileb://function.zip \
  --handler lambda_function.lambda_handler \
  --runtime python3.9 \
  --role arn:aws:iam::123456789012:role/lambda-transcribe-bedrock-role
</code></pre>
  </section>

  <section>
    <h2>Logging & Observability</h2>
    <p>
      Notebook <code>3_Enable_logging.ipynb</code> provides examples for structured, auditable logs. Recommended practices:
    </p>
    <ul>
      <li>Log each interaction with the LLM including request metadata (prompt hash/ID, non-sensitive metadata), timestamps, and response IDs.</li>
      <li>Mask or avoid logging PII or sensitive audio contents. If you must, encrypt logs or store PII in a governed secure store.</li>
      <li>Send critical metrics to CloudWatch (invocation latency, failures, transcription job times) and create alarms for anomalies.</li>
    </ul>
  </section>

  <section>
    <h2>Prompting & LLM best practices</h2>
    <ul>
      <li>Keep prompts specific and use examples (few-shot) for consistent summarization style.</li>
      <li>Prefer structured outputs (JSON schema) when you need machine-readable summaries.</li>
      <li>Rate-limit and backoff on Bedrock calls; handle transient errors gracefully.</li>
    </ul>
  </section>

  <section>
    <h2>Security & Compliance</h2>
    <ul>
      <li>Review data residency and retention policies when sending transcripts to an LLM service.</li>
      <li>Use encryption at rest and in transit (S3 SSE, TLS).</li>
      <li>Apply access controls and audit logs for all Bedrock and Transcribe uses.</li>
    </ul>
  </section>

  <section>
    <h2>Extending this repo</h2>
    <p>Ideas for next steps:</p>
    <ul>
      <li>Support multi-speaker diarization and speaker-labeled summaries.</li>
      <li>Implement a web UI or Slack integration for user requests and summary delivery.</li>
      <li>Persist summaries to a searchable store (OpenSearch, Pinecone, or ChromaDB) for retrieval-augmented workflows.</li>
      <li>Wrap LLM calls in a policy/enforcement layer for sensitive content detection before generation.</li>
    </ul>
  </section>

  <section>
    <h2>Quick reference — environment variables & placeholders</h2>
    <pre><code>AWS_PROFILE=your-aws-profile
S3_BUCKET=your-audio-bucket
BEDROCK_MODEL=titan-<model-name>
TRANSCRIBE_LANGUAGE_CODE=en-US
LAMBDA_ROLE_ARN=arn:aws:iam::123456789012:role/...
</code></pre>
  </section>

  <section>
    <h2>License & Credits</h2>
    <p>Include your preferred open-source license file (e.g., <code>LICENSE</code>) in the repo. This project shows examples using Amazon Bedrock (Titan) and Amazon Transcribe and demonstrates serverless deployment patterns on AWS.</p>
  </section>

  <footer>
    <div>Created to demonstrate building serverless LLM applications with Amazon Bedrock, Titan, and AWS Lambda. For more details, check the notebooks in this repo.</div>
    <div style="margin-top:8px">If you'd like, I can also generate a <code>README.md</code> (Markdown) or a short quickstart <code>setup.sh</code> to accelerate running the examples locally — tell me which you'd prefer.</div>
  </footer>
</body>
</html>
