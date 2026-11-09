================================================================================
  AutoRespond.ai — Complete Setup & Configuration Guide
  React + FastAPI + LangGraph + RAG (ChromaDB)
================================================================================

PREREQUISITES
  - Python 3.10 or higher
  - Node.js 18 or higher
  - Git
  - Docker Desktop (optional, only for PostgreSQL + Redis)

================================================================================
  SECTION 1 — QUICK START (Fully Offline / Mock Mode)
================================================================================

No API keys needed. Everything runs locally using mock LLM and mock messaging.

--- Step 1: Clone / Download the project ---

  The project root should look like:
    autorespond-ai/
    ├── backend/
    ├── frontend/
    └── docker-compose.yml

--- Step 2: Set up the Backend ---

  Open a terminal and run:

    cd backend

  Create a virtual environment:
    Windows:    python -m venv venv
    Mac/Linux:  python3 -m venv venv

  Activate it:
    Windows:    .\venv\Scripts\activate
    Mac/Linux:  source venv/bin/activate

  Install dependencies:
    pip install -r requirements.txt

  Start the backend server:
    uvicorn app.main:app --reload --port 8000

  Verify it is running:
    Open http://localhost:8000/api/health in your browser.
    You should see: {"status":"ok","llm_provider":"mock",...}

  API documentation (Swagger UI):
    http://localhost:8000/docs

--- Step 3: Set up the Frontend ---

  Open a SECOND terminal:

    cd frontend
    npm install
    npm run dev

  Open http://localhost:5173 in your browser.
  Register a new account — the first account created is automatically Admin.

================================================================================
  SECTION 2 — DATABASE CONFIGURATION
================================================================================

The database is configured in one line inside:
  backend/.env  (file: DATABASE_URL)

--------------------------------------------------------------------------------
  OPTION A — SQLite (Default, Zero Setup Required)
--------------------------------------------------------------------------------

  In backend/.env:
    DATABASE_URL=sqlite:///./autorespond.db

  - File is created automatically at: backend/autorespond.db
  - No installation, no server, nothing to start
  - All tables are created automatically on first run
  - Best for: local development, demos, single-user use

--------------------------------------------------------------------------------
  OPTION B — PostgreSQL (Recommended for Production / Team Use)
--------------------------------------------------------------------------------

  Step 1 — Start PostgreSQL using Docker Compose:

    cd autorespond-ai        (project root, where docker-compose.yml is)
    docker compose up -d

    This starts:
      - PostgreSQL 16  on port 5432
      - Redis 7        on port 6379

  Step 2 — Edit backend/.env:

    Change this line:
      DATABASE_URL=sqlite:///./autorespond.db

    To this:
      DATABASE_URL=postgresql+psycopg2://postgres:postgres@localhost:5432/autorespond

    Format explained:
      postgresql+psycopg2://  <-- driver (already installed in requirements.txt)
      postgres:postgres       <-- username:password
      @localhost:5432         <-- host:port
      /autorespond            <-- database name

  Step 3 — Restart the backend:

    uvicorn app.main:app --reload --port 8000

    Tables are created automatically on startup — no migration commands needed.

  Step 4 (Optional) — Use a custom PostgreSQL server instead of Docker:

    Replace the connection string with your server details:
      DATABASE_URL=postgresql+psycopg2://myuser:mypassword@myserver.com:5432/mydb

  NOTE: psycopg2-binary driver is already included in requirements.txt.
        No extra pip install needed.

--------------------------------------------------------------------------------
  OPTION C — Enable Celery for Async Campaign Dispatch (Optional)
--------------------------------------------------------------------------------

  Requires Docker (Redis must be running from docker compose up -d).

  Edit backend/.env:
    USE_CELERY=true
    REDIS_URL=redis://localhost:6379/0

  Start the Celery worker (in a separate terminal):

    cd backend
    .\venv\Scripts\activate        (Windows)
    source venv/bin/activate       (Mac/Linux)

    celery -A app.tasks.celery_app worker --loglevel=info --pool=solo

  With Celery enabled, campaign dispatch runs in the background
  without blocking the API response.

--------------------------------------------------------------------------------
  WHERE DATA IS STORED
