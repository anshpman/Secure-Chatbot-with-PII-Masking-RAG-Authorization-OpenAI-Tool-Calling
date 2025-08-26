# Secure Chatbot with PII Masking, RAG Authorization & OpenAI Tool Calling

This repository contains the code for an advanced Python chatbot designed to securely collect user information (Name, Age, Email) by leveraging a multi-layered architecture that includes PII masking, RAG-based authorization, and OpenAI's tool-calling feature.

The application is built as a Jupyter Notebook for clear, cell-by-cell execution and demonstration of its modular components. It successfully fulfills the "all at once" or "one-by-one" data collection requirement through a robust, stateful design where the Python code remains in control of the logic.


## System Architecture

The chatbot operates on a modular, secure-by-design architecture. The core principle is that our local Python code remains in control of the state and logic, while leveraging external AI services as specialized tools.

![Alt Text](https://github.com/anshpman/Secure-Chatbot-with-PII-Masking-RAG-Authorization-OpenAI-Tool-Calling/blob/9f5f3437366147ce95e9e8e7565d3322d20a5755/System%20Design.png)

The data flow is executed in a specific order to maximize security and efficiency:
1.  **PII Masking & Extraction:** User input is immediately processed by a PII module using Microsoft Presidio to find and mask sensitive data. The real, unmasked data is stored in a secure local dictionary.
2.  **RAG Authorization:** If a name is extracted, it is immediately checked against a pre-loaded vector store of authorized users. Unauthorized users are rejected, and the process stops.
3.  **Conversational AI:** Only after a user is authorized does the application interact with the OpenAI API. The AI's role is to manage the conversational flow and ask for any missing information. All communication with the AI uses the masked text.
4.  **Tool Calling & Execution:** Once all required information is collected, our Python code makes a final, targeted API call to force the AI to trigger our local `check_eligibility` function via its tool-calling mechanism. The function is executed with the real, unmasked data from our secure local vault.
## Setup & Usage

Follow these steps to get the chatbot running on your local machine.

### Prerequisites
* Python 3.9+
* Jupyter Notebook or JupyterLab

### Clone the Repository
Open your terminal and clone this repository to your local machine:
```bash
git clone <https://github.com/anshpman/Secure-Chatbot-with-PII-Masking-RAG-Authorization-OpenAI-Tool-Calling/edit/main/README.md>
cd <Secure-Chatbot-with-PII-Masking-RAG-Authorization-OpenAI-Tool-Calling>
```

There are two items to configure before running the main application:

**Authorized Users** : The `authorized_users.txt` file is created automatically by the first cell. You can edit this file to change or add to the list of authorized names (one name per line).

**OpenAI API Key**: Open the final cell of the main.ipynb notebook and replace the placeholder text `"YOUR_OPENAI_API_KEY"` with your actual OpenAI API key.
## Architectural Decisions & Learnings

#### Q: Why were custom Presidio recognizers necessary?
**A:** During testing, I found that Presidio's default recognizers had limitations for this specific task. The built-in `AGE` recognizer was unreliable for simple, conversational inputs (e.g., "I am 19"), and the default `EMAIL` recognizer was too broad. I engineered two custom, regex-based recognizers to ensure the bot could reliably capture ages and *only* `@gmail.com` addresses, significantly improving the system's accuracy and robustness.

#### Q: Why is the application's main loop controlled by Python instead of being fully managed by the AI?
**A:** While an AI-driven approach using `tool_choice='auto'` is possible, early prototypes showed it could be unreliable. The AI would sometimes get stuck in conversational loops or fail to recognize when it had collected all the necessary information. I pivoted to a more robust, stateful design where the Python code explicitly tracks the collected data in a dictionary. This makes the application's logic predictable and guarantees it can handle both "all-at-once" and "one-by-one" user inputs correctly.

#### Q: What is the primary limitation of this system?
**A:** The system's main vulnerability is its dependence on Presidio's built-in `PERSON` recognizer. If the model fails to identify a less common or uncapitalized name, the RAG authorization check is bypassed. This highlights a real-world challenge of building systems on top of imperfect AI components. A future production-ready version would include a fallback mechanism, such as explicitly asking the user to confirm their name if one isn't detected, to ensure the security gate is never missed.

#### Q: Why use a complex RAG system for authorization instead of just a simple database lookup?
**A:** While a simple database lookup is faster and more direct for checking an exact name, the task  specifically required the implementation of a RAG pipeline with semantic search. 

#### Q: What is the specific role of the OpenAI Tool Calling feature in the final, robust version of the code?
**A:** In the final architecture, my Python code is in complete control of the application's state. It knows when all the required information (name, age, email) has been collected. The OpenAI Tool Calling feature serves a very specific and crucial purpose: it acts as the formal trigger to satisfy the project's core requirement. Even though my local code knows it's time to run the eligibility check, it makes one final, targeted API call with tool_choice="required". This forces the AI to formally "request" the execution of the check_eligibility function. This design is the best of both worlds: it provides the reliability of a Python-controlled state machine while correctly using the tool-calling feature as the final invocation mechanism, just as the prompt required.
