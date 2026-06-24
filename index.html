// =====================================================================
// Somesh Chandra — Personal Site Chatbot Backend (Cloudflare Worker)
// =====================================================================
//
// SECURITY IN THIS VERSION:
// 1. CORS locked to your domain only (someshchandra007.github.io) —
//    no other website can call this Worker and burn your OpenAI credits.
// 2. Rate limiting: each visitor (identified by IP) gets a maximum of
//    10 questions per rolling 24-hour window, stored in Cloudflare KV.
//
// ===================== ONE-TIME SETUP: KV NAMESPACE =====================
// Rate limiting needs a small free Cloudflare KV storage namespace:
// 1. Cloudflare Dashboard -> Workers & Pages -> KV -> Create a namespace
//    Name it: CHAT_RATE_LIMIT
// 2. Go to your Worker -> Settings -> Variables -> KV Namespace Bindings
//    -> Add binding:
//       Variable name: RATE_LIMIT_KV
//       KV namespace: CHAT_RATE_LIMIT (the one you just created)
// 3. Save and deploy.
// (Without this binding, the rate limiter will fail open and just allow
//  requests through with a console warning — site won't break, but won't
//  be rate-limited either, so it's worth doing this step.)
// ==========================================================================

const CHATBOT_ENABLED = true; // <-- flip to false to switch the bot off instantly
const MAX_QUESTIONS_PER_WINDOW = 10; // max questions per visitor per window
const WINDOW_SECONDS = 24 * 60 * 60; // 24 hours

// Only this origin is allowed to call this Worker.
const ALLOWED_ORIGIN = "https://someshchandra007.github.io";

// ---- YOUR BIO / CONTEXT — edit this any time to update what the bot knows ----
const SOMESH_PROFILE = `
You are answering questions on behalf of Somesh Chandra, as his personal AI assistant on his portfolio website.
Speak about him in the third person (e.g. "Somesh has..." / "He specializes in...").

PROFILE:
Name: Somesh Chandra
Role: Data Architect, based in Slough, UK
Experience: 18+ years in IT and Data

SUMMARY:
A seasoned data professional with in-depth expertise in designing and building data architecture and
engineering solutions, leveraging Cloud and Big Data technologies to deliver scalable, cost-efficient,
business-focused outcomes. Hands-on experience building Data Lake, Delta Lake, and Lakehouse architectures
on AWS and Azure for advanced analytics and AI/ML workloads.

CERTIFICATIONS:
- Databricks Certified Data Engineer Professional
- Databricks Certified Generative AI Engineer Associate
- Databricks Certified Data Engineer Associate
- AWS Certified Solutions Architect – Associate
- SnowPro Associate (Snowflake)

CORE SKILLS:
Data Architecture, Data Engineering, Data Warehousing, Cloud Solutions (AWS & Azure), Databricks, AWS EMR,
AWS Glue, Spark, PySpark, Python, Data Pipeline Design, SQL Query Optimization, Machine Learning,
DevOps (GitHub, Terraform, Sonar, CI/CD), Snowflake, Redshift, DynamoDB, Apache Airflow, dbt.

GENAI WORK:
Built a GenAI solution for internal user communities on Databricks using RAG (Retrieval-Augmented Generation),
vector search, and customer profile embedding. Deployed ML models (logistic regression, XGBoost) using
AWS SageMaker, including feature engineering and model latency optimization.

EXPERIENCE TIMELINE:
- Data Architect, Tata Consultancy Services (Dec 2021 – Present): Led the build of an enterprise-grade
  Lakehouse on Databricks/Azure for one of the UK's largest building societies. Owned the data strategy
  roadmap, led the data engineering and BI teams, designed ingestion frameworks following Medallion
  Architecture, implemented DevOps practices (GitHub, Python-lint, SQLFluff, SonarQube).
- Senior Data Engineer, Novartis (Jun 2020 – Dec 2021): Built an enterprise-grade Data Lake on AWS using
  Databricks. Worked extensively on clinical trial and drug development data. Established governance
  models for data integration. Used Spark, Python, SQL, AWS Lambda, Athena, SNS, SES, EventBridge.
- Cloud Data Architect, Cognizant Technology Solutions (Nov 2014 – Jun 2020): Oversaw construction of
  "FormulaOne", an enterprise Data Lake on AWS. Defined data architecture and data layers
  (landing/curated/publish). Implemented best practices using Parquet, Delta formats, S3, Spark
  partitioning, and KMS encryption.
- Senior Software Developer, Orga Systems India Pvt Ltd (Jul 2014 – Nov 2014): Led design/development
  of backend Oracle DB functionality for a Telecom billing solution. Prepared design documents and LLDs.
- System Engineer, IBM (Aug 2011 – Jun 2014): ETL development using Oracle PL/SQL and Shell scripting —
  procedures, packages, functions, views, materialized views, SQL Loader, performance tuning.
- Technical Associate, Tech Mahindra Ltd (Jul 2007 – Aug 2011): Oracle PL/SQL developer; data analyst
  work including performance tuning, materialized views, job scheduling, and data migration.

EDUCATION:
Bachelor of Technology in Computer Science and Engineering, Haldia Institute of Technology, Kolkata, India
(May 2003 – Jun 2007)

DOMAINS SERVED: Banking, Energy Supply, Pharmaceutical, Telecom

CONTACT:
Email: someshchandra007@gmail.com
LinkedIn: https://www.linkedin.com/in/somesh-chandra-9853023a/
GitHub: https://github.com/someshchandra007
Phone: 07747 267818

INSTRUCTIONS FOR HOW TO ANSWER:
- Only answer questions related to Somesh's professional background, skills, experience, certifications, and how to contact him.
- If asked something unrelated to Somesh (general trivia, coding help, world facts, opinions on other topics, etc.),
  politely decline and redirect, e.g.: "I'm just here to answer questions about Somesh's background and experience —
  feel free to ask me about his skills, certifications, or projects!"
- Keep answers concise and conversational — 2 to 4 sentences, not long essays, unless the user clearly asks for detail.
- Be warm and professional, like a knowledgeable colleague introducing Somesh, not a robotic database lookup.
- Never make up experience, employers, or skills not listed above.
`;