--------------------------------------------------------------------------------

  backend/autorespond.db    -- SQLite file (all app data when using SQLite)
  backend/uploads/          -- Raw uploaded PDF/DOCX/TXT product files
  backend/chroma_db/        -- ChromaDB vector embeddings (RAG knowledge base)

  Database tables:
    users                   -- Login accounts and roles
    campaigns               -- Campaign records (status: draft/approved/launched)
    documents               -- Uploaded file metadata and chunk counts
    customers               -- Customer contacts, consent, opt-out flag
    messages                -- Every inbound and outbound message
    leads                   -- Interested / callback / complaint leads
    audit_logs              -- Campaign approval and launch history

================================================================================
  SECTION 3 — LLM (AI) CONFIGURATION
================================================================================

By default LLM_PROVIDER=mock which uses rule-based logic and template responses.
Switch to a real provider to get actual AI-generated campaign copy and replies.

The LLM config is in backend/.env:
  LLM_PROVIDER=mock           <-- change to: gemini | azure | cohere
  EMBEDDING_PROVIDER=mock     <-- change to: gemini | azure | cohere

IMPORTANT: After installing real LLM providers run:
  pip install -r requirements-llm.txt

IMPORTANT: If you change EMBEDDING_PROVIDER you MUST re-upload all product
           documents. Vectors from different providers are incompatible.

--------------------------------------------------------------------------------
  OPTION A — Google Gemini (Easiest to set up)
--------------------------------------------------------------------------------

  Get API key:
    1. Go to https://aistudio.google.com/app/apikey
    2. Click "Create API key"
    3. Copy the key

  Edit backend/.env:
    LLM_PROVIDER=gemini
    EMBEDDING_PROVIDER=gemini
    GOOGLE_API_KEY=your_google_api_key_here
    GEMINI_MODEL=gemini-2.0-flash
    GEMINI_EMBEDDING_MODEL=models/text-embedding-004

  Install provider package:
    pip install -r requirements-llm.txt

  Restart backend:
    uvicorn app.main:app --reload --port 8000

--------------------------------------------------------------------------------
  OPTION B — Azure OpenAI
--------------------------------------------------------------------------------

  You need an Azure subscription with Azure OpenAI resource deployed.

  Edit backend/.env:
    LLM_PROVIDER=azure
    EMBEDDING_PROVIDER=azure
    AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
    AZURE_OPENAI_API_KEY=your_azure_openai_key_here
    AZURE_OPENAI_API_VERSION=2024-06-01
    AZURE_OPENAI_DEPLOYMENT=your_chat_deployment_name
    AZURE_OPENAI_EMBEDDING_DEPLOYMENT=your_embedding_deployment_name

  Install provider package:
    pip install -r requirements-llm.txt

  Restart backend:
    uvicorn app.main:app --reload --port 8000

  Where to find Azure values:
    - Endpoint and Key: Azure Portal -> Azure OpenAI -> Keys and Endpoint
    - Deployment names: Azure OpenAI Studio -> Deployments

--------------------------------------------------------------------------------
  OPTION C — Cohere
--------------------------------------------------------------------------------

  Get API key:
    1. Go to https://dashboard.cohere.com/api-keys
    2. Create a new API key

  Edit backend/.env:
    LLM_PROVIDER=cohere
    EMBEDDING_PROVIDER=cohere
    COHERE_API_KEY=your_cohere_api_key_here
    COHERE_MODEL=command-r
    COHERE_EMBEDDING_MODEL=embed-english-v3.0

  Install provider package:
    pip install -r requirements-llm.txt

  Restart backend:
    uvicorn app.main:app --reload --port 8000

--------------------------------------------------------------------------------
  FALLBACK PROVIDER (Optional)
--------------------------------------------------------------------------------

  Set a second LLM provider. If the primary fails, it retries with the fallback:

    LLM_PROVIDER=gemini
    LLM_FALLBACK_PROVIDER=cohere

  The fallback only applies to LLM calls, not embeddings.

================================================================================
  SECTION 4 — MESSAGING CONFIGURATION
================================================================================

By default MESSAGING_MODE=mock — all sends print to the backend terminal.
Set MESSAGING_MODE=live to send real messages.

--------------------------------------------------------------------------------
  EMAIL (SMTP)
