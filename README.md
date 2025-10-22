import React, { useState, useEffect } from "react";

export default function HashtagTrendingFinder() { const [topic, setTopic] = useState(""); const [platform, setPlatform] = useState("tiktok"); const [count, setCount] = useState(10); const [hashtags, setHashtags] = useState([]); const [sounds, setSounds] = useState([]); const [loading, setLoading] = useState(false); const [favorites, setFavorites] = useState(() => { try { return JSON.parse(localStorage.getItem("ht_favs")) || []; } catch (e) { return []; } }); const [theme, setTheme] = useState("light");

useEffect(() => { localStorage.setItem("ht_favs", JSON.stringify(favorites)); }, [favorites]);

const mockTrendingKeywords = { tiktok: ["challenge", "duet", "viral", "fyp", "dance", "lifehack", "storytime", "trending", "tutorial", "forYou"], instagram: ["reels", "explore", "photography", "ootd", "instadaily", "foodie", "style", "motivation", "travel", "artist"], twitter: ["breaking", "news", "thread", "viral", "meme", "nowplaying", "opinion", "trending", "hot", "update"], youtube: ["shorts", "howto", "reaction", "gaming", "vlog", "tutorial", "challenge", "review", "behindthescenes", "collab"], };

const mockTrendingSounds = { tiktok: [ { title: "Feel-Good Beat", artist: "ProducerX", id: "tkt-001" }, { title: "Catchy Hook", artist: "Singer Y", id: "tkt-002" }, { title: "Retro Loop", artist: "DJ Z", id: "tkt-003" }, ], instagram: [ { title: "Acoustic Short", artist: "Indie A", id: "ig-001" }, { title: "Uplifting Pop", artist: "PopB", id: "ig-002" }, ], twitter: [], youtube: [ { title: "Instrumental Hook", artist: "Composer C", id: "yt-001" }, { title: "Looped Beat", artist: "LoopMaster", id: "yt-002" }, ], };

function buildHashtag(word) { return ( "#" + word .replace(/[^a-zA-Z0-9 ]/g, "") .split(" ") .map((w, i) => (i === 0 ? w.toLowerCase() : capitalize(w))) .join("") ); } function capitalize(s) { return s.charAt(0).toUpperCase() + s.slice(1).toLowerCase(); }

function extractHashtagsFromKeywords(topicInput, keywords, max) { const out = new Set(); const t = topicInput.trim(); if (!t) { for (let k of keywords.slice(0, max)) out.add(buildHashtag(k)); return Array.from(out); } out.add(buildHashtag(t)); out.add(buildHashtag(${t}Tips)); out.add(buildHashtag(${t}Life)); out.add(buildHashtag(${t}Hacks)); for (let k of keywords) { if (out.size >= max) break; out.add(buildHashtag(${t} ${k})); if (out.size >= max) break; out.add(buildHashtag(${k} ${t})); } let idx = 0; while (out.size < max && idx < keywords.length) { out.add(buildHashtag(keywords[idx])); idx++; } return Array.from(out).slice(0, max); }

async function fetchMockTrending(platformKey) { return new Promise((resolve) => { setTimeout(() => { resolve({ keywords: mockTrendingKeywords[platformKey] || [], sounds: mockTrendingSounds[platformKey] || [], }); }, 350); }); }

async function generate() { setLoading(true); try { const data = await fetchMockTrending(platform); const suggested = extractHashtagsFromKeywords(topic, data.keywords, count); setHashtags(suggested); setSounds(data.sounds.slice(0, 6)); } catch (e) { console.error(e); alert("Failed to generate hashtags."); } finally { setLoading(false); } }

function copyToClipboard(text) { navigator.clipboard.writeText(text).then(() => alert("Copied!")); }

function downloadCSV() { const csv = hashtags.join("\n"); const blob = new Blob([csv], { type: "text/csv;charset=utf-8;" }); const url = URL.createObjectURL(blob); const a = document.createElement("a"); a.href = url; a.download = ${topic || "hashtags"}_suggestions.csv; a.click(); URL.revokeObjectURL(url); }

function toggleFavorite(item) { setFavorites((prev) => (prev.includes(item) ? prev.filter((p) => p !== item) : [item, ...prev].slice(0, 50))); }

return ( <div className={${theme === 'dark' ? 'bg-gray-900 text-white' : 'bg-gradient-to-b from-slate-50 to-white'} min-h-screen p-6}> <div className="max-w-4xl mx-auto"> <header className="mb-6 flex justify-between items-center"> <div> <h1 className="text-3xl font-extrabold">Hashtag & Trending Sound Finder</h1> <p className="text-sm text-gray-600 mt-1">Enter a topic and platform — get suggested hashtags and trending sounds.</p> </div> <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')} className="px-4 py-2 rounded-lg border"> {theme === 'light' ? 'Dark Mode' : 'Light Mode'} </button> </header>

<section className={`${theme === 'dark' ? 'bg-gray-800' : 'bg-white'} p-6 rounded-2xl shadow-md`}>
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
          <div className="mt-3 flex flex-wrap gap-2">
            {hashtags.map((h) => (
              <div key={h} className="flex items-center gap-2 bg-gray-100 rounded-full px-3 py-2">
                <button onClick={() => copyToClipboard(h)} className="text-sm font-medium">{h}</button>
                <button onClick={() => toggleFavorite(h)} title="Favorite" className={`text-sm ${favorites.includes(h) ? "text-yellow-500" : "text-gray-400"}`}>★</button>
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
          <h2 className="text-lg font-semibold">Trending Sounds</h2>
          <ul className="mt-3 space-y-2">
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

      <footer className="mt-6 text-xs text-gray-500">Tip: Toggle theme with the button above!</footer>
    </section>
  </div>
</div>

); }
