const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type"
};

export default {
  async fetch(request, env) {
    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    const json = (data, status = 200) =>
      new Response(JSON.stringify(data), {
        status,
        headers: {
          ...corsHeaders,
          "Content-Type": "application/json"
        }
      });

    try {
      const url = new URL(request.url);

      if (request.method !== "POST") {
        return json({ error: "Use POST" }, 405);
      }

      const body = await request.json();

      if (url.pathname === "/chat") {
        const response = await fetch(
          "https://api.openai.com/v1/responses",
          {
            method: "POST",
            headers: {
              "Authorization": `Bearer ${env.OPENAI_API_KEY}`,
              "Content-Type": "application/json"
            },
            body: JSON.stringify({
              model: "gpt-4.1-mini",
              instructions: "You are JARVIS, a helpful AI assistant. Reply in Russian unless asked otherwise.",
              input: String(body.message || "").slice(0, 4000)
            })
          }
        );

        const data = await response.json();

        if (!response.ok) {
          return json({ error: "AI request failed" }, 502);
        }

        const answer = data.output
          ?.flatMap(item => item.content || [])
          .filter(item => item.type === "output_text")
          .map(item => item.text)
          .join("") || "Не удалось получить ответ.";

        return json({ answer });
      }

      if (url.pathname === "/speak") {
        const response = await fetch(
          "https://api.openai.com/v1/audio/speech",
          {
            method: "POST",
            headers: {
              "Authorization": `Bearer ${env.OPENAI_API_KEY}`,
              "Content-Type": "application/json"
            },
            body: JSON.stringify({
              model: "gpt-4o-mini-tts",
              voice: "onyx",
              input: String(body.text || "").slice(0, 1000),
              instructions: "Speak calmly, deeply, clearly and confidently, like a cinematic AI assistant. This is an AI-generated voice.",
              response_format: "mp3"
            })
          }
        );

        if (!response.ok) {
          return json({ error: "Voice generation failed" }, 502);
        }

        return new Response(response.body, {
          headers: {
            ...corsHeaders,
            "Content-Type": "audio/mpeg"
          }
        });
      }

      return json({ error: "Unknown route" }, 404);

    } catch (error) {
      return json({ error: "Server error" }, 500);
    }
  }
};
