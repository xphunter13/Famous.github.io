import React, { useState, useEffect } from "react";

// Hashtag & Trending Sound Finder // Single-file React component (can be used in Create React App, Vite, Next.js page, etc.) // Styling uses Tailwind CSS classes (link tailwind in your app or use CDN for quick demo). // // How it works (local demo): // - You enter a topic (e.g. "football", "makeup", "lifehacks"). // - Pick a platform (TikTok, Instagram, Twitter, YouTube Shorts). // - The generator combines the topic with a list of mock "trending keywords" and surfaces //   hashtag suggestions and "trending sounds" (mock data). // - There are utilities to copy hashtags, download them as CSV, and save favorites locally. // // To add live, real-time trending data and sounds: // 1) Sign up for a trends API (RapidAPI has several options for TikTok/Instagram trends). // 2) Replace the fetchMockTrending(...) function with a real fetch call to the API. // 3) Pass your API key in the Authorization header or query string as required. // 4) Map the API response fields into the format expected by extractHashtagsFromKeywords. // // Important: Many platforms (TikTok/Instagram) don't provide free official public APIs for // trending hashtags/sounds. You may need to use third-party services or build a lightweight // web-scraper/server-side fetch to collect trending keywords legally and respect each // platform's terms of service.

export default function HashtagTrendingFinder() { const [topic, setTopic] = useState(""); const [platform, setPlatform] = useState("tiktok"); const [count, setCount] = useState(10); const [hashtags, setHashtags] = useState([]); const [sounds, setSounds] = useState([]); const [loading, setLoading] = useState(false); const [favorites, setFavorites] = useState(() => { try { return JSON.parse(localStorage.getItem("ht_favs")) || []; } catch (e) { return []; } });

useEffect(() => { localStorage.setItem("ht_favs", JSON.stringify(favorites)); }, [favorites]);

// Mock trending keywords per platform — replace with real API responses for live data const mockTrendingKeywords = { tiktok: ["challenge", "duet", "viral", "fyp", "dance", "lifehack", "storytime", "trending", "tutorial", "forYou"], instagram: ["reels", "explore", "photography", "ootd", "instadaily", "foodie", "style", "motivation", "travel", "artist"], twitter: ["breaking", "news", "thread", "viral", "meme", "nowplaying", "opinion", "trending", "hot", "update"], youtube: ["shorts", "howto", "reaction", "gaming", "vlog", "tutorial", "challenge", "review", "behindthescenes", "collab"], };

const mockTrendingSounds = { tiktok: [ { title: "Feel-Good Beat", artist: "ProducerX", id: "tkt-001" }, { title: "Catchy Hook", artist: "Singer Y", id: "tkt-002" }, { title: "Retro Loop", artist: "DJ Z", id: "tkt-003" }, ], instagram: [ { title: "Acoustic Short", artist: "Indie A", id: "ig-001" }, { title: "Uplifting Pop", artist: "PopB", id: "ig-002" }, ], twitter: [], youtube: [ { title: "Instrumental Hook", artist: "Composer C", id: "yt-001" }, { title: "Looped Beat", artist: "LoopMaster", id: "yt-002" }, ], };

// Utility: sanitize and build hashtag strings function buildHashtag(word) { // remove invalid characters, spaces -> camelCase return ( "#" + word .replace(/[^a-zA-Z0-9 ]/g, "") .split(" ") .map((w, i) => (i === 0 ? w.toLowerCase() : capitalize(w))) .join("") ); } function capitalize(s) { return s.charAt(0).toUpperCase() + s.slice(1).toLowerCase(); }

// Combine topic + trending keyword to make sensible hashtag suggestions function extractHashtagsFromKeywords(topicInput, keywords, max) { const out = new Set(); const t = topicInput.trim();

if (!t) {
  // If no topic, just return top trending keyword hashtags
  for (let k of keywords.slice(0, max)) out.add(buildHashtag(k));
  return Array.from(out);
}

// Base tag from topic
out.add(buildHashtag(t));

// Variations
out.add(buildHashtag(`${t}Tips`));
out.add(buildHashtag(`${t}Life`));
out.add(buildHashtag(`${t}Hacks`));

// Mix trending keywords
for (let k of keywords) {
  if (out.size >= max) break;
  out.add(buildHashtag(`${t} ${k}`));
  if (out.size >= max) break;
  out.add(buildHashtag(`${k} ${t}`));
}

// If still fewer than requested, fill with platform keywords
let idx = 0;
while (out.size < max && idx < keywords.length) {
  out.add(buildHashtag(keywords[idx]));
  idx++;
}

return Array.from(out).slice(0, max);

}

// Mock fetch function to emulate calling an API for trending keywords/sounds async function fetchMockTrending(platformKey) { // In real integration, call a server-side endpoint which queries platform APIs or // third-party trend services and returns a list of keywords and popular sounds. // For demo we return a promise that resolves after a short delay. return new Promise((resolve) => { setTimeout(() => { resolve({ keywords: mockTrendingKeywords[platformKey] || [], sounds: mockTrendingSounds[platformKey] || [], }); }, 350); }); }