--------------------------------------------------------------------------------

  Edit backend/.env:
    MESSAGING_MODE=live
    SMTP_HOST=smtp.gmail.com
    SMTP_PORT=587
    SMTP_USER=your_email@gmail.com
    SMTP_PASSWORD=your_app_password_here
    EMAIL_FROM=your_email@gmail.com

  For Gmail: use an App Password (not your regular password).
    Go to: Google Account -> Security -> 2-Step Verification -> App Passwords

  For other providers, replace SMTP_HOST and SMTP_PORT accordingly:
    Outlook/Office365:  SMTP_HOST=smtp.office365.com   SMTP_PORT=587
    Yahoo:              SMTP_HOST=smtp.mail.yahoo.com   SMTP_PORT=587
    SendGrid:           SMTP_HOST=smtp.sendgrid.net     SMTP_PORT=587

--------------------------------------------------------------------------------
  SMS (Twilio)
--------------------------------------------------------------------------------

  Get Twilio credentials:
    1. Sign up at https://www.twilio.com
    2. Go to Console Dashboard
    3. Copy Account SID and Auth Token
    4. Buy a phone number (or use trial number)

  Edit backend/.env:
    MESSAGING_MODE=live
    TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    TWILIO_AUTH_TOKEN=your_auth_token_here
    TWILIO_SMS_FROM=+1234567890      <-- your Twilio number

  Install Twilio package:
    pip install -r requirements-llm.txt

--------------------------------------------------------------------------------
  WHATSAPP (Twilio WhatsApp API)
--------------------------------------------------------------------------------

  Requires a Twilio WhatsApp-enabled number or Twilio Sandbox.

  For Sandbox (testing):
    1. Go to Twilio Console -> Messaging -> Try it out -> Send a WhatsApp message
    2. Follow sandbox join instructions
    3. Use sandbox number: whatsapp:+14155238886

  Edit backend/.env:
    MESSAGING_MODE=live
    TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    TWILIO_AUTH_TOKEN=your_auth_token_here
    TWILIO_WHATSAPP_FROM=whatsapp:+14155238886

  NOTE: For production WhatsApp campaigns, provider-approved message templates
        are required by WhatsApp Business API policies.

--------------------------------------------------------------------------------
  INBOUND WEBHOOKS (Receiving Replies from Customers)
--------------------------------------------------------------------------------

  The app listens for inbound messages at:
    POST /api/webhooks/email
    POST /api/webhooks/whatsapp
    POST /api/webhooks/sms

  For local development, expose your backend using ngrok:
    ngrok http 8000
    -- Copy the https://xxxx.ngrok.io URL

  Set as your provider's webhook URL:
    Twilio SMS/WhatsApp: https://xxxx.ngrok.io/api/webhooks/sms
                         https://xxxx.ngrok.io/api/webhooks/whatsapp

  Secure webhooks with a secret:
    Edit backend/.env:
      WEBHOOK_SECRET=your_random_secret_string

    Add header X-Webhook-Secret: your_random_secret_string in provider settings.

================================================================================
  SECTION 5 — SECURITY CONFIGURATION
================================================================================

--------------------------------------------------------------------------------
  JWT Secret Key
--------------------------------------------------------------------------------

  IMPORTANT: Change this before going to production!

  Edit backend/.env:
    SECRET_KEY=change-me-to-a-long-random-string

  Generate a secure key:
    Python:  python -c "import secrets; print(secrets.token_hex(32))"
    Or use any long random string (32+ characters)

  Token expiry (default 480 minutes = 8 hours):
    TOKEN_EXPIRE_MINUTES=480

--------------------------------------------------------------------------------
  CORS (Frontend Origin)
--------------------------------------------------------------------------------

  If your frontend runs on a different port or domain, update:
    FRONTEND_ORIGIN=http://localhost:5173

  For production:
    FRONTEND_ORIGIN=https://your-frontend-domain.com

================================================================================
  SECTION 6 — ENVIRONMENT VARIABLE REFERENCE (backend/.env)
