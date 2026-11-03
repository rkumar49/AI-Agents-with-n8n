
# 1. Personal Voice Assistant

A personal voice assistant built with **n8n** and **Vapi**.

## 🎥 Demo Video

[▶️ Watch the demo]

https://github.com/user-attachments/assets/1ce68cab-c399-4c9c-9356-447ace1662fb



## What It Does

- Listens to voice commands
- Responds with AI-powered voice
- Built entirely with n8n workflows + Vapi

## Tech Stack

- n8n
- Vapi



# 2. ✈️ AI Travel Agent Workflow (n8n)

This repository contains an advanced, multi-agent AI workflow built in [n8n](https://n8n.io/). It acts as an automated travel agent that takes a user's natural language request, extracts parameters, searches for real-time flights, hotels, and activities, and delivers a beautifully formatted HTML travel itinerary straight to their email.

---
## 🎥 Demo Video

[▶️ Watch the demo]

https://github.com/user-attachments/assets/39f5d2fb-1eaf-4f28-a31e-6fdd38bccfc5


## 🏗️ How It Works (Architecture)

The workflow uses a chain of three specialized AI Agents to ensure high-quality data processing and output:

1.  **Extract User Request Agent (Data Parsing)**
    *   Receives the raw chat message (e.g., *"I want to plan a 5-day trip to Tokyo for 2 adults next month with a budget of $3000 from Frankfurt"*).
    *   Uses an OpenAI model and a Structured Output Parser to convert messy human language into a strict JSON schema (extracting dates, origin, destination, budget, and traveler demographics).
2.  **Planner Agent (Research & Tool Usage)**
    *   Takes the structured JSON and acts as an expert travel coordinator.
    *   Uses four external tools to fetch real-time data:
        *   **Flights:** Uses SerpApi (Google Flights) to find up to 3 flight options.
        *   **Accommodations:** Uses SerpApi (Google Hotels) to find hotels matching the budget/dates.
        *   **Accommodation Details:** Fetches specific hotel websites and amenities using property tokens.
        *   **Activities:** Uses Tavily Search to find the top 5 activities based on the destination and season.
    *   Compiles all this real-world data into a unified JSON travel plan.
3.  **Email Agent (HTML Generation)**
    *   Takes the comprehensive travel plan JSON.
    *   Uses another OpenAI model to generate a fully responsive, inline-CSS styled HTML email.
    *   Sends the final email via the Gmail node.

## 🛠️ Prerequisites

To run this workflow, you will need accounts and API keys for the following services:
*   **OpenAI:** For the language models (uses `gpt-4.1` / `gpt-4.1-mini`).
*   **SerpApi:** For fetching real-time Google Flights and Google Hotels data.
*   **Tavily:** For searching activities and attractions.
*   **Gmail:** To send the final email.

## 🚀 Setup Instructions

1.  **Import the Workflow:** Download the `.json` file from this repo and import it into your n8n instance.
2.  **Configure Credentials:**
    *   Add your **OpenAI API** credential to the three OpenAI Chat Model nodes.
    *   Add your **SerpApi** credential to the `Flights`, `Accommodations`, and `Accommodation Details` nodes.
    *   Add your **Tavily** credential to the `Activities` node.
    *   Add your **Gmail OAuth2** credential to the `Send a message` node.
3.  **Update the Email Address:** Open the `Send a message` (Gmail) node and change the `Send To` field from the placeholder email to your own email address.
4.  **Activate and Test:** Send a chat message through the n8n chat trigger to test the flow.

---

## 🐛 Troubleshooting & Lessons Learned (Bug Fixes)

During the development of this workflow, we encountered and solved several common AI/API issues. If you are building similar workflows, keep these in mind:

### 1. Context Window Overflow (Token Limits)
*   **Problem:** The `Accommodation Details` tool returned massive amounts of raw JSON, causing the workflow to crash with a `ContextOverflowError` (exceeding 128k tokens).
*   **The Fix:** 
    *   Switched the Planner Agent's model to `gpt-4.1` (which has a 1M+ token limit).
    *   **Crucial:** Used the **"Optimize Response"** feature in the HTTP Request node (`Accommodations`) to only extract the `properties` field from the SerpApi response, stripping out useless metadata and saving thousands of tokens.

### 2. SerpApi "Invalid API Key" Errors
*   **Problem:** The `Flights` node worked, but the `Accommodations` nodes failed with `401 Unauthorized`. 
*   **The Fix:** The workflow uses two different types of authentication. The `Flights` node uses a predefined SerpApi credential, while the HTTP nodes use a generic `Query Auth` credential. We had to ensure the *actual API key string* was pasted into the `Value` field of the generic credential, not just the parameter name.

### 3. AI Hallucinations & Bad Parameters
*   **Problem:** When API keys failed, the AI "hallucinated" fake flights to satisfy the JSON schema. Also, the AI was confusing parameter fields (e.g., putting the word "hotel" into the `check_in_date` field).
*   **The Fix:** 
    *   Fixed all API keys so real data flows.
    *   In the `Accommodations` node, we changed the generic `$fromAI('parameters1_Value')` to explicit variable names like `$fromAI('check_in_date', 'The check-in date in YYYY-MM-DD format')`. This gave the AI strict instructions on exactly what data belongs in which field.

### 4. Truncated/Incomplete Emails
*   **Problem:** The Email Agent started generating the HTML email but cut off abruptly after the greeting. 
*   **The Fix:** The AI ran out of output tokens while writing complex inline CSS. We fixed this by:
    *   Going to the `OpenAI Chat Model2` node (connected to the Email Agent) and setting the **Max Tokens** option to `4000`+.
    *   Updating the Email Agent's System Message to explicitly say: *"Keep the HTML code as lean and concise as possible... Do not use overly complex CSS styling."*





