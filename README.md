# Introduction
In today’s digital environment, responding to large volumes of emails can be time-consuming.
People and businesses often struggle to reply promptly, especially when messages are repetitive or simple in nature.
This project introduces a no-code AI automation workflow that automatically replies to new emails using intelligent text generation from a local AI model (Ollama).
The system is designed to:
- Detect new incoming emails.
- Generate polite, context-aware replies automatically.
- Send responses back instantly
  
This project provides the perfect foundation for beginners to understand workflow automation, AI prompt generation, and event-driven logic in n8n.

# Statement of the Problem
Managing email communication manually can lead to:
- Delayed Responses: Human response time is inconsistent.
- Repetitive Tasks: Many messages require similar replies.
- Lost Opportunities: Important messages get buried or forgotten.

By automating email replies with n8n and Ollama, these issues are solved through real-time, intelligent, and logged communication.

# Project Objectives

i. Automate the process of reading and replying to new emails.

ii. Use Ollama’s local AI model to generate polite, natural responses.

iv. Ensure professional workflow sequencing using trigger → action → condition → transform → output.

v. Build a reusable system that can be scaled for support desks, business inquiries, or customer response bots.

# Technology Stack

| **Category**        | **Tool / Technology**   | **Purpose**                                         |
|----------------------|------------------------|-----------------------------------------------------|
| Workflow Engine      | n8n                    | Orchestrates all automation steps.                  |
| Email Service        | Gmail API              | Triggers workflow & sends email replies.            |
| AI Engine            | Ollama (Llama3)        | Generates intelligent email responses locally.       |
| Communication        | HTTP Request Node      | Connects n8n to Ollama’s local API.                 |

# Workflow Architecture

| **Node Type** | **Function**                                      | **Tool Used**          |
|----------------|---------------------------------------------------|------------------------|
| Trigger        | Detect new emails and start automation.           | Gmail Trigger          |
| Action         | Generate AI reply based on email body.            | HTTP Request → Ollama  |
| Conditional    | Check if email needs a reply or is filtered.      | If Node                |
| Transform      | Format and personalize message content.           | Set Node               |
| Output         | Send reply via Gmail.                             | Gmail Node             |

# Step-by-Step Implementation
## Step 1 – Trigger Setup (Gmail Trigger Node)

### Goal: Start the workflow whenever a new email is received.

Add Gmail Trigger Node
- On your n8n canvas, click ➕ Add Node → search “Gmail.”
- Choose Gmail Trigger; On message recieved
  <img width="853" height="606" alt="image" src="https://github.com/user-attachments/assets/4f6ae08d-7fdc-4e22-b300-94603d00801c" />

 

Connect Your Gmail Account
- Click Create New Credential → OAuth2.
  <img width="934" height="479" alt="image" src="https://github.com/user-attachments/assets/7602b0c1-7018-4226-b3dd-b93ba330017f" />

- Sign in with your Gmail account.
- Allow n8n permission to read emails.
  <img width="925" height="433" alt="image" src="https://github.com/user-attachments/assets/75ab9fe1-e0e2-4bf2-af70-6ab04cc88b59" />


Configure Trigger Settings
- Under “Trigger On,” select New Email.
  <img width="888" height="299" alt="image" src="https://github.com/user-attachments/assets/b9737a79-83e4-4182-9706-9258c646defb" />

- Save and test the node by sending a sample email.
- When executed, n8n displays email data like sender, subject, and message body.
  <img width="906" height="336" alt="image" src="https://github.com/user-attachments/assets/99ed40d5-30ca-408a-a421-f24d67075358" />

## Step 2 – Action Setup (HTTP Request → Ollama API)
### Goal: Send the email content to Ollama and generate an intelligent reply.

Add HTTP Request Node
- Click ➕ Add Node → HTTP Request.
  <img width="921" height="370" alt="image" src="https://github.com/user-attachments/assets/5b682917-4f25-4993-b7a2-814f80883285" />

- Rename it to AI Response Generator.


Configure the API Request
- Method: POST
- URL: http://185.193.17.27:11434/api/chat
- Send Body: Turn on
- Body Content: JSON
- Specify Body: RAW (JSON)
  <img width="873" height="263" alt="image" src="https://github.com/user-attachments/assets/17476dbb-5d73-4047-a4ff-899d7d16e08e" />

 

