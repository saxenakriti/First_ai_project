# First_ai_project
Health-Agent-ai

#Setting Project
gcloud config set project health-agent-491301

# enabling API
gcloud services enable \
  run.googleapis.com \
  artifactregistry.googleapis.com \
  cloudbuild.googleapis.com \
  aiplatform.googleapis.com \
  compute.googleapis.com
  
# Creating directories
cd && mkdir health_agent && cd health_agent

#Workspace initialize
cloudshell open-workspace ~/health_agent

#Create requirements
cloudshell edit requirements.txt
#Adding in requirement
google-adk==1.14.0
langchain-community==0.3.27
wikipedia==1.4.0

#VIRTUAL ENVIRONMENT
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt

#SETTING UP VARIABLES
PROJECT_ID=health-agent-491301
PROJECT_NUMBER=452616494009

#CREATING ENV FILE WITH VARIABLES
cat <<EOF > .env
PROJECT_ID=$PROJECT_ID
PROJECT_NUMBER=$PROJECT_NUMBER
SA_NAME=$SA_NAME
SERVICE_ACCOUNT=${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com
MODEL="gemini-2.5-flash"
EOF

#CREATE INIT.PY FILE
cloudshell edit __init__.py
from . import agent

#CREATE AGENT.PY
cloudshell edit agent.py
import os
import logging
from dotenv import load_dotenv

from google.adk import Agent
from google.adk.tools.langchain_tool import LangchainTool

from langchain_community.tools import WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper

# --- Logging ---
logging.basicConfig(level=logging.INFO)

# --- ENV SAFE LOAD ---
try:
    load_dotenv()
except Exception:
    logging.warning("dotenv not loaded (ok in Cloud Run)")

model_name = os.getenv("MODEL", "gemini-2.0-flash")

# --- Wikipedia Tool (SAFE) ---
try:
    wiki = WikipediaQueryRun(
        api_wrapper=WikipediaAPIWrapper(
            top_k_results=2,
            doc_content_chars_max=500
        )
    )
    wiki_tool = LangchainTool(tool=wiki)
    tools = [wiki_tool]
except Exception:
    logging.warning("Wikipedia tool failed, running without it")
    tools = []

# --- SINGLE INTELLIGENT AGENT ---
root_agent = Agent(
    name="health_assistant",
    model=model_name,
    description="Advanced AI health assistant",
    instruction="""
You are an intelligent medical assistant (educational use only).

--------------------------------
UNDERSTAND USER INPUT STRICTLY
--------------------------------
1. GREETING (hi, hello, hey):
Politely refuse off topic conversation 
Reply in EXACTLY 2 lines:
Hi! I analyze symptoms and suggest possible conditions.
You can also ask about any disease.

2. DISEASE QUERY:
If user asks about a disease, respond in 3-4 lines ONLY:
- What it is
- Main symptoms
- When to worry

3. SYMPTOMS INPUT:
Normalize: bukhar = fever, saans problem = breathing issue, chest tightness/pressure = chest pain.
Filtering: Ignore random words. Only keep real symptoms.
If nothing valid remains → ask again.

4. SEVERE CASE:
If symptoms include: chest pain, breathing issue, or unconsciousness.
FIRST LINE: ⚠️ Seek immediate medical attention.

--------------------------------
OUTPUT FORMAT (STRICT)
--------------------------------
Possible Conditions:

1. [Disease Name]
   - Likelihood: XX%
   - Severity: Low / Medium / High
   - Reason: short
   - Suggested Tests: tests

(Repeat for top 3 conditions. Total likelihood MUST be 100%)

--------------------------------
RULES
--------------------------------
- Keep output clean. No extra explanation.
- ALWAYS END WITH: This is not medical advice. Consult a doctor.
""",
    tools=tools
)