================================================================================

  KEY                               DEFAULT         DESCRIPTION
  --------------------------------  --------------  ----------------------------
  SECRET_KEY                        dev-secret...   JWT signing key (CHANGE THIS)
  TOKEN_EXPIRE_MINUTES              480             JWT token expiry in minutes
  FRONTEND_ORIGIN                   localhost:5173  Allowed CORS origin

  DATABASE_URL                      sqlite:///...   Database connection string
  USE_CELERY                        false           Enable Celery async workers
  REDIS_URL                         localhost:6379  Redis URL (for Celery)

  LLM_PROVIDER                      mock            mock | gemini | azure | cohere
  LLM_FALLBACK_PROVIDER             (empty)         Retry provider on LLM failure
  EMBEDDING_PROVIDER                mock            mock | gemini | azure | cohere

  GOOGLE_API_KEY                    (empty)         Gemini API key
  GEMINI_MODEL                      gemini-2.0-...  Gemini chat model name
  GEMINI_EMBEDDING_MODEL            text-emb-004    Gemini embedding model name

  AZURE_OPENAI_ENDPOINT             (empty)         Azure OpenAI resource URL
  AZURE_OPENAI_API_KEY              (empty)         Azure OpenAI API key
  AZURE_OPENAI_API_VERSION          2024-06-01      Azure API version
  AZURE_OPENAI_DEPLOYMENT           (empty)         Chat deployment name
  AZURE_OPENAI_EMBEDDING_DEPLOYMENT (empty)         Embedding deployment name

  COHERE_API_KEY                    (empty)         Cohere API key
  COHERE_MODEL                      command-r       Cohere chat model
  COHERE_EMBEDDING_MODEL            embed-eng-v3    Cohere embedding model

  MESSAGING_MODE                    mock            mock | live
  SMTP_HOST                         (empty)         SMTP server hostname
  SMTP_PORT                         587             SMTP server port
  SMTP_USER                         (empty)         SMTP login username
  SMTP_PASSWORD                     (empty)         SMTP login password
  EMAIL_FROM                        (empty)         Sender email address

  TWILIO_ACCOUNT_SID                (empty)         Twilio Account SID
  TWILIO_AUTH_TOKEN                 (empty)         Twilio Auth Token
  TWILIO_SMS_FROM                   (empty)         Twilio SMS phone number
  TWILIO_WHATSAPP_FROM              whatsapp:+14... Twilio WhatsApp number

  WEBHOOK_SECRET                    (empty)         Secret for inbound webhooks
  UPLOAD_DIR                        backend/uploads Folder for uploaded files
  CHROMA_DIR                        backend/chroma  Folder for ChromaDB vectors

================================================================================
  SECTION 7 — CONFIGURATION PROFILES (COMMON SETUPS)
================================================================================

--- Profile 1: Local Development (fully offline, default) ---

  DATABASE_URL=sqlite:///./autorespond.db
  LLM_PROVIDER=mock
  EMBEDDING_PROVIDER=mock
  MESSAGING_MODE=mock
  USE_CELERY=false

--- Profile 2: Local Dev with real AI (Gemini) ---

  DATABASE_URL=sqlite:///./autorespond.db
  LLM_PROVIDER=gemini
  EMBEDDING_PROVIDER=gemini
  GOOGLE_API_KEY=your_key_here
  MESSAGING_MODE=mock
  USE_CELERY=false

--- Profile 3: Staging / Production ---

  DATABASE_URL=postgresql+psycopg2://user:pass@host:5432/autorespond
  LLM_PROVIDER=gemini
  LLM_FALLBACK_PROVIDER=cohere
  EMBEDDING_PROVIDER=gemini
  GOOGLE_API_KEY=your_key_here
  COHERE_API_KEY=your_key_here
  MESSAGING_MODE=live
  SMTP_HOST=smtp.sendgrid.net
  SMTP_PORT=587
  SMTP_USER=apikey
  SMTP_PASSWORD=your_sendgrid_key
  EMAIL_FROM=no-reply@yourdomain.com
  TWILIO_ACCOUNT_SID=ACxxxxxxxx
  TWILIO_AUTH_TOKEN=xxxxxxxx
  TWILIO_SMS_FROM=+1234567890
  TWILIO_WHATSAPP_FROM=whatsapp:+1234567890
  SECRET_KEY=your_long_random_secret_here
  FRONTEND_ORIGIN=https://your-frontend.com
  USE_CELERY=true
  REDIS_URL=redis://your-redis-host:6379/0

================================================================================
  SECTION 8 — TROUBLESHOOTING
