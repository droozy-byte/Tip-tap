# Tip-tap
App for posting 
import { useState, useEffect, useRef, useCallback } from “react”;

const FONT_URL = “https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800;900&family=Space+Mono:wght@400;700&display=swap”;

// ─── Persistent Storage Helpers ───
const store = window.storage;
async function loadState(key, fallback) {
try {
const r = await store.get(key);
return r ? JSON.parse(r.value) : fallback;
} catch { return fallback; }
}
async function saveState(key, val) {
try { await store.set(key, JSON.stringify(val)); } catch {}
}

// ─── Seed Data ───
const DEMO_USERS = [
{ id: “u1”, username: “nova.lens”, displayName: “Nova”, avatar: “🌌”, bio: “chasing light”, color: “#FF6B6B” },
{ id: “u2”, username: “byte.wolf”, displayName: “Byte”, avatar: “🐺”, bio: “code & chill”, color: “#4ECDC4” },
{ id: “u3”, username: “luna.wav”, displayName: “Luna”, avatar: “🌙”, bio: “sound designer”, color: “#A78BFA” },
{ id: “u4”, username: “pixel.riot”, displayName: “Pixel”, avatar: “🎮”, bio: “game dev things”, color: “#F59E0B” },
{ id: “u5”, username: “drift.mp4”, displayName: “Drift”, avatar: “🏎️”, bio: “motion graphics”, color: “#EC4899” },
];

const DEMO_POSTS = [
{ id: “p1”, userId: “u1”, type: “photo”, caption: “golden hour never misses 🌅”, media: “🌅”, timestamp: Date.now() - 3600000, likes: 42, comments: 7 },
{ id: “p2”, userId: “u2”, type: “text”, caption: “just shipped v2.0 — 3 months of pure grind. the codebase is clean and I’m not 🫠”, media: null, timestamp: Date.now() - 7200000, likes: 89, comments: 15 },
{ id: “p3”, userId: “u3”, type: “photo”, caption: “new sample pack dropping friday 🎧”, media: “🎵”, timestamp: Date.now() - 18000000, likes: 156, comments: 23 },
{ id: “p4”, userId: “u4”, type: “video”, caption: “gameplay sneak peek 👀”, media: “🎬”, timestamp: Date.now() - 36000000, likes: 234, comments: 31 },
{ id: “p5”, userId: “u5”, type: “photo”, caption: “this transition took me 6 hours but worth it”, media: “✨”, timestamp: Date.now() - 72000000, likes: 67, comments: 9 },
];

const DEMO_STORIES = [
{ userId: “u1”, items: [{ id: “s1”, emoji: “🏔️”, bg: “linear-gradient(135deg, #FF6B6B, #ee5a24)”, text: “hiking day!”, ts: Date.now() - 1800000 }] },
{ userId: “u3”, items: [{ id: “s2”, emoji: “🎹”, bg: “linear-gradient(135deg, #A78BFA, #6D28D9)”, text: “studio session”, ts: Date.now() - 3600000 }, { id: “s3”, emoji: “🎤”, bg: “linear-gradient(135deg, #6D28D9, #4C1D95)”, text: “recording vocals”, ts: Date.now() - 1200000 }] },
{ userId: “u5”, items: [{ id: “s4”, emoji: “🎨”, bg: “linear-gradient(135deg, #EC4899, #BE185D)”, text: “new project wip”, ts: Date.now() - 5400000 }] },
{ userId: “u4”, items: [{ id: “s5”, emoji: “🕹️”, bg: “linear-gradient(135deg, #F59E0B, #D97706)”, text: “bug hunting lol”, ts: Date.now() - 7200000 }] },
];

const DEMO_MESSAGES = {
“u1”: [
{ id: “m1”, from: “u1”, text: “yo check this shot i got”, ts: Date.now() - 60000, type: “text” },
{ id: “m2”, from: “me”, text: “that’s insane!! where was that?”, ts: Date.now() - 50000, type: “text” },
{ id: “m3”, from: “u1”, text: “rooftop on 5th, golden hour”, ts: Date.now() - 40000, type: “text” },
],
“u3”: [
{ id: “m4”, from: “u3”, text: “collab?”, ts: Date.now() - 120000, type: “text” },
{ id: “m5”, from: “me”, text: “let’s do it 🔥”, ts: Date.now() - 110000, type: “text” },
],
};

function timeAgo(ts) {
const diff = Date.now() - ts;
if (diff < 60000) return “now”;
if (diff < 3600000) return `${Math.floor(diff / 60000)}m`;
if (diff < 86400000) return `${Math.floor(diff / 3600000)}h`;
return `${Math.floor(diff / 86400000)}d`;
}

// ─── Styles ───
const css = `
@import url(’${FONT_URL}’);

- { margin:0; padding:0; box-sizing:border-box; }

:root {
–bg: #0A0A0F;
–surface: #13131A;
–surface2: #1C1C26;
–surface3: #252532;
–border: #2A2A3A;
–text: #EEEEF0;
–text2: #9999AA;
–text3: #666677;
–accent: #00E5A0;
–accent2: #00C48C;
–danger: #FF4757;
–gradient1: linear-gradient(135deg, #00E5A0, #00B4D8);
–gradient2: linear-gradient(135deg, #A78BFA, #EC4899);
–gradient3: linear-gradient(135deg, #F59E0B, #EF4444);
–radius: 16px;
–radius-sm: 10px;
–radius-xs: 6px;
–font: ‘Outfit’, sans-serif;
–mono: ‘Space Mono’, monospace;
}

body { background: var(–bg); color: var(–text); font-family: var(–font); }

.app {
max-width: 430px;
margin: 0 auto;
min-height: 100vh;
background: var(–bg);
position: relative;
overflow: hidden;
}

/* ─── Onboarding ─── */
.onboarding {
min-height: 100vh;
display: flex;
flex-direction: column;
align-items: center;
justify-content: center;
padding: 40px 24px;
text-align: center;
position: relative;
overflow: hidden;
}

.onboarding::before {
content: ‘’;
position: absolute;
width: 500px; height: 500px;
background: radial-gradient(circle, rgba(0,229,160,0.12) 0%, transparent 70%);
top: -100px; left: -100px;
animation: pulse-glow 4s ease-in-out infinite;
}

.onboarding::after {
content: ‘’;
position: absolute;
width: 400px; height: 400px;
background: radial-gradient(circle, rgba(167,139,250,0.08) 0%, transparent 70%);
bottom: -50px; right: -100px;
animation: pulse-glow 4s ease-in-out infinite 2s;
}

@keyframes pulse-glow {
0%,100% { transform: scale(1); opacity: 0.5; }
50% { transform: scale(1.1); opacity: 1; }
}

.ob-logo {
font-size: 56px;
font-weight: 900;
background: var(–gradient1);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
letter-spacing: -2px;
margin-bottom: 8px;
position: relative;
z-index: 1;
}

.ob-tagline {
color: var(–text2);
font-size: 16px;
font-weight: 300;
margin-bottom: 48px;
letter-spacing: 2px;
text-transform: uppercase;
position: relative; z-index: 1;
}

.ob-input-group {
width: 100%;
max-width: 320px;
position: relative; z-index: 1;
margin-bottom: 16px;
}

.ob-input {
width: 100%;
padding: 16px 20px;
background: var(–surface);
border: 1px solid var(–border);
border-radius: var(–radius);
color: var(–text);
font-family: var(–font);
font-size: 16px;
outline: none;
transition: all 0.3s ease;
}

.ob-input:focus {
border-color: var(–accent);
box-shadow: 0 0 0 3px rgba(0,229,160,0.1);
}

.ob-input::placeholder { color: var(–text3); }

.ob-btn {
width: 100%;
max-width: 320px;
padding: 16px;
background: var(–gradient1);
border: none;
border-radius: var(–radius);
color: #0A0A0F;
font-family: var(–font);
font-size: 16px;
font-weight: 700;
cursor: pointer;
position: relative; z-index: 1;
transition: transform 0.2s, box-shadow 0.2s;
margin-top: 8px;
}

.ob-btn:hover { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(0,229,160,0.2); }
.ob-btn:active { transform: translateY(0); }

.ob-btn:disabled {
opacity: 0.4;
cursor: not-allowed;
transform: none !important;
box-shadow: none !important;
}

.ob-emoji-grid {
display: grid;
grid-template-columns: repeat(5, 1fr);
gap: 12px;
max-width: 320px;
position: relative; z-index: 1;
margin-bottom: 24px;
}

.ob-emoji-btn {
width: 56px; height: 56px;
display: flex; align-items: center; justify-content: center;
font-size: 28px;
background: var(–surface);
border: 2px solid var(–border);
border-radius: var(–radius-sm);
cursor: pointer;
transition: all 0.2s;
}

.ob-emoji-btn.active {
border-color: var(–accent);
background: rgba(0,229,160,0.1);
transform: scale(1.1);
}

.ob-step { color: var(–text3); font-size: 13px; margin-bottom: 24px; position: relative; z-index: 1; font-family: var(–mono); }

/* ─── Main Layout ─── */
.main {
display: flex;
flex-direction: column;
min-height: 100vh;
}

.content {
flex: 1;
overflow-y: auto;
padding-bottom: 80px;
}

/* ─── Header ─── */
.header {
display: flex;
align-items: center;
justify-content: space-between;
padding: 16px 20px;
border-bottom: 1px solid var(–border);
background: rgba(10,10,15,0.9);
backdrop-filter: blur(20px);
position: sticky; top: 0; z-index: 50;
}

.header-logo {
font-size: 24px;
font-weight: 800;
background: var(–gradient1);
-webkit-background-clip: text;
-webkit-text-fill-color: transparent;
letter-spacing: -1px;
}

.header-actions { display: flex; gap: 8px; }

.icon-btn {
width: 40px; height: 40px;
display: flex; align-items: center; justify-content: center;
background: var(–surface);
border: 1px solid var(–border);
border-radius: 50%;
color: var(–text2);
font-size: 18px;
cursor: pointer;
transition: all 0.2s;
position: relative;
}

.icon-btn:hover { background: var(–surface2); color: var(–text); }
.icon-btn .badge {
position: absolute; top: -2px; right: -2px;
width: 16px; height: 16px;
background: var(–danger);
border-radius: 50%;
font-size: 9px;
display: flex; align-items: center; justify-content: center;
color: white; font-weight: 700;
}

/* ─── Stories ─── */
.stories-bar {
display: flex;
gap: 14px;
padding: 16px 20px;
overflow-x: auto;
border-bottom: 1px solid var(–border);
scrollbar-width: none;
}

.stories-bar::-webkit-scrollbar { display: none; }

.story-avatar {
display: flex;
flex-direction: column;
align-items: center;
gap: 6px;
cursor: pointer;
flex-shrink: 0;
}

.story-ring {
width: 60px; height: 60px;
border-radius: 50%;
padding: 3px;
background: var(–gradient2);
display: flex; align-items: center; justify-content: center;
transition: transform 0.2s;
}

.story-ring:hover { transform: scale(1.08); }
.story-ring.add { background: var(–surface2); border: 2px dashed var(–border); padding: 0; }

.story-inner {
width: 100%; height: 100%;
border-radius: 50%;
background: var(–bg);
display: flex; align-items: center; justify-content: center;
font-size: 28px;
}

.story-ring.add .story-inner { font-size: 22px; color: var(–accent); }
.story-name { font-size: 11px; color: var(–text2); max-width: 64px; text-align: center; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }

/* ─── Story Viewer ─── */
.story-viewer {
position: fixed; inset: 0;
background: #000;
z-index: 200;
display: flex; flex-direction: column;
animation: fadeIn 0.3s ease;
}

@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

.story-progress { display: flex; gap: 4px; padding: 12px 16px 8px; }
.story-progress-bar { flex: 1; height: 3px; background: rgba(255,255,255,0.2); border-radius: 2px; overflow: hidden; }
.story-progress-fill { height: 100%; background: white; border-radius: 2px; transition: width 0.1s linear; }
.story-header { display: flex; align-items: center; gap: 10px; padding: 4px 16px 12px; }
.story-header-avatar { font-size: 28px; }
.story-header-name { font-weight: 600; font-size: 14px; }
.story-header-time { font-size: 12px; color: rgba(255,255,255,0.5); }
.story-close { margin-left: auto; font-size: 24px; color: white; cursor: pointer; background: none; border: none; }

.story-content {
flex: 1;
display: flex; flex-direction: column;
align-items: center; justify-content: center;
gap: 16px;
padding: 20px;
}

.story-emoji { font-size: 100px; }
.story-text { font-size: 28px; font-weight: 700; text-align: center; color: white; }
.story-tap-zones { position: absolute; inset: 0; display: flex; top: 100px; }
.story-tap-left, .story-tap-right { flex: 1; cursor: pointer; }

/* ─── Feed ─── */
.feed { padding: 8px 0; }

.post {
background: var(–surface);
margin: 8px 12px;
border-radius: var(–radius);
border: 1px solid var(–border);
overflow: hidden;
transition: transform 0.2s;
}

.post-header {
display: flex;
align-items: center;
gap: 10px;
padding: 14px 16px;
}

.post-avatar {
width: 40px; height: 40px;
border-radius: 50%;
display: flex; align-items: center; justify-content: center;
font-size: 20px;
flex-shrink: 0;
}

.post-user-info { flex: 1; }
.post-username { font-weight: 600; font-size: 14px; }
.post-time { font-size: 12px; color: var(–text3); }

.post-media {
width: 100%;
aspect-ratio: 4/3;
display: flex; align-items: center; justify-content: center;
font-size: 80px;
position: relative;
}

.post-media-badge {
position: absolute;
top: 12px; right: 12px;
background: rgba(0,0,0,0.6);
padding: 4px 10px;
border-radius: 20px;
font-size: 11px;
font-weight: 600;
color: white;
backdrop-filter: blur(8px);
}

.post-caption { padding: 14px 16px 8px; font-size: 14px; line-height: 1.5; }

.post-actions {
display: flex;
gap: 4px;
padding: 8px 12px 14px;
}

.post-action {
display: flex; align-items: center; gap: 6px;
padding: 8px 14px;
background: var(–surface2);
border: none;
border-radius: 20px;
color: var(–text2);
font-size: 13px;
font-family: var(–font);
cursor: pointer;
transition: all 0.2s;
}

.post-action:hover { background: var(–surface3); color: var(–text); }
.post-action.liked { color: var(–danger); background: rgba(255,71,87,0.1); }

/* ─── Create Post ─── */
.create-overlay {
position: fixed; inset: 0;
background: rgba(0,0,0,0.8);
backdrop-filter: blur(10px);
z-index: 100;
display: flex; align-items: flex-end;
animation: fadeIn 0.2s ease;
}

.create-sheet {
width: 100%;
max-width: 430px;
margin: 0 auto;
background: var(–surface);
border-radius: var(–radius) var(–radius) 0 0;
padding: 24px 20px 32px;
animation: slideUp 0.3s ease;
}

@keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }

.create-header {
display: flex;
align-items: center;
justify-content: space-between;
margin-bottom: 20px;
}

.create-title { font-size: 18px; font-weight: 700; }
.create-close { background: none; border: none; color: var(–text2); font-size: 24px; cursor: pointer; }

.create-textarea {
width: 100%;
min-height: 120px;
background: var(–surface2);
border: 1px solid var(–border);
border-radius: var(–radius-sm);
padding: 14px 16px;
color: var(–text);
font-family: var(–font);
font-size: 15px;
resize: none;
outline: none;
margin-bottom: 16px;
}

.create-textarea:focus { border-color: var(–accent); }
.create-textarea::placeholder { color: var(–text3); }

.create-media-row {
display: flex; gap: 10px; margin-bottom: 20px;
}

.create-media-btn {
flex: 1;
padding: 12px;
background: var(–surface2);
border: 1px dashed var(–border);
border-radius: var(–radius-sm);
color: var(–text2);
font-family: var(–font);
font-size: 13px;
cursor: pointer;
display: flex; align-items: center; justify-content: center; gap: 6px;
transition: all 0.2s;
}

.create-media-btn:hover { border-color: var(–accent); color: var(–accent); }

.create-submit {
width: 100%;
padding: 14px;
background: var(–gradient1);
border: none;
border-radius: var(–radius-sm);
color: #0A0A0F;
font-family: var(–font);
font-size: 15px;
font-weight: 700;
cursor: pointer;
transition: transform 0.2s;
}

.create-submit:hover { transform: translateY(-1px); }
.create-submit:disabled { opacity: 0.4; cursor: not-allowed; transform: none; }

/* ─── Messages ─── */
.messages-list { padding: 8px 0; }

.msg-item {
display: flex;
align-items: center;
gap: 12px;
padding: 14px 20px;
cursor: pointer;
transition: background 0.2s;
}

.msg-item:hover { background: var(–surface); }

.msg-avatar {
width: 50px; height: 50px;
border-radius: 50%;
display: flex; align-items: center; justify-content: center;
font-size: 24px;
flex-shrink: 0;
}

.msg-info { flex: 1; min-width: 0; }
.msg-name { font-weight: 600; font-size: 15px; }
.msg-preview { font-size: 13px; color: var(–text2); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.msg-time { font-size: 11px; color: var(–text3); font-family: var(–mono); }

.msg-unread-dot {
width: 10px; height: 10px;
background: var(–accent);
border-radius: 50%;
flex-shrink: 0;
}

/* ─── Chat View ─── */
.chat-view {
position: fixed; inset: 0;
background: var(–bg);
z-index: 100;
display: flex; flex-direction: column;
max-width: 430px;
margin: 0 auto;
animation: slideRight 0.25s ease;
}

@keyframes slideRight { from { transform: translateX(100%); } to { transform: translateX(0); } }

.chat-header {
display: flex; align-items: center; gap: 12px;
padding: 14px 16px;
border-bottom: 1px solid var(–border);
background: rgba(10,10,15,0.95);
backdrop-filter: blur(20px);
}

.chat-back {
background: none; border: none;
color: var(–text2); font-size: 22px;
cursor: pointer; padding: 4px;
}

.chat-header-avatar { font-size: 28px; }
.chat-header-name { font-weight: 600; font-size: 16px; flex: 1; }

.chat-messages {
flex: 1;
overflow-y: auto;
padding: 16px;
display: flex;
flex-direction: column;
gap: 8px;
}

.chat-bubble {
max-width: 80%;
padding: 10px 16px;
border-radius: 18px;
font-size: 14px;
line-height: 1.45;
animation: popIn 0.2s ease;
}

@keyframes popIn { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }

.chat-bubble.mine {
align-self: flex-end;
background: var(–accent2);
color: #0A0A0F;
border-bottom-right-radius: 4px;
}

.chat-bubble.theirs {
align-self: flex-start;
background: var(–surface2);
color: var(–text);
border-bottom-left-radius: 4px;
}

.chat-bubble.file-msg {
display: flex; align-items: center; gap: 8px;
background: var(–surface2);
border: 1px solid var(–border);
}

.chat-bubble.file-msg.mine { background: rgba(0,229,160,0.15); border-color: rgba(0,229,160,0.3); }

.file-icon { font-size: 24px; }
.file-details { flex: 1; }
.file-name { font-size: 13px; font-weight: 600; }
.file-size { font-size: 11px; color: var(–text3); }

.chat-input-bar {
display: flex; align-items: center; gap: 8px;
padding: 12px 16px;
border-top: 1px solid var(–border);
background: var(–surface);
}

.chat-attach {
width: 40px; height: 40px;
display: flex; align-items: center; justify-content: center;
background: var(–surface2);
border: 1px solid var(–border);
border-radius: 50%;
color: var(–text2);
font-size: 18px;
cursor: pointer;
flex-shrink: 0;
transition: all 0.2s;
position: relative;
overflow: hidden;
}

.chat-attach:hover { border-color: var(–accent); color: var(–accent); }
.chat-attach input { position: absolute; inset: 0; opacity: 0; cursor: pointer; }

.chat-input {
flex: 1;
padding: 10px 16px;
background: var(–surface2);
border: 1px solid var(–border);
border-radius: 20px;
color: var(–text);
font-family: var(–font);
font-size: 14px;
outline: none;
}

.chat-input:focus { border-color: var(–accent); }
.chat-input::placeholder { color: var(–text3); }

.chat-send {
width: 40px; height: 40px;
display: flex; align-items: center; justify-content: center;
background: var(–gradient1);
border: none;
border-radius: 50%;
color: #0A0A0F;
font-size: 18px;
cursor: pointer;
flex-shrink: 0;
transition: transform 0.2s;
}

.chat-send:hover { transform: scale(1.08); }

/* ─── Profile ─── */
.profile { padding: 24px 20px; }

.profile-hero {
display: flex; flex-direction: column; align-items: center;
padding: 32px 0 24px;
border-bottom: 1px solid var(–border);
margin-bottom: 24px;
}

.profile-avatar-lg {
width: 90px; height: 90px;
border-radius: 50%;
display: flex; align-items: center; justify-content: center;
font-size: 48px;
margin-bottom: 14px;
border: 3px solid var(–accent);
}

.profile-display-name { font-size: 22px; font-weight: 700; margin-bottom: 2px; }
.profile-handle { font-size: 14px; color: var(–text2); font-family: var(–mono); margin-bottom: 8px; }
.profile-bio { font-size: 14px; color: var(–text3); margin-bottom: 16px; }

.profile-stats {
display: flex; gap: 32px;
}

.profile-stat { text-align: center; }
.profile-stat-num { font-size: 20px; font-weight: 700; }
.profile-stat-label { font-size: 11px; color: var(–text3); text-transform: uppercase; letter-spacing: 1px; }

.profile-edit-btn {
width: 100%;
padding: 12px;
background: var(–surface);
border: 1px solid var(–border);
border-radius: var(–radius-sm);
color: var(–text);
font-family: var(–font);
font-size: 14px;
font-weight: 600;
cursor: pointer;
margin-top: 20px;
transition: all 0.2s;
}

.profile-edit-btn:hover { border-color: var(–accent); }

.profile-posts-grid {
display: grid;
grid-template-columns: repeat(3, 1fr);
gap: 4px;
}

.profile-post-tile {
aspect-ratio: 1;
border-radius: var(–radius-xs);
display: flex; align-items: center; justify-content: center;
font-size: 36px;
cursor: pointer;
transition: transform 0.2s;
}

.profile-post-tile:hover { transform: scale(1.04); }

.profile-section-title {
font-size: 13px;
font-weight: 600;
color: var(–text3);
text-transform: uppercase;
letter-spacing: 2px;
margin-bottom: 16px;
}

/* ─── Bottom Nav ─── */
.bottom-nav {
position: fixed;
bottom: 0;
left: 50%;
transform: translateX(-50%);
width: 100%;
max-width: 430px;
display: flex;
align-items: center;
justify-content: space-around;
padding: 10px 0 24px;
background: linear-gradient(to top, var(–bg) 60%, transparent);
z-index: 60;
}

.nav-item {
display: flex; flex-direction: column; align-items: center; gap: 4px;
cursor: pointer;
padding: 6px 16px;
border-radius: 12px;
transition: all 0.2s;
background: none;
border: none;
color: var(–text3);
font-family: var(–font);
}

.nav-item.active { color: var(–accent); }
.nav-item:hover { color: var(–text); }
.nav-icon { font-size: 22px; }
.nav-label { font-size: 10px; font-weight: 600; letter-spacing: 0.5px; }

.nav-create {
width: 48px; height: 48px;
background: var(–gradient1);
border: none;
border-radius: 50%;
font-size: 26px;
color: #0A0A0F;
cursor: pointer;
display: flex; align-items: center; justify-content: center;
transition: transform 0.2s, box-shadow 0.2s;
box-shadow: 0 4px 16px rgba(0,229,160,0.3);
margin-top: -16px;
}

.nav-create:hover { transform: scale(1.1); box-shadow: 0 6px 24px rgba(0,229,160,0.4); }

/* ─── Empty States ─── */
.empty-state {
display: flex; flex-direction: column; align-items: center; justify-content: center;
padding: 60px 20px;
text-align: center;
}

.empty-icon { font-size: 48px; margin-bottom: 16px; opacity: 0.5; }
.empty-text { font-size: 15px; color: var(–text3); }

/* ─── Create Story Sheet ─── */
.story-create-sheet {
width: 100%;
max-width: 430px;
margin: 0 auto;
background: var(–surface);
border-radius: var(–radius) var(–radius) 0 0;
padding: 24px 20px 32px;
animation: slideUp 0.3s ease;
}

.story-bg-grid {
display: grid;
grid-template-columns: repeat(3, 1fr);
gap: 10px;
margin-bottom: 16px;
}

.story-bg-option {
aspect-ratio: 3/4;
border-radius: var(–radius-sm);
cursor: pointer;
border: 2px solid transparent;
transition: all 0.2s;
display: flex; align-items: center; justify-content: center;
font-size: 24px; color: white;
}

.story-bg-option.active { border-color: var(–accent); transform: scale(1.04); }

/* ─── Scrollbar ─── */
::-webkit-scrollbar { width: 4px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: var(–surface3); border-radius: 4px; }
`;

// ─── AVATARS for choosing ───
const AVATAR_OPTIONS = [“😎”,“🔥”,“💀”,“🦋”,“🌸”,“🐉”,“👾”,“🎭”,“🌊”,“💎”];
const STORY_BGS = [
“linear-gradient(135deg, #FF6B6B, #ee5a24)”,
“linear-gradient(135deg, #A78BFA, #6D28D9)”,
“linear-gradient(135deg, #00E5A0, #00B4D8)”,
“linear-gradient(135deg, #F59E0B, #D97706)”,
“linear-gradient(135deg, #EC4899, #BE185D)”,
“linear-gradient(135deg, #3B82F6, #1D4ED8)”,
];

const MEDIA_COLORS = [
“linear-gradient(135deg, #1a1a2e, #16213e)”,
“linear-gradient(135deg, #0f0c29, #302b63)”,
“linear-gradient(135deg, #1a1a2e, #e94560)”,
“linear-gradient(135deg, #0f3460, #533483)”,
“linear-gradient(135deg, #2c003e, #512b58)”,
];

// ─── Component ───
export default function TipTap() {
const [loaded, setLoaded] = useState(false);
const [user, setUser] = useState(null); // null = onboarding
const [obStep, setObStep] = useState(0);
const [obName, setObName] = useState(””);
const [obUsername, setObUsername] = useState(””);
const [obAvatar, setObAvatar] = useState(””);
const [tab, setTab] = useState(“feed”);
const [posts, setPosts] = useState(DEMO_POSTS);
const [likedPosts, setLikedPosts] = useState(new Set());
const [stories, setStories] = useState(DEMO_STORIES);
const [viewingStory, setViewingStory] = useState(null);
const [storyIdx, setStoryIdx] = useState(0);
const [storyProgress, setStoryProgress] = useState(0);
const [messages, setMessages] = useState(DEMO_MESSAGES);
const [openChat, setOpenChat] = useState(null);
const [chatInput, setChatInput] = useState(””);
const [showCreate, setShowCreate] = useState(false);
const [createText, setCreateText] = useState(””);
const [showStoryCreate, setShowStoryCreate] = useState(false);
const [storyText, setStoryText] = useState(””);
const [storyBg, setStoryBg] = useState(STORY_BGS[0]);
const storyTimer = useRef(null);
const chatEndRef = useRef(null);

// Load persisted user
useEffect(() => {
(async () => {
const u = await loadState(“tiptap-user”, null);
const p = await loadState(“tiptap-posts”, null);
const m = await loadState(“tiptap-messages”, null);
if (u) setUser(u);
if (p) setPosts(p);
if (m) setMessages(m);
setLoaded(true);
})();
}, []);

// Save on change
useEffect(() => { if (loaded && user) saveState(“tiptap-user”, user); }, [user, loaded]);
useEffect(() => { if (loaded) saveState(“tiptap-posts”, posts); }, [posts, loaded]);
useEffect(() => { if (loaded) saveState(“tiptap-messages”, messages); }, [messages, loaded]);

// Auto-scroll chat
useEffect(() => { chatEndRef.current?.scrollIntoView({ behavior: “smooth” }); }, [openChat, messages]);

// Story auto-progress
useEffect(() => {
if (!viewingStory) return;
const story = viewingStory;
const item = story.items[storyIdx];
if (!item) { setViewingStory(null); return; }
setStoryProgress(0);
let elapsed = 0;
storyTimer.current = setInterval(() => {
elapsed += 50;
setStoryProgress((elapsed / 5000) * 100);
if (elapsed >= 5000) {
if (storyIdx < story.items.length - 1) setStoryIdx(i => i + 1);
else setViewingStory(null);
}
}, 50);
return () => clearInterval(storyTimer.current);
}, [viewingStory, storyIdx]);

function finishOnboarding() {
const newUser = {
id: “me”,
username: obUsername.toLowerCase().replace(/\s/g, “.”),
displayName: obName,
avatar: obAvatar,
bio: “”,
color: “#00E5A0”,
};
setUser(newUser);
}

function getUserById(id) {
if (id === “me” && user) return user;
return DEMO_USERS.find(u => u.id === id);
}

function toggleLike(postId) {
setLikedPosts(prev => {
const next = new Set(prev);
if (next.has(postId)) next.delete(postId);
else next.add(postId);
return next;
});
setPosts(prev => prev.map(p => p.id === postId ? { …p, likes: p.likes + (likedPosts.has(postId) ? -1 : 1) } : p));
}

function createPost() {
if (!createText.trim()) return;
const emoji = [“🎵”,“🎨”,“💡”,“🔥”,“✨”,“📸”,“🎬”,“🌟”][Math.floor(Math.random() * 8)];
const newPost = {
id: “p” + Date.now(),
userId: “me”,
type: “text”,
caption: createText,
media: Math.random() > 0.5 ? emoji : null,
timestamp: Date.now(),
likes: 0,
comments: 0,
};
setPosts(prev => [newPost, …prev]);
setCreateText(””);
setShowCreate(false);
}

function createStory() {
if (!storyText.trim()) return;
const emoji = [“✨”,“🎯”,“💫”,“⚡”,“🌈”,“🎪”][Math.floor(Math.random() * 6)];
const myStoryIdx = stories.findIndex(s => s.userId === “me”);
const newItem = { id: “s” + Date.now(), emoji, bg: storyBg, text: storyText, ts: Date.now() };
if (myStoryIdx >= 0) {
setStories(prev => prev.map((s, i) => i === myStoryIdx ? { …s, items: […s.items, newItem] } : s));
} else {
setStories(prev => [{ userId: “me”, items: [newItem] }, …prev]);
}
setStoryText(””);
setShowStoryCreate(false);
}

function sendMessage() {
if (!chatInput.trim() || !openChat) return;
const newMsg = { id: “m” + Date.now(), from: “me”, text: chatInput, ts: Date.now(), type: “text” };
setMessages(prev => ({ …prev, [openChat]: […(prev[openChat] || []), newMsg] }));
setChatInput(””);
}

function handleFileAttach(e) {
const file = e.target.files?.[0];
if (!file || !openChat) return;
const exts = { “image/png”: “📷”, “image/jpeg”: “📷”, “image/gif”: “🖼️”, “video/mp4”: “🎬”, “audio/mpeg”: “🎵”, “application/pdf”: “📄” };
const icon = exts[file.type] || “📎”;
const size = file.size > 1048576 ? (file.size / 1048576).toFixed(1) + “ MB” : (file.size / 1024).toFixed(0) + “ KB”;
const newMsg = { id: “m” + Date.now(), from: “me”, text: file.name, ts: Date.now(), type: “file”, fileIcon: icon, fileSize: size };
setMessages(prev => ({ …prev, [openChat]: […(prev[openChat] || []), newMsg] }));
e.target.value = “”;
}

if (!loaded) return <div className=“app” style={{ display: “flex”, alignItems: “center”, justifyContent: “center”, minHeight: “100vh” }}><div className="ob-logo">tip tap</div></div>;

// ─── ONBOARDING ───
if (!user) {
return (
<div className="app">
<style>{css}</style>
<div className="onboarding">
<div className="ob-logo">tip tap</div>
<div className="ob-tagline">share everything</div>
<div className="ob-step">step {obStep + 1} of 3</div>

```
      {obStep === 0 && (
        <>
          <div className="ob-input-group">
            <input className="ob-input" placeholder="your name" value={obName} onChange={e => setObName(e.target.value)} autoFocus />
          </div>
          <button className="ob-btn" disabled={!obName.trim()} onClick={() => setObStep(1)}>next →</button>
        </>
      )}

      {obStep === 1 && (
        <>
          <div className="ob-input-group">
            <input className="ob-input" placeholder="pick a username" value={obUsername} onChange={e => setObUsername(e.target.value)} autoFocus />
          </div>
          <button className="ob-btn" disabled={!obUsername.trim()} onClick={() => setObStep(2)}>next →</button>
        </>
      )}

      {obStep === 2 && (
        <>
          <div className="ob-emoji-grid">
            {AVATAR_OPTIONS.map(em => (
              <button key={em} className={`ob-emoji-btn ${obAvatar === em ? "active" : ""}`} onClick={() => setObAvatar(em)}>{em}</button>
            ))}
          </div>
          <button className="ob-btn" disabled={!obAvatar} onClick={finishOnboarding}>let's go 🚀</button>
        </>
      )}
    </div>
  </div>
);
```

}

// ─── STORY VIEWER ───
if (viewingStory) {
const storyUser = getUserById(viewingStory.userId);
const item = viewingStory.items[storyIdx];
if (!item) { setViewingStory(null); return null; }
return (
<div className="app">
<style>{css}</style>
<div className=“story-viewer” style={{ background: item.bg }}>
<div className="story-progress">
{viewingStory.items.map((_, i) => (
<div key={i} className="story-progress-bar">
<div className=“story-progress-fill” style={{ width: i < storyIdx ? “100%” : i === storyIdx ? `${storyProgress}%` : “0%” }} />
</div>
))}
</div>
<div className="story-header">
<span className="story-header-avatar">{storyUser?.avatar}</span>
<span className="story-header-name">{storyUser?.displayName}</span>
<span className="story-header-time">{timeAgo(item.ts)}</span>
<button className=“story-close” onClick={() => setViewingStory(null)}>✕</button>
</div>
<div className="story-content">
<div className="story-emoji">{item.emoji}</div>
<div className="story-text">{item.text}</div>
</div>
<div className="story-tap-zones">
<div className=“story-tap-left” onClick={() => { if (storyIdx > 0) setStoryIdx(i => i - 1); }} />
<div className=“story-tap-right” onClick={() => {
if (storyIdx < viewingStory.items.length - 1) setStoryIdx(i => i + 1);
else setViewingStory(null);
}} />
</div>
</div>
</div>
);
}

// ─── CHAT VIEW ───
if (openChat) {
const chatUser = getUserById(openChat);
const chatMsgs = messages[openChat] || [];
return (
<div className="app">
<style>{css}</style>
<div className="chat-view">
<div className="chat-header">
<button className=“chat-back” onClick={() => setOpenChat(null)}>←</button>
<span className="chat-header-avatar">{chatUser?.avatar}</span>
<span className="chat-header-name">{chatUser?.displayName}</span>
</div>
<div className="chat-messages">
{chatMsgs.map(msg => (
msg.type === “file” ? (
<div key={msg.id} className={`chat-bubble file-msg ${msg.from === "me" ? "mine" : "theirs"}`}>
<span className="file-icon">{msg.fileIcon}</span>
<div className="file-details">
<div className="file-name">{msg.text}</div>
<div className="file-size">{msg.fileSize}</div>
</div>
</div>
) : (
<div key={msg.id} className={`chat-bubble ${msg.from === "me" ? "mine" : "theirs"}`}>{msg.text}</div>
)
))}
<div ref={chatEndRef} />
</div>
<div className="chat-input-bar">
<label className="chat-attach">
📎
<input type="file" accept="image/*,video/*,audio/*,.pdf,.doc,.docx,.zip" onChange={handleFileAttach} />
</label>
<input className=“chat-input” placeholder=“message…” value={chatInput} onChange={e => setChatInput(e.target.value)} onKeyDown={e => e.key === “Enter” && sendMessage()} autoFocus />
<button className="chat-send" onClick={sendMessage}>↑</button>
</div>
</div>
</div>
);
}

// ─── MAIN APP ───
const conversationUsers = DEMO_USERS.filter(u => messages[u.id]?.length);
const myPosts = posts.filter(p => p.userId === “me”);

return (
<div className="app">
<style>{css}</style>

```
  {/* Header */}
  <div className="header">
    <div className="header-logo">tip tap</div>
    <div className="header-actions">
      <button className="icon-btn" onClick={() => setTab("messages")}>
        💬
        {conversationUsers.length > 0 && <span className="badge">{conversationUsers.length}</span>}
      </button>
    </div>
  </div>

  <div className="content">
    {/* ─── FEED TAB ─── */}
    {tab === "feed" && (
      <>
        {/* Stories */}
        <div className="stories-bar">
          <div className="story-avatar" onClick={() => setShowStoryCreate(true)}>
            <div className="story-ring add"><div className="story-inner">+</div></div>
            <span className="story-name">add story</span>
          </div>
          {stories.map(story => {
            const su = getUserById(story.userId);
            return (
              <div key={story.userId} className="story-avatar" onClick={() => { setViewingStory(story); setStoryIdx(0); }}>
                <div className="story-ring"><div className="story-inner">{su?.avatar}</div></div>
                <span className="story-name">{su?.displayName}</span>
              </div>
            );
          })}
        </div>

        {/* Feed */}
        <div className="feed">
          {posts.map(post => {
            const pu = getUserById(post.userId);
            const isLiked = likedPosts.has(post.id);
            return (
              <div key={post.id} className="post">
                <div className="post-header">
                  <div className="post-avatar" style={{ background: pu?.color || "#333" }}>{pu?.avatar}</div>
                  <div className="post-user-info">
                    <div className="post-username">{pu?.username}</div>
                    <div className="post-time">{timeAgo(post.timestamp)}</div>
                  </div>
                </div>
                {post.media && (
                  <div className="post-media" style={{ background: MEDIA_COLORS[Math.abs(post.id.charCodeAt(1)) % MEDIA_COLORS.length] }}>
                    {post.media}
                    {post.type === "video" && <span className="post-media-badge">▶ VIDEO</span>}
                  </div>
                )}
                <div className="post-caption">{post.caption}</div>
                <div className="post-actions">
                  <button className={`post-action ${isLiked ? "liked" : ""}`} onClick={() => toggleLike(post.id)}>
                    {isLiked ? "♥" : "♡"} {post.likes + (isLiked ? 1 : 0)}
                  </button>
                  <button className="post-action">💬 {post.comments}</button>
                  <button className="post-action">↗</button>
                </div>
              </div>
            );
          })}
        </div>
      </>
    )}

    {/* ─── MESSAGES TAB ─── */}
    {tab === "messages" && (
      <div className="messages-list">
        {DEMO_USERS.map(u => {
          const uMsgs = messages[u.id] || [];
          const lastMsg = uMsgs[uMsgs.length - 1];
          return (
            <div key={u.id} className="msg-item" onClick={() => setOpenChat(u.id)}>
              <div className="msg-avatar" style={{ background: u.color }}>{u.avatar}</div>
              <div className="msg-info">
                <div className="msg-name">{u.displayName}</div>
                <div className="msg-preview">{lastMsg ? (lastMsg.type === "file" ? `📎 ${lastMsg.text}` : lastMsg.text) : "start a conversation"}</div>
              </div>
              {lastMsg && <span className="msg-time">{timeAgo(lastMsg.ts)}</span>}
              {lastMsg?.from !== "me" && <div className="msg-unread-dot" />}
            </div>
          );
        })}
      </div>
    )}

    {/* ─── PROFILE TAB ─── */}
    {tab === "profile" && (
      <div className="profile">
        <div className="profile-hero">
          <div className="profile-avatar-lg" style={{ background: "var(--surface2)" }}>{user.avatar}</div>
          <div className="profile-display-name">{user.displayName}</div>
          <div className="profile-handle">@{user.username}</div>
          <div className="profile-bio">{user.bio || "no bio yet"}</div>
          <div className="profile-stats">
            <div className="profile-stat"><div className="profile-stat-num">{myPosts.length}</div><div className="profile-stat-label">posts</div></div>
            <div className="profile-stat"><div className="profile-stat-num">0</div><div className="profile-stat-label">followers</div></div>
            <div className="profile-stat"><div className="profile-stat-num">{DEMO_USERS.length}</div><div className="profile-stat-label">following</div></div>
          </div>
        </div>
        <div className="profile-section-title">your posts</div>
        {myPosts.length > 0 ? (
          <div className="profile-posts-grid">
            {myPosts.map((p, i) => (
              <div key={p.id} className="profile-post-tile" style={{ background: MEDIA_COLORS[i % MEDIA_COLORS.length] }}>
                {p.media || "📝"}
              </div>
            ))}
          </div>
        ) : (
          <div className="empty-state">
            <div className="empty-icon">📸</div>
            <div className="empty-text">no posts yet — tap + to create one</div>
          </div>
        )}
      </div>
    )}
  </div>

  {/* ─── Create Post Sheet ─── */}
  {showCreate && (
    <div className="create-overlay" onClick={e => e.target === e.currentTarget && setShowCreate(false)}>
      <div className="create-sheet">
        <div className="create-header">
          <div className="create-title">new post</div>
          <button className="create-close" onClick={() => setShowCreate(false)}>✕</button>
        </div>
        <textarea className="create-textarea" placeholder="what's on your mind?" value={createText} onChange={e => setCreateText(e.target.value)} autoFocus />
        <div className="create-media-row">
          <button className="create-media-btn">📷 photo</button>
          <button className="create-media-btn">🎬 video</button>
          <button className="create-media-btn">📎 file</button>
        </div>
        <button className="create-submit" disabled={!createText.trim()} onClick={createPost}>post it →</button>
      </div>
    </div>
  )}

  {/* ─── Create Story Sheet ─── */}
  {showStoryCreate && (
    <div className="create-overlay" onClick={e => e.target === e.currentTarget && setShowStoryCreate(false)}>
      <div className="story-create-sheet">
        <div className="create-header">
          <div className="create-title">new story</div>
          <button className="create-close" onClick={() => setShowStoryCreate(false)}>✕</button>
        </div>
        <div className="story-bg-grid">
          {STORY_BGS.map(bg => (
            <div key={bg} className={`story-bg-option ${storyBg === bg ? "active" : ""}`} style={{ background: bg }} onClick={() => setStoryBg(bg)}>
              ✦
            </div>
          ))}
        </div>
        <textarea className="create-textarea" placeholder="story text..." value={storyText} onChange={e => setStoryText(e.target.value)} style={{ minHeight: "80px" }} autoFocus />
        <button className="create-submit" disabled={!storyText.trim()} onClick={createStory}>add story →</button>
      </div>
    </div>
  )}

  {/* ─── Bottom Nav ─── */}
  <div className="bottom-nav">
    <button className={`nav-item ${tab === "feed" ? "active" : ""}`} onClick={() => setTab("feed")}>
      <span className="nav-icon">🏠</span>
      <span className="nav-label">feed</span>
    </button>
    <button className={`nav-item ${tab === "messages" ? "active" : ""}`} onClick={() => setTab("messages")}>
      <span className="nav-icon">💬</span>
      <span className="nav-label">messages</span>
    </button>
    <button className="nav-create" onClick={() => setShowCreate(true)}>+</button>
    <button className={`nav-item ${tab === "discover" ? "active" : ""}`} onClick={() => setTab("discover")}>
      <span className="nav-icon">🔍</span>
      <span className="nav-label">discover</span>
    </button>
    <button className={`nav-item ${tab === "profile" ? "active" : ""}`} onClick={() => setTab("profile")}>
      <span className="nav-icon">👤</span>
      <span className="nav-label">profile</span>
    </button>
  </div>
</div>
```

);
}
