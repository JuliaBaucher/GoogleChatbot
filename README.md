Here is the updated guide formatted specifically for a GitHub README.md. I have used professional Markdown syntax, including task lists, bold highlights, and code blocks to ensure it looks perfect on your repository page.

🤖 Julia's AI Chatbot: End-to-End Deployment Guide
This guide covers the secure setup and deployment of a professional AI assistant using Google Cloud and Gemini.

🛠 Step 0: The Toolbox
Google Cloud Account: Cloud Console

Google AI Studio: AI Studio (To get your API key)

GitHub: Where your CV website is hosted.

🔑 Step 1: Get Your Gemini API Key
Go to Google AI Studio.

Click "Get API key" on the left sidebar.

Click "Create API key in new project".

Copy the key and save it in a Notepad. You will need it for Step 4.

🏗 Step 2: Set Up Your Google Cloud Project
Go to the Cloud Console.

At the top, click the Project Dropdown > New Project.

Name it julia-cv-bot and click Create.

⚠️ Important: Ensure your new project is selected in the top bar before proceeding.

💡 Step 3: Enable the "Cloud Brain" APIs
Search for and Enable these three APIs in the search bar:

[ ] Cloud Functions API

[ ] Cloud Build API

[ ] Generative Language API (Crucial for Gemini access)

🚀 Step 4: Create Your Backend (Cloud Function)
4A) Initial Setup
Search for Cloud Functions and click Create Function.

Environment: 2nd Gen.

Function name: askgemini (Keep it lowercase for consistency).

Region: europe-west1.

Authentication: Select "Allow unauthenticated invocations".

4B) Linking the Key (Secure Method)
Go to APIs & Services > Credentials.

Verify: If your key isn't there, click + CREATE CREDENTIALS > API Key.

Restrictions: Click your Key Name. Under API restrictions, ensure "Don't restrict key" is selected.

In Cloud Function Settings: Click Runtime, build, connections and security settings.

Under Environment variables, click Add Variable:

Name: GEMINI_API_KEY

Value: [Your API Key]

(Optional) Add another variable:

Name: KNOWLEDGE_BASE

Value: [Paste your long Bio here]

4C) Add the Logic (The Code)
Runtime: Node.js 20

{
  "name": "julia-cv-bot",
  "version": "1.0.0",
  "dependencies": {
    "@google/generative-ai": "^0.21.0"
  }
}

#Index.js with knoweldge base and system prompt in the environemental variable

const { GoogleGenerativeAI } = require("@google/generative-ai");

exports.askgemini = async (req, res) => {
  // CORS setup: Essential for GitHub Pages
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
      return res.status(200).send({ reply: "Bonjour! I'm Julia's AI assistant. How can I help you today?" });
    }

    const fullPrompt = `Context: ${knowledgeBase}\n\nQuestion: ${userMessage}`;

    const result = await model.generateContent(fullPrompt);
    const response = await result.response;
    const text = response.text();

    res.status(200).send({ reply: text });
  } catch (error) {
    console.error("Error:", error.message);
    const errorMsg = error.message.includes("429") 
      ? "I'm receiving many questions! Please wait 60 seconds." 
      : "I'm having a technical moment. Please try again shortly.";
    res.status(200).send({ reply: errorMsg });
  }
};
#index.js with knowledge base integrated in the fonction 
const { GoogleGenerativeAI } = require("@google/generative-ai");

// Note: On utilise askgemini (minuscule) pour correspondre à votre version fonctionnelle
exports.askgemini = async (req, res) => {
  res.set("Access-Control-Allow-Origin", "*");
  res.set("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.set("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") return res.status(204).send("");

  try {
    const apiKey = process.env.GEMINI_API_KEY;
    if (!apiKey) throw new Error("La clé API n'est pas configurée.");

    const genAI = new GoogleGenerativeAI(apiKey);
    const model = genAI.getGenerativeModel({ model: "gemini-2.0-flash" });

    const userMessage = req.body.message;
    if (!userMessage) {
      return res.status(200).send({ reply: "Connexion établie ! Posez votre question." });
    }

    // Intégration de la base de connaissances (Knowledge Base)
    const knowledgeBase = `
      Tu es l'assistant IA de Julia Baucher sur son site CV. 
      Julia est Senior Product Manager III chez Amazon (ML/AI). 
      Détails clés : PhD École des Mines, experte en GenAI, RAG, AWS Bedrock, et ex-Amadeus. 
      Réponds de façon professionnelle et concise.
    `;

    // On combine la base de connaissances avec la question de l'utilisateur
    const fullPrompt = `${knowledgeBase}\n\nQuestion de l'utilisateur: ${userMessage}`;

    const result = await model.generateContent(fullPrompt);
    const response = await result.response;
    const text = response.text();

    res.status(200).send({ reply: text });

  } catch (error) {
    console.error("ERREUR:", error.message);
    res.status(500).send({ 
      reply: "Désolé, j'ai une erreur technique : " + error.message 
    });
  }
};

Click Deploy. Once finished, copy the Trigger URL.

🌐 Step 5: Update GitHub
Paste the Trigger URL from Google Cloud into your website's JavaScript fetch() command.

🔍 Step 6: The Architect's Secret Test
If the bot doesn't respond, test your key directly in the browser: https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_API_KEY

Result,Meaning
JSON list of models,Your Key and API are working perfectly.
403 Restricted,Your Key has API restrictions that block Gemini.
404 Not Found,The Generative Language API is not enabled in your project.

Entry point: askgemini (Must match the exports name in the code).

package.json