================================================================================

  PROBLEM: Backend fails to start — "No module named X"
  SOLUTION:
    pip install -r requirements.txt
    If using real LLM/Twilio: pip install -r requirements-llm.txt

  PROBLEM: Database error on startup
  SOLUTION:
    For SQLite — check write permissions in backend/ folder
    For PostgreSQL — make sure Docker container is running:
      docker compose up -d
      docker compose ps   (should show db as "Up")

  PROBLEM: LLM returns None / no AI response
  SOLUTION:
    Check LLM_PROVIDER is set correctly in .env
    Check the API key is valid and has quota
    Set LLM_FALLBACK_PROVIDER as a backup
    Check backend terminal for error messages like "[llm] gemini failed: ..."

  PROBLEM: Switched EMBEDDING_PROVIDER, RAG returns no results
  SOLUTION:
    Vectors from different providers are incompatible.
    Re-upload all product documents after changing EMBEDDING_PROVIDER.
    Old ChromaDB collections are stored at backend/chroma_db/ — you can
    safely delete that folder and re-upload documents.

  PROBLEM: Emails / SMS not sending
  SOLUTION:
    Check MESSAGING_MODE=live in .env (not "mock")
    Check SMTP credentials — for Gmail use App Password not regular password
    Check Twilio Account SID and Auth Token are correct
    Test webhook endpoint with curl or Postman first

  PROBLEM: Campaign won't launch — "No eligible customers"
  SOLUTION:
    Go to Customers page and add customers with:
      - consent checkbox TICKED
      - unsubscribed = false (not opted out)

  PROBLEM: Campaign won't launch — "Approve content first"
  SOLUTION:
    Go to Campaigns -> Generate content -> then click Approve
    If you edited content after approving, you must approve again

  PROBLEM: Celery tasks not running
  SOLUTION:
    Make sure Redis is running: docker compose up -d
    Make sure Celery worker is running (see Section 2, Option C)
    Check USE_CELERY=true in .env

  PROBLEM: Frontend cannot connect to backend (CORS error)
  SOLUTION:
    Check FRONTEND_ORIGIN in .env matches your frontend URL exactly
    Default: FRONTEND_ORIGIN=http://localhost:5173

================================================================================
  SECTION 9 — PRODUCTION CHECKLIST
================================================================================

  Before deploying to production:

  [ ] Change SECRET_KEY to a long random string (32+ characters)
  [ ] Switch DATABASE_URL to PostgreSQL
  [ ] Set LLM_PROVIDER to a real provider (gemini / azure / cohere)
  [ ] Set MESSAGING_MODE=live and configure SMTP + Twilio
  [ ] Set FRONTEND_ORIGIN to your actual frontend domain
  [ ] Set WEBHOOK_SECRET and configure it in Twilio dashboard
  [ ] Set USE_CELERY=true for async campaign sending
  [ ] Replace SQLAlchemy create_all with Alembic migrations
  [ ] Move JWT token from localStorage to httpOnly cookies
  [ ] Add HTTPS / TLS termination (via nginx or cloud load balancer)
  [ ] Add OCR support for scanned (image-only) PDFs if needed
  [ ] Do NOT commit .env to git — add it to .gitignore

================================================================================
  SECTION 10 — PROJECT FOLDER STRUCTURE
================================================================================

  autorespond-ai/
  |
  +-- backend/
  |   +-- app/
  |   |   +-- api/            FastAPI route handlers (8 modules)
  |   |   +-- agents/         Intent, sentiment, campaign, response agents
  |   |   +-- workflows/      LangGraph: response_graph, campaign_graph
  |   |   +-- rag/            ingestion, embeddings, retrieval (ChromaDB)
  |   |   +-- services/       LLM, email, SMS, WhatsApp, campaign runner
  |   |   +-- database/       SQLAlchemy models and DB connection
  |   |   +-- config.py       All settings loaded from .env
  |   |   +-- security.py     JWT auth, password hashing
  |   |   +-- main.py         FastAPI app entry point
  |   +-- .env                Your local config (DO NOT COMMIT)
  |   +-- .env.example        Config template
  |   +-- requirements.txt    Core dependencies
  |   +-- requirements-llm.txt  LLM provider packages
  |   +-- uploads/            Uploaded product files (auto-created)
  |   +-- chroma_db/          ChromaDB vector store (auto-created)
  |   +-- autorespond.db      SQLite database (auto-created)
  |
  +-- frontend/
  |   +-- src/
  |   |   +-- pages/          Dashboard, Campaigns, Customers, Inbox, Leads, etc.
  |   |   +-- components/     Layout, Bars
  |   |   +-- services/api.js  Axios API calls to backend
  |   +-- .env                Frontend config (VITE_API_URL)
  |   +-- package.json        Node dependencies
  |
  +-- docker-compose.yml      PostgreSQL + Redis (optional)
  +-- README.md               Original project readme
  +-- README_SETUP.txt        This file

================================================================================
  END OF SETUP GUIDE
================================================================================