function corsHeaders(origin) {
  // Only ever echo back the allowed origin, never the caller's arbitrary origin.
  return {
    "Access-Control-Allow-Origin": ALLOWED_ORIGIN,
    "Access-Control-Allow-Methods": "POST, OPTIONS",
    "Access-Control-Allow-Headers": "Content-Type",
  };
}

export default {
  async fetch(request, env) {
    const origin = request.headers.get("Origin") || "";
    const headers = corsHeaders(origin);

    if (request.method === "OPTIONS") {
      return new Response(null, { headers });
    }

    // Reject anything not coming from your site (browsers send Origin on cross-site fetches).
    if (origin && origin !== ALLOWED_ORIGIN) {
      return new Response(JSON.stringify({ error: "Origin not allowed" }), {
        status: 403,
        headers: { ...headers, "Content-Type": "application/json" },
      });
    }

    if (request.method !== "POST") {
      return new Response(JSON.stringify({ error: "Use POST" }), {
        status: 405,
        headers: { ...headers, "Content-Type": "application/json" },
      });
    }

    if (!CHATBOT_ENABLED) {
      return new Response(
        JSON.stringify({
          reply: "The assistant is temporarily offline. Please reach out via the contact section below in the meantime!",
        }),
        { headers: { ...headers, "Content-Type": "application/json" } }
      );
    }

    // ---------------- Rate limiting (per visitor IP, via Cloudflare KV) ----------------
    const ip = request.headers.get("CF-Connecting-IP") || "unknown";
    const rateLimitKey = `ratelimit:${ip}`;

    if (env.RATE_LIMIT_KV) {
      try {
        const existing = await env.RATE_LIMIT_KV.get(rateLimitKey);
        const count = existing ? parseInt(existing, 10) : 0;

        if (count >= MAX_QUESTIONS_PER_WINDOW) {
          return new Response(
            JSON.stringify({
              reply: "You've reached the question limit for now. Please check back later, or reach out directly via the contact section below!",
            }),
            { status: 429, headers: { ...headers, "Content-Type": "application/json" } }
          );
        }

        // Increment count, keep the same expiry window (or start a fresh one if new).
        await env.RATE_LIMIT_KV.put(rateLimitKey, String(count + 1), {
          expirationTtl: WINDOW_SECONDS,
        });
      } catch (e) {
        console.warn("Rate limit check failed, allowing request:", e.message);
      }
    } else {
      console.warn("RATE_LIMIT_KV not bound — rate limiting is disabled. See setup instructions.");
    }

    // ---------------- Main chat logic ----------------
    try {
      const { message } = await request.json();

      if (!message || typeof message !== "string" || message.length > 500) {
        return new Response(JSON.stringify({ error: "Invalid message" }), {
          status: 400,
          headers: { ...headers, "Content-Type": "application/json" },
        });
      }

      const openaiRes = await fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${env.OPENAI_API_KEY}`,
        },
        body: JSON.stringify({
          model: "gpt-4o-mini",
          max_tokens: 300,
          temperature: 0.4,
          messages: [
            { role: "system", content: SOMESH_PROFILE },
            { role: "user", content: message },
          ],
        }),
      });

      if (!openaiRes.ok) {
        const errText = await openaiRes.text();
        return new Response(JSON.stringify({ error: "Upstream error", detail: errText }), {
          status: 502,
          headers: { ...headers, "Content-Type": "application/json" },
        });
      }

      const data = await openaiRes.json();
      const reply = data.choices?.[0]?.message?.content?.trim() || "Sorry, I couldn't generate a response just now.";

      return new Response(JSON.stringify({ reply }), {
        headers: { ...headers, "Content-Type": "application/json" },
      });
    } catch (err) {
      return new Response(JSON.stringify({ error: "Server error", detail: err.message }), {
        status: 500,
        headers: { ...headers, "Content-Type": "application/json" },
      });
    }
  },
};
