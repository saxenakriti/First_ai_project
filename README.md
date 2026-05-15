# First_ai_project
Health-Agent-ai

#Setting Project
gcloud config set project health-agent-491301

# ENABLE API
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

<img width="1036" height="558" alt="image" src="https://github.com/user-attachments/assets/19dd6bbf-efce-463f-b62a-5ebca235a565" />
<img width="1069" height="591" alt="image" src="https://github.com/user-attachments/assets/5441ecad-8966-48e3-a6d5-0f89a8e2af1c" />
<img width="1048" height="609" alt="image" src="https://github.com/user-attachments/assets/60d154bc-4ac2-4657-9a25-124ca1f46984" />
<img width="1076" height="582" alt="image" src="https://github.com/user-attachments/assets/56fa1230-56d6-4809-ab25-224dcfd38c4c" />
<img width="1057" height="637" alt="image" src="https://github.com/user-attachments/assets/5c61f1f4-bd7c-4c1b-86c3-329176cca65e" />
<img width="1027" height="586" alt="image" src="https://github.com/user-attachments/assets/3d580ef2-f524-49e7-ba3e-7303bf2704ee" />
<img width="1104" height="624" alt="image" src="https://github.com/user-attachments/assets/2817bd68-bdc0-46ef-b1aa-e8371ff9a4e3" />
<img width="1051" height="638" alt="image" src="https://github.com/user-attachments/assets/8fa21130-74a8-446b-bb0b-dabe6326f42d" />
<img width="1074" height="610" alt="image" src="https://github.com/user-attachments/assets/8efe6097-0729-4844-b248-9307b12b8432" />
<img width="1065" height="619" alt="image" src="https://github.com/user-attachments/assets/8c090fb6-e0e0-4b2c-96f0-a719ee5f6c1b" />
<img width="1046" height="565" alt="image" src="https://github.com/user-attachments/assets/1b065133-f76e-4804-9031-045dbd4a777a" />
<img width="1039" height="581" alt="image" src="https://github.com/user-attachments/assets/5da728f0-4ca3-4d5a-bf52-ac3e32363c6f" />
<img width="1055" height="561" alt="image" src="https://github.com/user-attachments/assets/37b9e22a-1b34-4a91-a4a5-301ee1c9a41a" />
<img width="1016" height="549" alt="image" src="https://github.com/user-attachments/assets/d6338ba0-d126-412d-810b-54545ddff75f" />
<img width="1044" height="575" alt="image" src="https://github.com/user-attachments/assets/76ce96c8-d12f-428f-9515-6238af7c95fa" />
<img width="1051" height="565" alt="image" src="https://github.com/user-attachments/assets/b99bc0a1-ed9b-4124-bbb7-4cc6764f8a67" />
<img width="1050" height="558" alt="image" src="https://github.com/user-attachments/assets/dad4ca54-dd86-4369-9eb6-7bde9a61f0f0" />
<img width="1030" height="582" alt="image" src="https://github.com/user-attachments/assets/0886dea3-68ac-40e7-8763-752843794ea5" />
<img width="1044" height="567" alt="image" src="https://github.com/user-attachments/assets/9bd8e8c5-3b8f-4d6c-9531-e7275a929aa3" />
<img width="1011" height="540" alt="image" src="https://github.com/user-attachments/assets/824deeae-24af-46f5-a50e-da39d8556bc0" />
