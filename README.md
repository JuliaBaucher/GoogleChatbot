**JULIA BAUCHER AI CHATBOT: END-TO-END DEPLOYMENT GUIDE**

This guide covers the secure setup and deployment of a professional AI assistant using Google Cloud and Gemini.

================================================================================

#STEP 0: THE TOOLBOX

• Google Cloud Account: Access via the Google Cloud Console.

• Google AI Studio: Access via AI Studio to retrieve your API key.

• GitHub: Host for your professional CV website.

================================================================================

#STEP 1: GET YOUR GEMINI API KEY

• Navigate to Google AI Studio.

[Link text](https://aistudio.google.com/)

• Click Get API key on the left sidebar.

• Select Create API key in new project.

• Copy the generated key and save it in a secure Notepad file.

================================================================================

#STEP 2: SET UP YOUR GOOGLE CLOUD PROJECT

• Open the Google Cloud Console.

[Link text](https://console.cloud.google.com/)

• Click the Project Dropdown at the top of the page and select New Project.

• Name the project julia-cv-bot and click Create.

• Ensure the new project is active in the top navigation bar before proceeding.

================================================================================

#STEP 3: ENABLE THE CLOUD BRAIN APIS

• Click the Hamburger Menu (three horizontal lines) in the top-left corner.

• Hover over APIs and Services and select Library.

• Search for Cloud Functions API in the search bar and click Enable.

• Search for Cloud Build API in the search bar and click Enable.

• Search for Generative Language API in the search bar and click Enable.

================================================================================

#STEP 4: CREATE YOUR BACKEND CLOUD FUNCTION

##4A) INITIAL SETUP

• Search for Cloud Functions in the top search bar (Marketplace) and select the service.

• Click Create Function.

• Set Environment to 2nd Gen.

• Set Function name to askgemini.

• Set Region to europe-west1.

• Select Allow unauthenticated invocations under the Authentication section.

##4B) LINKING THE KEY (SECURE METHOD)

• Navigate to APIs and Services then Credentials.

• Click Create Credentials and select API Key if the Studio key is not listed.

• Click the Key Name and set API restrictions to Don't restrict key.

• Return to the Cloud Function setup and click Runtime, build, connections and security settings.

• Add an Environment variable with the name GEMINI_API_KEY and paste your key.

• Add an Environment variable with the name KNOWLEDGE_BASE and paste your detailed bio.

• Click Next to proceed to the code editor.

##4C) ADD THE LOGIC

• Set Runtime to Node.js 20.

• Update the Entry point field to askgemini to match the function name in the code.

PACKAGE.JSON

```
{
  "name": "julia-cv-bot",
  "version": "1.0.0",
  "dependencies": {
    "@google/generative-ai": "^0.21.0"
  }
}
```


INDEX.JS

```

const { GoogleGenerativeAI } = require("@google/generative-ai");

exports.askgemini = async (req, res) => {
  res.set("Access-Control-Allow-Origin", "*");
  res.set("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.set("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") return res.status(204).send("");

  try {
    const apiKey = process.env.GEMINI_API_KEY;
    const knowledgeBase = process.env.KNOWLEDGE_BASE || "Senior Product Manager at Amazon.";
    const genAI = new GoogleGenerativeAI(apiKey);
    const model = genAI.getGenerativeModel({ model: "gemini-2.0-flash" });

    const userMessage = req.body.message;
    if (!userMessage) {
      return res.status(200).send({ reply: "Bonjour! I am Julia's AI assistant. How can I help you today?" });
    }

    const fullPrompt = `Context: ${knowledgeBase}\n\nQuestion: ${userMessage}`;
    const result = await model.generateContent(fullPrompt);
    const response = await result.response;
    const text = response.text();

    res.status(200).send({ reply: text });
  } catch (error) {
    console.error("Error:", error.message);
    const errorMsg = error.message.includes("429") 
      ? "I am receiving many questions! Please wait 60 seconds." 
      : "I am having a technical moment. Please try again shortly.";
    res.status(200).send({ reply: errorMsg });
  }
};

```

• Click Deploy and wait approximately 2 minutes for completion.

• Copy the Trigger URL provided after successful deployment.

================================================================================

#STEP 5: UPDATE GITHUB

• Open your website repository on GitHub.

• Paste the Trigger URL into the JavaScript fetch() command in your code.

• Commit and push the changes to your live site.

================================================================================

#STEP 6: THE ARCHITECT'S SECRET TEST

• Verify the API status by visiting this URL: https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_API_KEY

• If you see a JSON list of models: The key and API are correctly configured.

• If you see 403 Restricted: The key has restrictions blocking Gemini.

• If you see 404 Not Found: The Generative Language API is disabled.