async function generate() { setLoading(true); try { // Replace this call with your real API call when integrating live data. const data = await fetchMockTrending(platform); const suggested = extractHashtagsFromKeywords(topic, data.keywords, count); setHashtags(suggested); setSounds(data.sounds.slice(0, 6)); } catch (e) { console.error(e); alert("Failed to generate hashtags — check console for details."); } finally { setLoading(false); } }

function copyToClipboard(text) { navigator.clipboard.writeText(text).then( () => { // small UI feedback alert("Copied to clipboard!"); }, (err) => { alert("Copy failed: " + String(err)); } ); }

function downloadCSV() { const csv = hashtags.map((h) => ${h}).join("\n"); const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" }); const url = URL.createObjectURL(blob); const a = document.createElement("a"); a.href = url; a.download = ${topic || "hashtags"}_suggestions.csv; a.click(); URL.revokeObjectURL(url); }

function toggleFavorite(item) { setFavorites((prev) => { if (prev.includes(item)) return prev.filter((p) => p !== item); return [item, ...prev].slice(0, 50); }); }

return ( <div className="min-h-screen bg-gradient-to-b from-slate-50 to-white p-6"> <div className="max-w-4xl mx-auto"> <header className="mb-6"> <h1 className="text-3xl font-extrabold">Hashtag & Trending Sound Finder</h1> <p className="text-sm text-gray-600 mt-1">Enter a topic and platform — get suggested hashtags and trending sounds.</p> </header>

<section className="bg-white p-6 rounded-2xl shadow-md">
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div className="col-span-2">
          <label className="block text-sm font-medium text-gray-700">Topic</label>
          <input
            value={topic}
            onChange={(e) => setTopic(e.target.value)}
            className="mt-1 block w-full rounded-lg border p-3"
            placeholder="e.g. 'makeup', 'football', 'study tips'"
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700">Platform</label>
          <select value={platform} onChange={(e) => setPlatform(e.target.value)} className="mt-1 block w-full rounded-lg border p-3">
            <option value="tiktok">TikTok</option>
            <option value="instagram">Instagram</option>
            <option value="twitter">Twitter/X</option>
            <option value="youtube">YouTube Shorts</option>
          </select>
        </div>

        <div className="md:col-span-3 flex items-end gap-3 mt-2">
          <div className="flex-1">
            <label className="block text-sm font-medium text-gray-700">How many hashtags</label>
            <input type="number" min={1} max={30} value={count} onChange={(e) => setCount(Number(e.target.value))} className="mt-1 block w-24 rounded-lg border p-2" />
          </div>

          <div className="flex gap-2">
            <button onClick={generate} className="inline-flex items-center px-4 py-2 rounded-xl bg-indigo-600 text-white hover:opacity-95">
              {loading ? "Generating..." : "Generate"}
            </button>

            <button onClick={() => { setTopic(""); setHashtags([]); setSounds([]); }} className="inline-flex items-center px-4 py-2 rounded-xl border">
              Clear
            </button>
          </div>
        </div>
      </div>

      <hr className="my-6" />

      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <h2 className="text-lg font-semibold">Suggested Hashtags</h2>
          <p className="text-sm text-gray-500">Click a tag to copy it, star to favorite, or download all as CSV.</p>

          <div className="mt-3 flex flex-wrap gap-2">
            {hashtags.length === 0 && <div className="text-sm text-gray-400">No hashtags yet — generate some!</div>}
            {hashtags.map((h) => (
              <div key={h} className="flex items-center gap-2 bg-gray-100 rounded-full px-3 py-2">
                <button onClick={() => copyToClipboard(h)} className="text-sm font-medium">{h}</button>
                <button onClick={() => toggleFavorite(h)} title="Favorite" className={`text-sm ${favorites.includes(h) ? "text-yellow-500" : "text-gray-400"}`}>
                  ★
                </button>
              </div>
            ))}
          </div>

          {hashtags.length > 0 && (
            <div className="mt-4 flex gap-2">
              <button onClick={() => copyToClipboard(hashtags.join(" "))} className="px-4 py-2 rounded-lg border">Copy all</button>
              <button onClick={downloadCSV} className="px-4 py-2 rounded-lg border">Download CSV</button>
            </div>
          )}
        </div>

        <div>
          <h2 className="text-lg font-semibold">Trending Sounds (mock)</h2>
          <p className="text-sm text-gray-500">These are demo sounds — replace with platform API data for live sounds.</p>

          <ul className="mt-3 space-y-2">
            {sounds.length === 0 && <li className="text-sm text-gray-400">No trending sounds available for this platform.</li>}
            {sounds.map((s) => (
              <li key={s.id} className="flex items-center justify-between bg-gray-50 p-3 rounded-lg">
                <div>
                  <div className="font-medium">{s.title}</div>
                  <div className="text-xs text-gray-500">{s.artist}</div>
                </div>
                <div className="flex gap-2">
                  <button onClick={() => alert('Pretend-play sound: "' + s.title + '"')} className="px-3 py-1 rounded-md border">Preview</button>
                  <button onClick={() => toggleFavorite(s.title)} className={`px-3 py-1 rounded-md border ${favorites.includes(s.title) ? 'text-yellow-500' : ''}`}>☆</button>
                </div>
              </li>
            ))}
          </ul>
        </div>
      </div>

      <hr className="my-6" />

      <div>
        <h3 className="font-semibold">Favorites</h3>
        <div className="mt-2 flex flex-wrap gap-2">
          {favorites.length === 0 && <div className="text-sm text-gray-400">You have no favorites yet.</div>}
          {favorites.map((f) => (
            <div key={f} className="bg-white px-3 py-2 rounded-lg border flex items-center gap-2">
              <div className="text-sm">{f}</div>
              <button onClick={() => toggleFavorite(f)} className="text-sm text-red-400">remove</button>
            </div>
          ))}
        </div>
      </div>

      <footer className="mt-6 text-xs text-gray-500">
        <div>Tip: For live trends, hook this UI to a server endpoint that queries platform-specific trend services.</div>
      </footer>
    </section>
  </div>
</div>

); }
