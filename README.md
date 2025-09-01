import React, { useState } from "react";
import { Configuration, OpenAIApi } from "openai";

export default function QuemTemRazaoGame() {
  const [aText, setAText] = useState("");
  const [bText, setBText] = useState("");
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const configuration = new Configuration({
    apiKey: process.env.NEXT_PUBLIC_OPENAI_API_KEY,
  });
  const openai = new OpenAIApi(configuration);

  const avaliarDiscussao = async () => {
    setLoading(true);
    const prompt = `
Você é um árbitro divertido de discussões de casal. 
Pessoa A diz: "${aText}" 
Pessoa B diz: "${bText}" 
Analise quem tem mais razão com base em clareza, justificativas e lógica. 
Dê uma pontuação de 0 a 100 para cada um e uma mensagem divertida ou carinhosa.
Retorne JSON: { "winner": "Pessoa A ou Pessoa B ou Empate", "scoreA": X, "scoreB": Y, "feedback": "mensagem divertida" }
`;

    try {
      const response = await openai.createChatCompletion({
        model: "gpt-4o-mini",
        messages: [{ role: "user", content: prompt }],
      });

      const text = response.data.choices[0].message.content;
      setResult(JSON.parse(text));
    } catch (err) {
      console.error(err);
      alert("Erro na IA, tente novamente!");
    }

    setLoading(false);
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-pink-50 p-6">
      <h1 className="text-3xl font-bold mb-6">Quem Tem Razão? 💑</h1>
      <div className="grid md:grid-cols-2 gap-4 mb-4 w-full max-w-4xl">
        <textarea
          placeholder="Pessoa A: sua versão..."
          className="p-3 border rounded"
          value={aText}
          onChange={(e) => setAText(e.target.value)}
        />
        <textarea
          placeholder="Pessoa B: sua versão..."
          className="p-3 border rounded"
          value={bText}
          onChange={(e) => setBText(e.target.value)}
        />
      </div>
      <button
        onClick={avaliarDiscussao}
        disabled={loading || !aText || !bText}
        className="bg-pink-500 text-white px-6 py-2 rounded hover:bg-pink-600"
      >
        {loading ? "Analisando..." : "Descobrir quem tem razão"}
      </button>

      {result && (
        <div className="mt-6 p-4 border rounded bg-white w-full max-w-2xl text-center space-y-2">
          <h2 className="text-xl font-semibold">Resultado: {result.winner}</h2>
          <p>Pessoa A: {result.scoreA}% | Pessoa B: {result.scoreB}%</p>
          <p className="mt-2">{result.feedback}</p>
        </div>
      )}
    </div>
  );
}
