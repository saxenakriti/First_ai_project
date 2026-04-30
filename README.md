# First_ai_project
Health-Agent-ai
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