iii.	Under JSON, paste the below
```
{
  "model": "llama3",
  "stream": false,
  "messages": [
    {
      "role": "system",
      "content": "You are Philip’s AI Email Assistant. Read each new incoming email carefully and generate a concise, polite, and context-aware professional reply. The reply must always sound natural and human. You are assisting Philip with managing all incoming messages. Output only a JSON object with the following keys: reply, intent, confidence. The reply must never include greetings like 'Dear Sender' or signatures unless explicitly required by the message."
    },
    {
      "role": "user",
      "content": "Incoming Email:\nFrom: {{ $json.payload?.headers?.from || $json.from || 'Sender' }}\nSubject: {{ $json.subject || $json.payload?.headers?.subject || '(no subject)' }}\nBody:\n{{ $json.snippet || $json.text || $json.body || '(no body)' }}\n\nPlease analyze the content above and generate only strict JSON output in this exact format:\n{\n  \"reply\": \"<Your short, natural, and context-relevant reply text here>\",\n  \"intent\": \"auto_reply\",\n  \"confidence\": 0.8\n}"
    }
  ],
  "options": {
    "temperature": 0.6,
    "top_p": 0.9,
    "format": "json"
  }
}
```

#### What the JSON Script Does
- Uses the Llama 3 model to generate intelligent email replies.
- Disables streaming so the full response is received at once.
- Sets the AI’s role and tone as Philip’s polite professional assistant.
- Sends the real From, Subject, and Body of each email for analysis.
- Instructs the AI to return only strict JSON with reply, intent, and confidence.
- Applies balanced creativity settings (temperature 0.6, top_p 0.9) for natural text.
- Ensures the output is always valid JSON ready for the next node.


Test the Node
- Click Execute Node.
- You should receive a JSON response containing the AI-generated reply text.
  <img width="923" height="263" alt="image" src="https://github.com/user-attachments/assets/1e79f0a8-1224-4526-b207-4f827e720d46" />

 



## Step 3 – Conditional Setup (If Node)
### Goal: Check whether the incoming email should be replied to or ignored.

Add If Node
- Click ➕ Add Node → If.
  <img width="931" height="432" alt="image" src="https://github.com/user-attachments/assets/e218ebae-d8cb-40e0-858a-e3b2e1e89024" />

- Rename to Reply Filter.

Set Condition Logic
In the first value box (value1):

- Click inside the box.
- Choose “Expression” (the small gear ⚙️ or the blue fx icon).
- Enter the following expression:
 ```
{{$json["subject"]}}
``` 
<img width="802" height="282" alt="image" src="https://github.com/user-attachments/assets/41255efb-b092-42f5-a247-bf02a9a02786" />

This means you’re reading the “subject” field from the incoming email (from your Gmail trigger).


Operator
In the middle dropdown, change:
- From “is equal to”, To “does not contain”
  <img width="696" height="128" alt="image" src="https://github.com/user-attachments/assets/f7076ea7-cf08-4c07-9178-789e6e5f6c6b" />

 
Comparison Value
- In the second value box (value2): Enter the first word to check for, for example:

newsletter
<img width="469" height="69" alt="image" src="https://github.com/user-attachments/assets/d957ad2a-8efc-4b1c-b606-803b184562a0" />

 
Now, click “Add condition” and add another condition line below.
On the second line:
- In value1, again put
  ```
  {{$json["subject"]}}
  ```
- In the operator, select “does not contain”
- In value2, type:

  no-reply
 <img width="975" height="98" alt="image" src="https://github.com/user-attachments/assets/d9a27f0a-f9bb-46c8-b52f-be740dd59a37" />


Combine Logic
- Scroll down a little — you’ll see “Combine conditions with”.
- Change it from AND → AND (keep it like that).
  Meaning both must not appear.
  <img width="975" height="213" alt="image" src="https://github.com/user-attachments/assets/d542ea2e-cd6a-453c-986f-d5a6e23c1b1d" />

 



Connect Flow

Once saved:
- From the True output (green line) → connect to your next action node (e.g., Ollama or reply). (we would continue in step 4)
- From the False output (red line) → connect to a Stop node or a Set node that just logs “Skipped newsletter or no-reply email.”
    - Click “+” → choose “No Operation, do nothing”.
    - Connect the False (red) output → to this node.
      <img width="903" height="249" alt="image" src="https://github.com/user-attachments/assets/9ed60be5-c2e1-4b8e-a89e-17f853a7cd76" />

 

That’s it. The workflow ends there without doing anything.

Connect Node Flow:
- True → Continue workflow.
- False → Stop.


 














