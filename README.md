import { useState, useEffect, useRef } from “react”;

// ─── Font ───
const _f = document.createElement(“link”);
_f.href = “https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;500;600;700;800;900&family=Space+Mono:wght@400;700&display=swap”;
_f.rel = “stylesheet”;
if (!document.querySelector(‘link[href*=“Outfit”]’)) document.head.appendChild(_f);

// ─── Constants ───
const ZODIAC = [
{ sign: “aries”, symbol: “\u2648”, dates: “Mar 21 – Apr 19”, element: “fire” },
{ sign: “taurus”, symbol: “\u2649”, dates: “Apr 20 – May 20”, element: “earth” },
{ sign: “gemini”, symbol: “\u264A”, dates: “May 21 – Jun 20”, element: “air” },
{ sign: “cancer”, symbol: “\u264B”, dates: “Jun 21 – Jul 22”, element: “water” },
{ sign: “leo”, symbol: “\u264C”, dates: “Jul 23 – Aug 22”, element: “fire” },
{ sign: “virgo”, symbol: “\u264D”, dates: “Aug 23 – Sep 22”, element: “earth” },
{ sign: “libra”, symbol: “\u264E”, dates: “Sep 23 – Oct 22”, element: “air” },
{ sign: “scorpio”, symbol: “\u264F”, dates: “Oct 23 – Nov 21”, element: “water” },
{ sign: “sagittarius”, symbol: “\u2650”, dates: “Nov 22 – Dec 21”, element: “fire” },
{ sign: “capricorn”, symbol: “\u2651”, dates: “Dec 22 – Jan 19”, element: “earth” },
{ sign: “aquarius”, symbol: “\u2652”, dates: “Jan 20 – Feb 18”, element: “air” },
{ sign: “pisces”, symbol: “\u2653”, dates: “Feb 19 – Mar 20”, element: “water” },
];

const PLANS = [
{ id: “free”, name: “Free”, price: 0, badge: “”, features: [“5 posts/day”, “View stories”, “Basic chat”], color: “#666677” },
{ id: “pro”, name: “Pro”, price: 4.99, badge: “POPULAR”, features: [“Unlimited posts”, “Tappy AI assistant”, “Daily AI horoscope”, “File sharing in DMs”, “Create stories”], color: “#00E5A0” },
{ id: “max”, name: “Max”, price: 9.99, badge: “BEST VALUE”, features: [“Everything in Pro”, “Unlimited Tappy AI”, “Custom themes”, “Analytics dashboard”, “Verified badge”], color: “#A78BFA” },
];

const USERS = [
{ id: “u1”, username: “nova.lens”, displayName: “Nova”, avatar: “\ud83c\udf0c”, bio: “chasing light”, color: “#FF6B6B”, zodiac: “leo” },
{ id: “u2”, username: “byte.wolf”, displayName: “Byte”, avatar: “\ud83d\udc3a”, bio: “code & chill”, color: “#4ECDC4”, zodiac: “virgo” },
{ id: “u3”, username: “luna.wav”, displayName: “Luna”, avatar: “\ud83c\udf19”, bio: “sound designer”, color: “#A78BFA”, zodiac: “pisces” },
{ id: “u4”, username: “pixel.riot”, displayName: “Pixel”, avatar: “\ud83c\udfae”, bio: “game dev”, color: “#F59E0B”, zodiac: “aries” },
{ id: “u5”, username: “drift.mp4”, displayName: “Drift”, avatar: “\ud83c\udfd4\ufe0f”, bio: “motion graphics”, color: “#EC4899”, zodiac: “sagittarius” },
];

const SEED_POSTS = [
{ id: “p1”, userId: “u1”, caption: “golden hour never misses”, media: “\ud83c\udf05”, timestamp: Date.now() - 3600000, likes: 42, comments: 7, type: “photo” },
{ id: “p2”, userId: “u2”, caption: “just shipped v2.0 \u2014 the codebase is clean and I’m not”, media: null, timestamp: Date.now() - 7200000, likes: 89, comments: 15, type: “text” },
{ id: “p3”, userId: “u3”, caption: “new sample pack dropping friday”, media: “\ud83c\udfb5”, timestamp: Date.now() - 18000000, likes: 156, comments: 23, type: “photo” },
{ id: “p4”, userId: “u4”, caption: “gameplay sneak peek”, media: “\ud83c\udfac”, timestamp: Date.now() - 36000000, likes: 234, comments: 31, type: “video” },
{ id: “p5”, userId: “u5”, caption: “this transition took 6 hours but worth it”, media: “\u2728”, timestamp: Date.now() - 72000000, likes: 67, comments: 9, type: “photo” },
];

const SEED_STORIES = [
{ userId: “u1”, items: [{ id: “s1”, emoji: “\ud83c\udfd4\ufe0f”, bg: “linear-gradient(135deg,#FF6B6B,#ee5a24)”, text: “hiking day!”, ts: Date.now() - 1800000 }] },
{ userId: “u3”, items: [{ id: “s2”, emoji: “\ud83c\udfb9”, bg: “linear-gradient(135deg,#A78BFA,#6D28D9)”, text: “studio session”, ts: Date.now() - 3600000 }] },
{ userId: “u5”, items: [{ id: “s4”, emoji: “\ud83c\udfa8”, bg: “linear-gradient(135deg,#EC4899,#BE185D)”, text: “new project wip”, ts: Date.now() - 5400000 }] },
];

const SEED_MSGS = {
u1: [{ id: “m1”, from: “u1”, text: “yo check this shot”, ts: Date.now() - 60000, type: “text” }, { id: “m2”, from: “me”, text: “insane!! where was that?”, ts: Date.now() - 50000, type: “text” }],
u3: [{ id: “m4”, from: “u3”, text: “collab?”, ts: Date.now() - 120000, type: “text” }, { id: “m5”, from: “me”, text: “let’s do it”, ts: Date.now() - 110000, type: “text” }],
};

const AVATARS = [”\ud83d\ude0e”, “\ud83d\udd25”, “\ud83d\udc80”, “\ud83e\udd8b”, “\ud83c\udf38”, “\ud83d\udc09”, “\ud83d\udc7e”, “\ud83c\udfad”, “\ud83c\udf0a”, “\ud83d\udc8e”];
const STORY_BGS = [“linear-gradient(135deg,#FF6B6B,#ee5a24)”, “linear-gradient(135deg,#A78BFA,#6D28D9)”, “linear-gradient(135deg,#00E5A0,#00B4D8)”, “linear-gradient(135deg,#F59E0B,#D97706)”, “linear-gradient(135deg,#EC4899,#BE185D)”, “linear-gradient(135deg,#3B82F6,#1D4ED8)”];
const MEDIA_BG = [“linear-gradient(135deg,#1a1a2e,#16213e)”, “linear-gradient(135deg,#0f0c29,#302b63)”, “linear-gradient(135deg,#1a1a2e,#e94560)”, “linear-gradient(135deg,#0f3460,#533483)”, “linear-gradient(135deg,#2c003e,#512b58)”];
const ELEM_BG = { fire: “linear-gradient(135deg,#FF6B6B,#ee5a24)”, water: “linear-gradient(135deg,#4ECDC4,#0077B6)”, earth: “linear-gradient(135deg,#8B9A46,#5C4B28)”, air: “linear-gradient(135deg,#A78BFA,#6D28D9)” };

function timeAgo(ts) { const d = Date.now() - ts; if (d < 60000) return “now”; if (d < 3600000) return Math.floor(d / 60000) + “m”; if (d < 86400000) return Math.floor(d / 3600000) + “h”; return Math.floor(d / 86400000) + “d”; }
function daysLeft(start) { if (!start) return 0; return Math.max(0, 7 - Math.floor((Date.now() - start) / 86400000)); }
function fmtCard(v) { return v.replace(/\D/g, “”).replace(/(\d{4})(?=\d)/g, “$1 “).slice(0, 19); }
function fmtExp(v) { const d = v.replace(/\D/g, “”).slice(0, 4); return d.length >= 3 ? d.slice(0, 2) + “/” + d.slice(2) : d; }

// ─── Styles (inline object) ───
const c = {
app: { maxWidth: 430, margin: “0 auto”, minHeight: “100vh”, background: “#0A0A0F”, fontFamily: “‘Outfit’,sans-serif”, color: “#EEEEF0”, position: “relative”, display: “flex”, flexDirection: “column” },
// Onboarding
ob: { minHeight: “100vh”, display: “flex”, flexDirection: “column”, alignItems: “center”, justifyContent: “center”, padding: “40px 24px”, textAlign: “center” },
obLogo: { fontSize: 56, fontWeight: 900, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, WebkitBackgroundClip: “text”, WebkitTextFillColor: “transparent”, letterSpacing: -2, marginBottom: 8 },
obTag: { color: “#9999AA”, fontSize: 16, fontWeight: 300, marginBottom: 48, letterSpacing: 2, textTransform: “uppercase” },
obStep: { color: “#666677”, fontSize: 13, marginBottom: 24, fontFamily: “‘Space Mono’,monospace” },
inp: { width: “100%”, maxWidth: 320, padding: “16px 20px”, background: “#13131A”, border: “1px solid #2A2A3A”, borderRadius: 16, color: “#EEEEF0”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 16, outline: “none”, marginBottom: 16 },
btn: { width: “100%”, maxWidth: 320, padding: 16, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, border: “none”, borderRadius: 16, color: “#0A0A0F”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 16, fontWeight: 700, cursor: “pointer”, marginTop: 8 },
btnOff: { opacity: 0.4, cursor: “not-allowed” },
emoGrid: { display: “grid”, gridTemplateColumns: “repeat(5,1fr)”, gap: 12, maxWidth: 320, marginBottom: 24 },
emoBtn: (on) => ({ width: 56, height: 56, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 28, background: on ? “rgba(0,229,160,.1)” : “#13131A”, border: `2px solid ${on ? "#00E5A0" : "#2A2A3A"}`, borderRadius: 10, cursor: “pointer”, transform: on ? “scale(1.1)” : “scale(1)”, transition: “all .2s” }),
zodGrid: { display: “grid”, gridTemplateColumns: “repeat(4,1fr)”, gap: 8, maxWidth: 320, marginBottom: 24 },
zodBtn: (on) => ({ display: “flex”, flexDirection: “column”, alignItems: “center”, gap: 4, padding: “12px 4px”, background: on ? “rgba(0,229,160,.1)” : “#13131A”, border: `2px solid ${on ? "#00E5A0" : "#2A2A3A"}`, borderRadius: 10, cursor: “pointer”, color: on ? “#EEEEF0” : “#9999AA”, transform: on ? “scale(1.05)” : “scale(1)”, transition: “all .2s” }),
// Header
hdr: { display: “flex”, alignItems: “center”, justifyContent: “space-between”, padding: “16px 20px”, borderBottom: “1px solid #2A2A3A”, background: “rgba(10,10,15,.9)”, backdropFilter: “blur(20px)”, position: “sticky”, top: 0, zIndex: 50 },
hdrLogo: { fontSize: 24, fontWeight: 800, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, WebkitBackgroundClip: “text”, WebkitTextFillColor: “transparent”, letterSpacing: -1 },
ibtn: { width: 40, height: 40, display: “flex”, alignItems: “center”, justifyContent: “center”, background: “#13131A”, border: “1px solid #2A2A3A”, borderRadius: “50%”, color: “#9999AA”, fontSize: 18, cursor: “pointer”, position: “relative” },
bdg: { position: “absolute”, top: -2, right: -2, width: 16, height: 16, background: “#FF4757”, borderRadius: “50%”, fontSize: 9, display: “flex”, alignItems: “center”, justifyContent: “center”, color: “white”, fontWeight: 700 },
subBadge: (plan) => ({ display: “inline-flex”, alignItems: “center”, fontSize: 10, fontWeight: 700, padding: “3px 10px”, borderRadius: 12, textTransform: “uppercase”, letterSpacing: 1, background: plan === “max” ? “rgba(167,139,250,.15)” : “rgba(0,229,160,.15)”, color: plan === “max” ? “#A78BFA” : “#00E5A0” }),
// Trial
trial: (expired) => ({ margin: “8px 12px”, padding: “12px 16px”, borderRadius: 10, background: expired ? “rgba(255,71,87,.08)” : “linear-gradient(135deg,rgba(0,229,160,.15),rgba(0,180,216,.1))”, border: `1px solid ${expired ? "rgba(255,71,87,.3)" : "rgba(0,229,160,.2)"}`, display: “flex”, alignItems: “center”, gap: 10, cursor: “pointer” }),
// Stories
storiesBar: { display: “flex”, gap: 14, padding: “16px 20px”, overflowX: “auto”, borderBottom: “1px solid #2A2A3A”, WebkitOverflowScrolling: “touch” },
stAv: { display: “flex”, flexDirection: “column”, alignItems: “center”, gap: 6, cursor: “pointer”, flexShrink: 0 },
stRing: (add) => ({ width: 60, height: 60, borderRadius: “50%”, padding: add ? 0 : 3, background: add ? “#1C1C26” : “linear-gradient(135deg,#A78BFA,#EC4899)”, border: add ? “2px dashed #2A2A3A” : “none”, display: “flex”, alignItems: “center”, justifyContent: “center” }),
stInner: (add) => ({ width: “100%”, height: “100%”, borderRadius: “50%”, background: “#0A0A0F”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: add ? 22 : 28, color: add ? “#00E5A0” : undefined }),
stNm: { fontSize: 11, color: “#9999AA”, maxWidth: 64, textAlign: “center”, overflow: “hidden”, textOverflow: “ellipsis”, whiteSpace: “nowrap” },
// Story Viewer
sv: (bg) => ({ position: “fixed”, inset: 0, zIndex: 200, display: “flex”, flexDirection: “column”, background: bg }),
svProg: { display: “flex”, gap: 4, padding: “12px 16px 8px” },
svBar: { flex: 1, height: 3, background: “rgba(255,255,255,.2)”, borderRadius: 2, overflow: “hidden” },
svFill: (w) => ({ height: “100%”, background: “white”, borderRadius: 2, width: w }),
svHd: { display: “flex”, alignItems: “center”, gap: 10, padding: “4px 16px 12px” },
svClose: { marginLeft: “auto”, fontSize: 24, color: “white”, cursor: “pointer”, background: “none”, border: “none” },
svBody: { flex: 1, display: “flex”, flexDirection: “column”, alignItems: “center”, justifyContent: “center”, gap: 16, padding: 20 },
svTaps: { position: “absolute”, inset: 0, display: “flex”, top: 100 },
svTap: { flex: 1, cursor: “pointer” },
// Horoscope
horo: (bg) => ({ margin: “12px 12px 4px”, borderRadius: 16, overflow: “hidden”, background: bg }),
horoIn: { padding: 20, position: “relative”, zIndex: 1 },
horoHd: { display: “flex”, alignItems: “center”, gap: 10, marginBottom: 12 },
horoSym: { width: 44, height: 44, borderRadius: “50%”, background: “rgba(255,255,255,.15)”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 22 },
horoTxt: { fontSize: 14, lineHeight: 1.6, color: “rgba(255,255,255,.9)” },
horoRef: { background: “rgba(255,255,255,.15)”, border: “none”, color: “rgba(255,255,255,.7)”, fontSize: 12, fontFamily: “‘Outfit’,sans-serif”, padding: “6px 14px”, borderRadius: 20, cursor: “pointer”, marginTop: 12, display: “inline-block” },
horoLoad: { display: “flex”, alignItems: “center”, justifyContent: “center”, gap: 8, padding: “12px 0”, color: “rgba(255,255,255,.6)”, fontSize: 13 },
// Post
post: { background: “#13131A”, margin: “8px 12px”, borderRadius: 16, border: “1px solid #2A2A3A”, overflow: “hidden” },
postHd: { display: “flex”, alignItems: “center”, gap: 10, padding: “14px 16px” },
postAv: (clr) => ({ width: 40, height: 40, borderRadius: “50%”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 20, flexShrink: 0, background: clr || “#333” }),
postMedia: (bg) => ({ width: “100%”, aspectRatio: “4/3”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 80, position: “relative”, background: bg }),
postCap: { padding: “14px 16px 8px”, fontSize: 14, lineHeight: 1.5 },
postActs: { display: “flex”, gap: 4, padding: “8px 12px 14px” },
postAct: (liked) => ({ display: “flex”, alignItems: “center”, gap: 6, padding: “8px 14px”, background: liked ? “rgba(255,71,87,.1)” : “#1C1C26”, border: “none”, borderRadius: 20, color: liked ? “#FF4757” : “#9999AA”, fontSize: 13, fontFamily: “‘Outfit’,sans-serif”, cursor: “pointer” }),
// Overlay / Sheet
overlay: { position: “fixed”, inset: 0, background: “rgba(0,0,0,.8)”, backdropFilter: “blur(10px)”, zIndex: 100, display: “flex”, alignItems: “flex-end”, justifyContent: “center” },
sheet: { width: “100%”, maxWidth: 430, background: “#13131A”, borderRadius: “16px 16px 0 0”, padding: “24px 20px 32px” },
shHd: { display: “flex”, alignItems: “center”, justifyContent: “space-between”, marginBottom: 20 },
shClose: { background: “none”, border: “none”, color: “#9999AA”, fontSize: 24, cursor: “pointer” },
txa: { width: “100%”, minHeight: 120, background: “#1C1C26”, border: “1px solid #2A2A3A”, borderRadius: 10, padding: “14px 16px”, color: “#EEEEF0”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 15, resize: “none”, outline: “none”, marginBottom: 16, boxSizing: “border-box” },
mRow: { display: “flex”, gap: 10, marginBottom: 20 },
mBtn: { flex: 1, padding: 12, background: “#1C1C26”, border: “1px dashed #2A2A3A”, borderRadius: 10, color: “#9999AA”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 13, cursor: “pointer”, display: “flex”, alignItems: “center”, justifyContent: “center”, gap: 6 },
subBtn: (disabled) => ({ width: “100%”, padding: 14, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, border: “none”, borderRadius: 10, color: “#0A0A0F”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 15, fontWeight: 700, cursor: disabled ? “not-allowed” : “pointer”, opacity: disabled ? 0.4 : 1 }),
// Messages
msgItem: { display: “flex”, alignItems: “center”, gap: 12, padding: “14px 20px”, cursor: “pointer” },
msgAv: (clr) => ({ width: 50, height: 50, borderRadius: “50%”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 24, flexShrink: 0, background: clr }),
// Chat
chat: { position: “fixed”, inset: 0, background: “#0A0A0F”, zIndex: 100, display: “flex”, flexDirection: “column”, maxWidth: 430, margin: “0 auto” },
chatHd: { display: “flex”, alignItems: “center”, gap: 12, padding: “14px 16px”, borderBottom: “1px solid #2A2A3A”, background: “rgba(10,10,15,.95)” },
chatBk: { background: “none”, border: “none”, color: “#9999AA”, fontSize: 22, cursor: “pointer”, padding: 4 },
chatMsgs: { flex: 1, overflowY: “auto”, padding: 16, display: “flex”, flexDirection: “column”, gap: 8 },
bub: (mine) => ({ alignSelf: mine ? “flex-end” : “flex-start”, maxWidth: “80%”, padding: “10px 16px”, borderRadius: mine ? “18px 18px 4px 18px” : “18px 18px 18px 4px”, fontSize: 14, lineHeight: 1.45, background: mine ? “#00C48C” : “#1C1C26”, color: mine ? “#0A0A0F” : “#EEEEF0” }),
bubFile: (mine) => ({ alignSelf: mine ? “flex-end” : “flex-start”, maxWidth: “80%”, padding: “10px 16px”, borderRadius: 18, fontSize: 14, display: “flex”, alignItems: “center”, gap: 8, background: mine ? “rgba(0,229,160,.15)” : “#1C1C26”, border: `1px solid ${mine ? "rgba(0,229,160,.3)" : "#2A2A3A"}` }),
chatBar: { display: “flex”, alignItems: “center”, gap: 8, padding: “12px 16px”, borderTop: “1px solid #2A2A3A”, background: “#13131A” },
chatAtt: { width: 40, height: 40, display: “flex”, alignItems: “center”, justifyContent: “center”, background: “#1C1C26”, border: “1px solid #2A2A3A”, borderRadius: “50%”, color: “#9999AA”, fontSize: 18, cursor: “pointer”, flexShrink: 0, position: “relative”, overflow: “hidden” },
chatInp: { flex: 1, padding: “10px 16px”, background: “#1C1C26”, border: “1px solid #2A2A3A”, borderRadius: 20, color: “#EEEEF0”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 14, outline: “none” },
chatSnd: { width: 40, height: 40, display: “flex”, alignItems: “center”, justifyContent: “center”, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, border: “none”, borderRadius: “50%”, color: “#0A0A0F”, fontSize: 18, cursor: “pointer”, flexShrink: 0 },
// Profile
profHero: { display: “flex”, flexDirection: “column”, alignItems: “center”, padding: “32px 0 24px”, borderBottom: “1px solid #2A2A3A”, marginBottom: 24 },
profAv: { width: 90, height: 90, borderRadius: “50%”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 48, marginBottom: 14, border: “3px solid #00E5A0”, background: “#1C1C26” },
profStats: { display: “flex”, gap: 32 },
profGrid: { display: “grid”, gridTemplateColumns: “repeat(3,1fr)”, gap: 4 },
profTile: (bg) => ({ aspectRatio: “1”, borderRadius: 6, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 36, cursor: “pointer”, background: bg }),
// Payment
payOv: { position: “fixed”, inset: 0, background: “rgba(0,0,0,.85)”, backdropFilter: “blur(12px)”, zIndex: 150, display: “flex”, alignItems: “center”, justifyContent: “center”, padding: 16 },
payCard: { width: “100%”, maxWidth: 400, background: “#13131A”, borderRadius: 16, overflow: “hidden”, maxHeight: “90vh”, overflowY: “auto” },
payHdr: { padding: “20px 20px 16px”, display: “flex”, alignItems: “center”, justifyContent: “space-between” },
plan: (sel) => ({ padding: 16, borderRadius: 10, border: `2px solid ${sel ? "#00E5A0" : "#2A2A3A"}`, cursor: “pointer”, background: sel ? “rgba(0,229,160,.05)” : “transparent”, transition: “all .2s” }),
stripeInp: { padding: “12px 14px”, background: “#1C1C26”, border: “1px solid #2A2A3A”, borderRadius: 8, color: “#EEEEF0”, fontFamily: “‘Space Mono’,monospace”, fontSize: 14, outline: “none”, width: “100%”, boxSizing: “border-box” },
applePay: { width: “100%”, padding: 14, background: “#000”, border: “none”, borderRadius: 10, color: “white”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 15, fontWeight: 600, cursor: “pointer”, display: “flex”, alignItems: “center”, justifyContent: “center”, gap: 8, marginBottom: 10 },
stripePay: (disabled) => ({ width: “100%”, padding: 14, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, border: “none”, borderRadius: 10, color: “#0A0A0F”, fontFamily: “‘Outfit’,sans-serif”, fontSize: 15, fontWeight: 700, cursor: disabled ? “not-allowed” : “pointer”, opacity: disabled ? 0.4 : 1 }),
paySuccess: { padding: “40px 20px”, textAlign: “center”, display: “flex”, flexDirection: “column”, alignItems: “center”, gap: 12 },
payCheck: { width: 64, height: 64, borderRadius: “50%”, background: “rgba(0,229,160,.15)”, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 32 },
// Nav
bnav: { position: “fixed”, bottom: 0, left: “50%”, transform: “translateX(-50%)”, width: “100%”, maxWidth: 430, display: “flex”, alignItems: “center”, justifyContent: “space-around”, padding: “10px 0 24px”, background: “linear-gradient(to top, #0A0A0F 60%, transparent)”, zIndex: 60 },
nv: (on) => ({ display: “flex”, flexDirection: “column”, alignItems: “center”, gap: 4, cursor: “pointer”, padding: “6px 16px”, borderRadius: 12, background: “none”, border: “none”, color: on ? “#00E5A0” : “#666677”, fontFamily: “‘Outfit’,sans-serif” }),
nvCr: { width: 48, height: 48, background: “linear-gradient(135deg,#00E5A0,#00B4D8)”, border: “none”, borderRadius: “50%”, fontSize: 26, color: “#0A0A0F”, cursor: “pointer”, display: “flex”, alignItems: “center”, justifyContent: “center”, boxShadow: “0 4px 16px rgba(0,229,160,.3)”, marginTop: -16 },
storyBgGrid: { display: “grid”, gridTemplateColumns: “repeat(3,1fr)”, gap: 10, marginBottom: 16 },
storyBgOpt: (on) => ({ aspectRatio: “3/4”, borderRadius: 10, cursor: “pointer”, border: `2px solid ${on ? "#00E5A0" : "transparent"}`, display: “flex”, alignItems: “center”, justifyContent: “center”, fontSize: 24, color: “white”, transform: on ? “scale(1.04)” : “scale(1)”, transition: “all .2s” }),
content: { flex: 1, overflowY: “auto”, paddingBottom: 80 },
};

// ─── Pulsing dots component ───
function Dots() {
const dot = (delay) => ({ width: 6, height: 6, background: “rgba(255,255,255,.5)”, borderRadius: “50%”, animation: `dotPulse 1.4s ease-in-out ${delay}s infinite` });
return (
<div style={c.horoLoad}>
<style>{`@keyframes dotPulse{0%,80%,100%{transform:scale(.6);opacity:.4}40%{transform:scale(1);opacity:1}}`}</style>
<div style={dot(0)} /><div style={dot(0.2)} /><div style={dot(0.4)} />
<span>reading the stars…</span>
</div>
);
}

// ═══ MAIN COMPONENT ═══
export default function TipTap() {
const [user, setUser] = useState(null);
const [obStep, setObStep] = useState(0);
const [obName, setObName] = useState(””);
const [obUser, setObUser] = useState(””);
const [obAvatar, setObAvatar] = useState(””);
const [obSign, setObSign] = useState(””);
const [tab, setTab] = useState(“feed”);
const [posts, setPosts] = useState(SEED_POSTS);
const [liked, setLiked] = useState({});
const [stories, setStories] = useState(SEED_STORIES);
const [viewStory, setViewStory] = useState(null);
const [sIdx, setSIdx] = useState(0);
const [sProg, setSProg] = useState(0);
const [msgs, setMsgs] = useState(SEED_MSGS);
const [openChat, setOpenChat] = useState(null);
const [chatIn, setChatIn] = useState(””);
const [showCreate, setShowCreate] = useState(false);
const [createTxt, setCreateTxt] = useState(””);
const [showStoryCr, setShowStoryCr] = useState(false);
const [sTxt, setSTxt] = useState(””);
const [sBg, setSBg] = useState(STORY_BGS[0]);
const [horo, setHoro] = useState(null);
const [horoLoad, setHoroLoad] = useState(false);
const [horoErr, setHoroErr] = useState(false);
const [showPay, setShowPay] = useState(false);
const [selPlan, setSelPlan] = useState(“pro”);
const [payStep, setPayStep] = useState(“plans”);
const [cardNum, setCardNum] = useState(””);
const [cardExp, setCardExp] = useState(””);
const [cardCvc, setCardCvc] = useState(””);
const [cardName, setCardName] = useState(””);
const [aiMsgs, setAiMsgs] = useState([{ id: “ai0”, role: “assistant”, text: “hey! i’m tappy, your AI sidekick on tip tap. ask me anything \u2014 caption ideas, post feedback, vibe checks, creative inspo, or just chat. what’s up?” }]);
const [aiIn, setAiIn] = useState(””);
const [aiLoading, setAiLoading] = useState(false);
const timerRef = useRef(null);
const chatEnd = useRef(null);
const aiEnd = useRef(null);

useEffect(() => { chatEnd.current?.scrollIntoView({ behavior: “smooth” }); }, [openChat, msgs]);
useEffect(() => { aiEnd.current?.scrollIntoView({ behavior: “smooth” }); }, [aiMsgs]);

// Story timer
useEffect(() => {
if (!viewStory) return;
if (!viewStory.items[sIdx]) { setViewStory(null); return; }
setSProg(0); let el = 0;
timerRef.current = setInterval(() => { el += 50; setSProg((el / 5000) * 100); if (el >= 5000) { if (sIdx < viewStory.items.length - 1) setSIdx(i => i + 1); else setViewStory(null); } }, 50);
return () => clearInterval(timerRef.current);
}, [viewStory, sIdx]);

// Horoscope fetch
useEffect(() => { if (user?.zodiac && isPro() && !horo && !horoLoad) fetchHoro(user.zodiac, user.displayName); }, [user]);

const getU = (id) => { if (id === “me” && user) return user; return USERS.find(u => u.id === id); };
const ap = () => { if (!user) return “free”; if (user.plan === “pro” || user.plan === “max”) return user.plan; if (user.trialStart && daysLeft(user.trialStart) > 0) return “pro”; return “free”; };
const isPro = () => ap() !== “free”;

async function fetchHoro(sign, name) {
setHoroLoad(true); setHoroErr(false);
const today = new Date().toLocaleDateString(“en-US”, { weekday: “long”, month: “long”, day: “numeric”, year: “numeric” });
const zd = ZODIAC.find(z => z.sign === sign);
try {
const r = await fetch(“https://api.anthropic.com/v1/messages”, { method: “POST”, headers: { “Content-Type”: “application/json” }, body: JSON.stringify({ model: “claude-sonnet-4-20250514”, max_tokens: 1000, messages: [{ role: “user”, content: `You are a mystical astrologer for Tip Tap social app. Generate a personalized daily horoscope for today (${today}) for "${name}", zodiac: ${sign} (${zd?.symbol}, ${zd?.element} element). 2-3 sentences, specific advice about creativity/social/growth. Warm mystical tone. No emojis. ONLY the horoscope text.` }] }) });
const d = await r.json();
const t = d.content?.map(x => x.text || “”).join(””) || “”;
if (t) setHoro({ text: t, date: today, sign }); else setHoroErr(true);
} catch { setHoroErr(true); }
setHoroLoad(false);
}

function finishOb() { setUser({ id: “me”, username: obUser.toLowerCase().replace(/\s/g, “.”), displayName: obName, avatar: obAvatar, bio: “”, color: “#00E5A0”, zodiac: obSign, plan: “free”, trialStart: Date.now() }); }
function toggleLike(pid) { const was = liked[pid]; setLiked(p => ({ …p, [pid]: !was })); setPosts(p => p.map(x => x.id === pid ? { …x, likes: x.likes + (was ? -1 : 1) } : x)); }
function doPost() { if (!createTxt.trim()) return; const em = [”\ud83c\udfb5”, “\ud83c\udfa8”, “\ud83d\udca1”, “\ud83d\udd25”, “\u2728”, “\ud83d\udcf8”][Math.floor(Math.random() * 6)]; setPosts(p => [{ id: “p” + Date.now(), userId: “me”, type: “text”, caption: createTxt, media: Math.random() > 0.5 ? em : null, timestamp: Date.now(), likes: 0, comments: 0 }, …p]); setCreateTxt(””); setShowCreate(false); }
function doStory() { if (!sTxt.trim()) return; const em = [”\u2728”, “\ud83c\udfaf”, “\ud83d\udcab”, “\u26a1”][Math.floor(Math.random() * 4)]; const item = { id: “s” + Date.now(), emoji: em, bg: sBg, text: sTxt, ts: Date.now() }; const mi = stories.findIndex(s => s.userId === “me”); if (mi >= 0) setStories(p => p.map((s, i) => i === mi ? { …s, items: […s.items, item] } : s)); else setStories(p => [{ userId: “me”, items: [item] }, …p]); setSTxt(””); setShowStoryCr(false); }
function sendMsg() { if (!chatIn.trim() || !openChat) return; setMsgs(p => ({ …p, [openChat]: […(p[openChat] || []), { id: “m” + Date.now(), from: “me”, text: chatIn, ts: Date.now(), type: “text” }] })); setChatIn(””); }
function attachFile(e) { const f = e.target.files?.[0]; if (!f || !openChat) return; const ic = { “image/png”: “\ud83d\udcf7”, “image/jpeg”: “\ud83d\udcf7”, “video/mp4”: “\ud83c\udfac”, “application/pdf”: “\ud83d\udcc4” }[f.type] || “\ud83d\udcce”; const sz = f.size > 1048576 ? (f.size / 1048576).toFixed(1) + “ MB” : (f.size / 1024).toFixed(0) + “ KB”; setMsgs(p => ({ …p, [openChat]: […(p[openChat] || []), { id: “m” + Date.now(), from: “me”, text: f.name, ts: Date.now(), type: “file”, fileIcon: ic, fileSize: sz }] })); e.target.value = “”; }
function openPayment() { setShowPay(true); setPayStep(“plans”); setSelPlan(ap() === “free” ? “pro” : “max”); setCardNum(””); setCardExp(””); setCardCvc(””); setCardName(””); }
function processPayment() { setPayStep(“processing”); setTimeout(() => { setUser(u => ({ …u, plan: selPlan, trialStart: null })); setPayStep(“success”); }, 2200); }
function handleApplePay() { setPayStep(“processing”); setTimeout(() => { setUser(u => ({ …u, plan: selPlan, trialStart: null })); setPayStep(“success”); }, 1800); }

async function sendAi() {
if (!aiIn.trim() || aiLoading) return;
const userMsg = { id: “ai” + Date.now(), role: “user”, text: aiIn };
setAiMsgs(p => […p, userMsg]);
setAiIn(””); setAiLoading(true);
const history = […aiMsgs, userMsg].slice(-20).map(m => ({ role: m.role === “assistant” ? “assistant” : “user”, content: m.text }));
const zd = user?.zodiac ? ZODIAC.find(z => z.sign === user.zodiac) : null;
try {
const r = await fetch(“https://api.anthropic.com/v1/messages”, {
method: “POST”, headers: { “Content-Type”: “application/json” },
body: JSON.stringify({
model: “claude-sonnet-4-20250514”, max_tokens: 1000,
system: `You are Tappy, a witty, creative AI assistant built into the social media app Tip Tap. You're talking to ${user?.displayName || "a user"} (@${user?.username || "user"}).${zd ? ` They’re a ${user.zodiac} (${zd.symbol}, ${zd.element} element).` : “”} Their current plan is ${ap()}.

Your personality: you’re chill, creative, supportive, slightly playful. You use lowercase mostly. You help with caption writing, post ideas, creative feedback, social strategy, vibe checks, and general chat. You know about their zodiac and can weave it in naturally when relevant. Keep responses concise — 1-3 sentences usually, unless they ask for something longer. You can use occasional emojis but don’t overdo it. You’re their personal creative sidekick, not a corporate bot.`,
messages: history,
}),
});
const d = await r.json();
const t = d.content?.map(x => x.text || “”).join(””) || “hmm, something went weird. try again?”;
setAiMsgs(p => […p, { id: “ai” + Date.now() + “r”, role: “assistant”, text: t }]);
} catch {
setAiMsgs(p => […p, { id: “ai” + Date.now() + “e”, role: “assistant”, text: “lost connection for a sec. try again?” }]);
}
setAiLoading(false);
}

// ═══ ONBOARDING ═══
if (!user) {
return (
<div style={c.app}>
<div style={c.ob}>
<div style={c.obLogo}>tip tap</div>
<div style={c.obTag}>share everything</div>
<div style={c.obStep}>step {obStep + 1} of 4</div>
{obStep === 0 && <><input style={c.inp} placeholder=“your name” value={obName} onChange={e => setObName(e.target.value)} autoFocus /><button style={{ …c.btn, …(obName.trim() ? {} : c.btnOff) }} disabled={!obName.trim()} onClick={() => setObStep(1)}>next \u2192</button></>}
{obStep === 1 && <><input style={c.inp} placeholder=“pick a username” value={obUser} onChange={e => setObUser(e.target.value)} autoFocus /><button style={{ …c.btn, …(obUser.trim() ? {} : c.btnOff) }} disabled={!obUser.trim()} onClick={() => setObStep(2)}>next \u2192</button></>}
{obStep === 2 && <><div style={{ color: “#9999AA”, fontSize: 14, marginBottom: 16 }}>what’s your sign?</div><div style={c.zodGrid}>{ZODIAC.map(z => <button key={z.sign} style={c.zodBtn(obSign === z.sign)} onClick={() => setObSign(z.sign)}><span style={{ fontSize: 22 }}>{z.symbol}</span><span style={{ fontSize: 10, textTransform: “capitalize”, fontWeight: 600 }}>{z.sign}</span></button>)}</div><button style={{ …c.btn, …(obSign ? {} : c.btnOff) }} disabled={!obSign} onClick={() => setObStep(3)}>next \u2192</button></>}
{obStep === 3 && <><div style={c.emoGrid}>{AVATARS.map(em => <button key={em} style={c.emoBtn(obAvatar === em)} onClick={() => setObAvatar(em)}>{em}</button>)}</div><button style={{ …c.btn, …(obAvatar ? {} : c.btnOff) }} disabled={!obAvatar} onClick={finishOb}>let’s go \ud83d\ude80</button></>}
</div>
</div>
);
}

// ═══ STORY VIEWER ═══
if (viewStory) {
const su = getU(viewStory.userId); const item = viewStory.items[sIdx];
if (!item) { setViewStory(null); return null; }
return (
<div style={c.app}>
<div style={c.sv(item.bg)}>
<div style={c.svProg}>{viewStory.items.map((_, i) => <div key={i} style={c.svBar}><div style={c.svFill(i < sIdx ? “100%” : i === sIdx ? sProg + “%” : “0%”)} /></div>)}</div>
<div style={c.svHd}><span style={{ fontSize: 28 }}>{su?.avatar}</span><span style={{ fontWeight: 600, fontSize: 14 }}>{su?.displayName}</span><span style={{ fontSize: 12, color: “rgba(255,255,255,.5)” }}>{timeAgo(item.ts)}</span><button style={c.svClose} onClick={() => setViewStory(null)}>\u2715</button></div>
<div style={c.svBody}><div style={{ fontSize: 100 }}>{item.emoji}</div><div style={{ fontSize: 28, fontWeight: 700, textAlign: “center”, color: “white” }}>{item.text}</div></div>
<div style={c.svTaps}><div style={c.svTap} onClick={() => { if (sIdx > 0) setSIdx(i => i - 1); }} /><div style={c.svTap} onClick={() => { if (sIdx < viewStory.items.length - 1) setSIdx(i => i + 1); else setViewStory(null); }} /></div>
</div>
</div>
);
}

// ═══ CHAT ═══
if (openChat) {
const cu = getU(openChat); const cm = msgs[openChat] || [];
return (
<div style={c.app}>
<div style={c.chat}>
<div style={c.chatHd}><button style={c.chatBk} onClick={() => setOpenChat(null)}>\u2190</button><span style={{ fontSize: 28 }}>{cu?.avatar}</span><span style={{ fontWeight: 600, fontSize: 16, flex: 1 }}>{cu?.displayName}</span></div>
<div style={c.chatMsgs}>
{cm.map(m => m.type === “file” ?
<div key={m.id} style={c.bubFile(m.from === “me”)}><span style={{ fontSize: 24 }}>{m.fileIcon}</span><div style={{ flex: 1 }}><div style={{ fontSize: 13, fontWeight: 600 }}>{m.text}</div><div style={{ fontSize: 11, color: “#666677” }}>{m.fileSize}</div></div></div>
: <div key={m.id} style={c.bub(m.from === “me”)}>{m.text}</div>
)}
<div ref={chatEnd} />
</div>
<div style={c.chatBar}>
<label style={c.chatAtt}>\ud83d\udcce<input type=“file” accept=“image/*,video/*,audio/*,.pdf,.doc,.zip” onChange={attachFile} style={{ position: “absolute”, inset: 0, opacity: 0, cursor: “pointer” }} /></label>
<input style={c.chatInp} placeholder=“message…” value={chatIn} onChange={e => setChatIn(e.target.value)} onKeyDown={e => e.key === “Enter” && sendMsg()} autoFocus />
<button style={c.chatSnd} onClick={sendMsg}>\u2191</button>
</div>
</div>
</div>
);
}

// ═══ PAYMENT MODAL ═══
const PayModal = () => {
if (!showPay) return null;
const plan = PLANS.find(p => p.id === selPlan);
const cardValid = cardNum.length >= 19 && cardExp.length >= 5 && cardCvc.length >= 3 && cardName.trim();
return (
<div style={c.payOv} onClick={e => e.target === e.currentTarget && payStep !== “processing” && setShowPay(false)}>
<div style={c.payCard}>
{payStep === “plans” && <>
<div style={c.payHdr}><div style={{ fontSize: 20, fontWeight: 800 }}>upgrade tip tap</div><button style={c.shClose} onClick={() => setShowPay(false)}>\u2715</button></div>
<div style={{ padding: “0 16px 16px”, display: “flex”, flexDirection: “column”, gap: 10 }}>
{PLANS.map(p => <div key={p.id} style={{ …c.plan(selPlan === p.id), opacity: p.id === “free” ? 0.5 : 1, cursor: p.id === “free” ? “default” : “pointer” }} onClick={() => p.id !== “free” && setSelPlan(p.id)}>
<div style={{ display: “flex”, alignItems: “center”, justifyContent: “space-between”, marginBottom: 8 }}>
<div style={{ display: “flex”, alignItems: “center”, gap: 8 }}><span style={{ fontSize: 16, fontWeight: 700 }}>{p.name}</span>{p.badge && <span style={{ fontSize: 9, fontWeight: 700, padding: “3px 8px”, borderRadius: 12, textTransform: “uppercase”, letterSpacing: 1, background: p.id === “pro” ? “rgba(0,229,160,.15)” : “rgba(167,139,250,.15)”, color: p.color }}>{p.badge}</span>}</div>
<span style={{ fontSize: 14, fontWeight: 600, color: p.color }}>{p.price === 0 ? “Free” : “$” + p.price + “/mo”}</span>
</div>
<div style={{ display: “flex”, flexDirection: “column”, gap: 4 }}>{p.features.map((f, i) => <div key={i} style={{ fontSize: 12, color: “#9999AA” }}>{”\u2713 “ + f}</div>)}</div>
{ap() === p.id && <div style={{ fontSize: 10, color: “#666677”, textTransform: “uppercase”, letterSpacing: 1, marginTop: 6 }}>current plan</div>}
</div>)}
</div>
<div style={{ padding: “0 16px 20px” }}><button style={c.subBtn(ap() === selPlan)} disabled={ap() === selPlan} onClick={() => setPayStep(“form”)}>continue with {plan?.name} \u2192</button></div>
</>}

```
      {payStep === "form" && <>
        <div style={c.payHdr}><div style={{ fontSize: 20, fontWeight: 800 }}>{plan?.name} \u00b7 ${plan?.price}/mo</div><button style={c.shClose} onClick={() => setPayStep("plans")}>\u2190</button></div>
        <div style={{ padding: "0 16px 16px" }}>
          <button style={c.applePay} onClick={handleApplePay}>
            <svg width="18" height="18" viewBox="0 0 24 24" fill="white"><path d="M17.05 20.28c-.98.95-2.05.88-3.08.4-1.09-.5-2.08-.48-3.24 0-1.44.62-2.2.44-3.06-.4C4.24 16.7 4.89 10.87 8.86 10.6c1.26.07 2.13.72 2.91.77.99-.2 1.94-.78 3-.7 1.27.1 2.23.6 2.84 1.53-2.6 1.56-1.98 4.98.44 5.94-.52 1.38-1.2 2.74-2 4.14zM12.03 10.54C11.88 8.27 13.7 6.42 15.8 6.25c.3 2.58-2.34 4.5-3.77 4.29z" /></svg>
            Subscribe with Apple Pay
          </button>
          <div style={{ textAlign: "center", color: "#666677", fontSize: 12, margin: "6px 0", position: "relative" }}>
            <span style={{ background: "#13131A", padding: "0 12px", position: "relative", zIndex: 1 }}>or pay with card</span>
            <div style={{ position: "absolute", top: "50%", left: 0, right: 0, height: 1, background: "#2A2A3A" }} />
          </div>
        </div>
        <div style={{ padding: "0 16px 16px" }}>
          <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 14, paddingTop: 10, borderTop: "1px solid #2A2A3A" }}>
            <div style={{ fontSize: 13, fontWeight: 700, color: "#9999AA", letterSpacing: 1, display: "flex", alignItems: "center", gap: 6 }}>
              <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2"><rect x="1" y="4" width="22" height="16" rx="2" /><line x1="1" y1="10" x2="23" y2="10" /></svg>
              stripe
            </div>
          </div>
          <div style={{ marginBottom: 10 }}><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, marginBottom: 4 }}>cardholder name</div><input style={c.stripeInp} placeholder="Name on card" value={cardName} onChange={e => setCardName(e.target.value)} /></div>
          <div style={{ marginBottom: 10 }}><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, marginBottom: 4 }}>card number</div><input style={c.stripeInp} placeholder="4242 4242 4242 4242" value={cardNum} onChange={e => setCardNum(fmtCard(e.target.value))} /></div>
          <div style={{ display: "flex", gap: 10, marginBottom: 14 }}>
            <div style={{ flex: 1 }}><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, marginBottom: 4 }}>expiry</div><input style={c.stripeInp} placeholder="MM/YY" value={cardExp} onChange={e => setCardExp(fmtExp(e.target.value))} /></div>
            <div style={{ flex: 1 }}><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, marginBottom: 4 }}>cvc</div><input style={c.stripeInp} placeholder="123" value={cardCvc} onChange={e => setCardCvc(e.target.value.replace(/\D/g, "").slice(0, 3))} /></div>
          </div>
          <button style={c.stripePay(!cardValid)} disabled={!cardValid} onClick={processPayment}>Subscribe \u00b7 ${plan?.price}/mo</button>
          <div style={{ textAlign: "center", marginTop: 10, fontSize: 11, color: "#666677" }}>\ud83d\udd12 Secured by Stripe \u00b7 Cancel anytime</div>
        </div>
      </>}

      {payStep === "processing" && <div style={{ ...c.paySuccess, minHeight: 300 }}>
        <div style={{ fontSize: 40 }}>\u23f3</div>
        <div style={{ fontSize: 20, fontWeight: 700 }}>Processing payment...</div>
        <div style={{ fontSize: 14, color: "#9999AA" }}>connecting to payment provider</div>
        <Dots />
      </div>}

      {payStep === "success" && <div style={c.paySuccess}>
        <div style={c.payCheck}>\u2713</div>
        <div style={{ fontSize: 20, fontWeight: 700 }}>Welcome to {plan?.name}!</div>
        <div style={{ fontSize: 14, color: "#9999AA" }}>your subscription is now active</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 4, marginTop: 8 }}>{plan?.features.map((f, i) => <div key={i} style={{ fontSize: 12, color: "#9999AA" }}>{"\u2713 " + f}</div>)}</div>
        <button style={{ ...c.subBtn(false), marginTop: 16, maxWidth: 280 }} onClick={() => setShowPay(false)}>let's go \u2192</button>
      </div>}
    </div>
  </div>
);
```

};

// ═══ MAIN ═══
const convUsers = USERS.filter(u => msgs[u.id]?.length);
const myPosts = posts.filter(p => p.userId === “me”);
const curPlan = ap();
const trial = user?.trialStart && !(user?.plan === “pro” || user?.plan === “max”);
const dl = daysLeft(user?.trialStart);
const zd = user?.zodiac && ZODIAC.find(z => z.sign === user.zodiac);

return (
<div style={c.app}>
<PayModal />

```
  {/* Header */}
  <div style={c.hdr}>
    <div style={c.hdrLogo}>tip tap</div>
    <div style={{ display: "flex", gap: 8, alignItems: "center" }}>
      {curPlan !== "free" && <span style={c.subBadge(curPlan)}>{curPlan.toUpperCase()}</span>}
      <button style={c.ibtn} onClick={() => setTab("messages")}>\ud83d\udcac{convUsers.length > 0 && <span style={c.bdg}>{convUsers.length}</span>}</button>
    </div>
  </div>

  <div style={c.content}>
    {/* ─── FEED ─── */}
    {tab === "feed" && <>
      {trial && dl > 0 && <div style={c.trial(false)} onClick={openPayment}><span style={{ fontSize: 20 }}>\u26a1</span><div style={{ flex: 1 }}><div style={{ fontSize: 13, fontWeight: 600, color: "#00E5A0" }}>Pro trial \u00b7 {dl} day{dl !== 1 ? "s" : ""} left</div><div style={{ fontSize: 11, color: "#666677" }}>unlimited posts, horoscopes, file sharing</div></div><span style={{ fontSize: 11, fontWeight: 700, color: "#00E5A0", textTransform: "uppercase", letterSpacing: 1 }}>upgrade</span></div>}
      {trial && dl === 0 && <div style={c.trial(true)} onClick={openPayment}><span style={{ fontSize: 20 }}>\ud83d\ude22</span><div style={{ flex: 1 }}><div style={{ fontSize: 13, fontWeight: 600, color: "#FF4757" }}>Trial ended</div><div style={{ fontSize: 11, color: "#666677" }}>upgrade to keep Pro features</div></div><span style={{ fontSize: 11, fontWeight: 700, color: "#FF4757", textTransform: "uppercase", letterSpacing: 1 }}>upgrade</span></div>}

      {/* Stories */}
      <div style={c.storiesBar}>
        <div style={c.stAv} onClick={() => setShowStoryCr(true)}><div style={c.stRing(true)}><div style={c.stInner(true)}>+</div></div><span style={c.stNm}>add story</span></div>
        {stories.map(st => { const su = getU(st.userId); return <div key={st.userId} style={c.stAv} onClick={() => { setViewStory(st); setSIdx(0); }}><div style={c.stRing(false)}><div style={c.stInner(false)}>{su?.avatar}</div></div><span style={c.stNm}>{su?.displayName}</span></div>; })}
      </div>

      {/* Horoscope */}
      {user?.zodiac && isPro() && (() => {
        const bg = ELEM_BG[zd?.element] || ELEM_BG.fire;
        return (<div style={c.horo(bg)}><div style={c.horoIn}>
          <div style={c.horoHd}><div style={c.horoSym}>{zd?.symbol}</div><div style={{ flex: 1 }}><div style={{ fontSize: 16, fontWeight: 700, textTransform: "capitalize", color: "white" }}>{user.zodiac}</div><div style={{ fontSize: 11, color: "rgba(255,255,255,.6)", fontFamily: "'Space Mono',monospace" }}>{zd?.dates}</div></div><div style={{ fontSize: 10, fontWeight: 700, textTransform: "uppercase", letterSpacing: 2, color: "rgba(255,255,255,.5)", padding: "4px 10px", background: "rgba(255,255,255,.1)", borderRadius: 20 }}>today</div></div>
          {horoLoad && <Dots />}
          {horoErr && !horoLoad && <div style={{ color: "rgba(255,255,255,.6)", fontSize: 13, textAlign: "center", padding: "8px 0" }}>the stars are shy today<br /><button style={c.horoRef} onClick={() => fetchHoro(user.zodiac, user.displayName)}>try again</button></div>}
          {horo && !horoLoad && <><div style={c.horoTxt}>{horo.text}</div><button style={c.horoRef} onClick={() => { setHoro(null); fetchHoro(user.zodiac, user.displayName); }}>{"\u2726"} refresh reading</button></>}
          {!horo && !horoLoad && !horoErr && <button style={c.horoRef} onClick={() => fetchHoro(user.zodiac, user.displayName)}>{"\u2726"} get your reading</button>}
        </div></div>);
      })()}
      {user?.zodiac && !isPro() && <div style={{ ...c.horo("linear-gradient(135deg,#1a1a2e,#252532)"), cursor: "pointer" }} onClick={openPayment}><div style={c.horoIn}><div style={c.horoHd}><div style={c.horoSym}>\ud83d\udd12</div><div style={{ flex: 1 }}><div style={{ fontSize: 16, fontWeight: 700, color: "white" }}>daily horoscope</div><div style={{ fontSize: 11, color: "rgba(255,255,255,.6)" }}>upgrade to Pro to unlock</div></div></div></div></div>}

      {/* Posts */}
      <div style={{ padding: "8px 0" }}>{posts.map(post => { const pu = getU(post.userId); const il = liked[post.id]; return (
        <div key={post.id} style={c.post}>
          <div style={c.postHd}><div style={c.postAv(pu?.color)}>{pu?.avatar}</div><div style={{ flex: 1 }}><div style={{ fontWeight: 600, fontSize: 14 }}>{pu?.username}</div><div style={{ fontSize: 12, color: "#666677" }}>{timeAgo(post.timestamp)}</div></div></div>
          {post.media && <div style={c.postMedia(MEDIA_BG[Math.abs(post.id.charCodeAt(1)) % MEDIA_BG.length])}>{post.media}{post.type === "video" && <span style={{ position: "absolute", top: 12, right: 12, background: "rgba(0,0,0,.6)", padding: "4px 10px", borderRadius: 20, fontSize: 11, fontWeight: 600, color: "white" }}>{"\u25b6"} VIDEO</span>}</div>}
          <div style={c.postCap}>{post.caption}</div>
          <div style={c.postActs}><button style={c.postAct(il)} onClick={() => toggleLike(post.id)}>{il ? "\u2665" : "\u2661"} {post.likes + (il ? 1 : 0)}</button><button style={c.postAct(false)}>\ud83d\udcac {post.comments}</button><button style={c.postAct(false)}>{"\u2197"}</button></div>
        </div>
      ); })}</div>
    </>}

    {/* ─── MESSAGES ─── */}
    {tab === "messages" && <div>{USERS.map(u => { const um = msgs[u.id] || []; const lm = um[um.length - 1]; return (
      <div key={u.id} style={c.msgItem} onClick={() => setOpenChat(u.id)}>
        <div style={c.msgAv(u.color)}>{u.avatar}</div>
        <div style={{ flex: 1, minWidth: 0 }}><div style={{ fontWeight: 600, fontSize: 15 }}>{u.displayName}</div><div style={{ fontSize: 13, color: "#9999AA", whiteSpace: "nowrap", overflow: "hidden", textOverflow: "ellipsis" }}>{lm ? (lm.type === "file" ? "\ud83d\udcce " + lm.text : lm.text) : "start a conversation"}</div></div>
        {lm && <span style={{ fontSize: 11, color: "#666677", fontFamily: "'Space Mono',monospace" }}>{timeAgo(lm.ts)}</span>}
        {lm?.from !== "me" && <div style={{ width: 10, height: 10, background: "#00E5A0", borderRadius: "50%", flexShrink: 0 }} />}
      </div>
    ); })}</div>}

    {/* ─── AI CHAT (Tappy) ─── */}
    {tab === "ai" && <div style={{ display: "flex", flexDirection: "column", height: "calc(100vh - 140px)" }}>
      {/* AI Header */}
      <div style={{ padding: "16px 20px", borderBottom: "1px solid #2A2A3A", display: "flex", alignItems: "center", gap: 12 }}>
        <div style={{ width: 44, height: 44, borderRadius: "50%", background: "linear-gradient(135deg,#00E5A0,#00B4D8)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 22, boxShadow: "0 0 20px rgba(0,229,160,.25)" }}>{"\u2728"}</div>
        <div style={{ flex: 1 }}>
          <div style={{ fontWeight: 700, fontSize: 16 }}>tappy</div>
          <div style={{ fontSize: 11, color: "#00E5A0", display: "flex", alignItems: "center", gap: 4 }}><div style={{ width: 6, height: 6, borderRadius: "50%", background: "#00E5A0" }} /> your AI sidekick</div>
        </div>
        <button onClick={() => setAiMsgs([{ id: "ai0", role: "assistant", text: "fresh start! what's on your mind?" }])} style={{ background: "#1C1C26", border: "1px solid #2A2A3A", borderRadius: 8, padding: "6px 12px", color: "#9999AA", fontSize: 11, fontFamily: "'Outfit',sans-serif", cursor: "pointer", fontWeight: 600 }}>clear</button>
      </div>

      {/* AI Suggestions */}
      {aiMsgs.length <= 1 && <div style={{ padding: "12px 16px", display: "flex", gap: 8, overflowX: "auto", flexShrink: 0 }}>
        {["write me a caption", "post ideas for today", "roast my profile", "creative inspo"].map(s => (
          <button key={s} onClick={() => { setAiIn(s); }} style={{ flexShrink: 0, padding: "8px 14px", background: "#1C1C26", border: "1px solid #2A2A3A", borderRadius: 20, color: "#9999AA", fontSize: 12, fontFamily: "'Outfit',sans-serif", cursor: "pointer", whiteSpace: "nowrap" }}>{s}</button>
        ))}
      </div>}

      {/* AI Messages */}
      <div style={{ flex: 1, overflowY: "auto", padding: 16, display: "flex", flexDirection: "column", gap: 10 }}>
        {aiMsgs.map(m => (
          <div key={m.id} style={{ display: "flex", gap: 8, alignSelf: m.role === "user" ? "flex-end" : "flex-start", maxWidth: "85%", flexDirection: m.role === "user" ? "row-reverse" : "row" }}>
            {m.role === "assistant" && <div style={{ width: 28, height: 28, borderRadius: "50%", background: "linear-gradient(135deg,#00E5A0,#00B4D8)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 14, flexShrink: 0, marginTop: 2 }}>{"\u2728"}</div>}
            <div style={{
              padding: "10px 14px",
              borderRadius: m.role === "user" ? "16px 16px 4px 16px" : "16px 16px 16px 4px",
              background: m.role === "user" ? "#00C48C" : "#1C1C26",
              color: m.role === "user" ? "#0A0A0F" : "#EEEEF0",
              fontSize: 14, lineHeight: 1.5,
              whiteSpace: "pre-wrap",
            }}>
              {m.text}
            </div>
          </div>
        ))}
        {aiLoading && <div style={{ display: "flex", gap: 8, alignSelf: "flex-start", maxWidth: "85%" }}>
          <div style={{ width: 28, height: 28, borderRadius: "50%", background: "linear-gradient(135deg,#00E5A0,#00B4D8)", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 14, flexShrink: 0 }}>{"\u2728"}</div>
          <div style={{ padding: "12px 16px", borderRadius: "16px 16px 16px 4px", background: "#1C1C26", display: "flex", gap: 4, alignItems: "center" }}>
            <style>{`@keyframes tappyDot{0%,80%,100%{opacity:.3}40%{opacity:1}}`}</style>
            <div style={{ width: 6, height: 6, borderRadius: "50%", background: "#00E5A0", animation: "tappyDot 1.4s ease-in-out infinite" }} />
            <div style={{ width: 6, height: 6, borderRadius: "50%", background: "#00E5A0", animation: "tappyDot 1.4s ease-in-out .2s infinite" }} />
            <div style={{ width: 6, height: 6, borderRadius: "50%", background: "#00E5A0", animation: "tappyDot 1.4s ease-in-out .4s infinite" }} />
          </div>
        </div>}
        <div ref={aiEnd} />
      </div>

      {/* AI Input */}
      <div style={{ display: "flex", alignItems: "center", gap: 8, padding: "12px 16px", borderTop: "1px solid #2A2A3A", background: "#13131A", flexShrink: 0 }}>
        <input style={{ flex: 1, padding: "10px 16px", background: "#1C1C26", border: "1px solid #2A2A3A", borderRadius: 20, color: "#EEEEF0", fontFamily: "'Outfit',sans-serif", fontSize: 14, outline: "none" }} placeholder="ask tappy anything..." value={aiIn} onChange={e => setAiIn(e.target.value)} onKeyDown={e => e.key === "Enter" && sendAi()} autoFocus />
        <button onClick={sendAi} disabled={!aiIn.trim() || aiLoading} style={{ width: 40, height: 40, display: "flex", alignItems: "center", justifyContent: "center", background: aiIn.trim() && !aiLoading ? "linear-gradient(135deg,#00E5A0,#00B4D8)" : "#1C1C26", border: "none", borderRadius: "50%", color: aiIn.trim() && !aiLoading ? "#0A0A0F" : "#666677", fontSize: 18, cursor: aiIn.trim() && !aiLoading ? "pointer" : "default", flexShrink: 0, transition: "all .2s" }}>{"\u2191"}</button>
      </div>
    </div>}

    {/* ─── PROFILE ─── */}
    {tab === "profile" && <div style={{ padding: "24px 20px" }}>
      <div style={c.profHero}>
        <div style={c.profAv}>{user.avatar}</div>
        <div style={{ fontSize: 22, fontWeight: 700, marginBottom: 2 }}>{user.displayName} {curPlan !== "free" && <span style={c.subBadge(curPlan)}>{curPlan}</span>}</div>
        <div style={{ fontSize: 14, color: "#9999AA", fontFamily: "'Space Mono',monospace", marginBottom: 8 }}>@{user.username}</div>
        {zd && <div style={{ fontSize: 13, color: "#9999AA", marginBottom: 8, display: "flex", alignItems: "center", gap: 6 }}><span style={{ fontSize: 16 }}>{zd.symbol}</span> {user.zodiac} <span style={{ color: "#666677" }}>{"\u00b7"} {zd.element}</span></div>}
        <div style={{ fontSize: 14, color: "#666677", marginBottom: 16 }}>{user.bio || "no bio yet"}</div>
        <div style={c.profStats}><div><div style={{ fontSize: 20, fontWeight: 700, textAlign: "center" }}>{myPosts.length}</div><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, textAlign: "center" }}>posts</div></div><div><div style={{ fontSize: 20, fontWeight: 700, textAlign: "center" }}>0</div><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, textAlign: "center" }}>followers</div></div><div><div style={{ fontSize: 20, fontWeight: 700, textAlign: "center" }}>{USERS.length}</div><div style={{ fontSize: 11, color: "#666677", textTransform: "uppercase", letterSpacing: 1, textAlign: "center" }}>following</div></div></div>
      </div>

      {/* Subscription */}
      <div style={{ fontSize: 13, fontWeight: 600, color: "#666677", textTransform: "uppercase", letterSpacing: 2, marginBottom: 16 }}>subscription</div>
      <div style={{ background: "#13131A", borderRadius: 10, border: "1px solid #2A2A3A", padding: 16, marginBottom: 24 }}>
        <div style={{ display: "flex", alignItems: "center", justifyContent: "space-between", marginBottom: 8 }}>
          <div style={{ display: "flex", alignItems: "center", gap: 8 }}><span style={{ fontSize: 16 }}>{curPlan === "max" ? "\ud83d\udc8e" : curPlan === "pro" ? "\u26a1" : "\ud83c\udd93"}</span><span style={{ fontWeight: 700, fontSize: 15 }}>{PLANS.find(p => p.id === curPlan)?.name}</span></div>
          <span style={{ fontSize: 13, color: curPlan === "free" ? "#666677" : "#00E5A0" }}>{curPlan === "free" ? "Free" : "$" + PLANS.find(p => p.id === curPlan)?.price + "/mo"}</span>
        </div>
        {trial && dl > 0 && <div style={{ fontSize: 12, color: "#00E5A0", marginBottom: 8 }}>{"\u23f0"} Trial: {dl} day{dl !== 1 ? "s" : ""} remaining</div>}
        {trial && dl === 0 && <div style={{ fontSize: 12, color: "#FF4757", marginBottom: 8 }}>Trial expired</div>}
        <button onClick={openPayment} style={{ width: "100%", padding: 12, background: curPlan === "max" ? "#1C1C26" : "linear-gradient(135deg,#00E5A0,#00B4D8)", border: curPlan === "max" ? "1px solid #2A2A3A" : "none", borderRadius: 10, color: curPlan === "max" ? "#EEEEF0" : "#0A0A0F", fontFamily: "'Outfit',sans-serif", fontSize: 14, fontWeight: 600, cursor: "pointer" }}>{curPlan === "max" ? "manage subscription" : curPlan === "pro" ? "upgrade to Max" : "upgrade plan"}</button>
      </div>

      <div style={{ fontSize: 13, fontWeight: 600, color: "#666677", textTransform: "uppercase", letterSpacing: 2, marginBottom: 16 }}>your posts</div>
      {myPosts.length > 0 ? <div style={c.profGrid}>{myPosts.map((p, i) => <div key={p.id} style={c.profTile(MEDIA_BG[i % MEDIA_BG.length])}>{p.media || "\ud83d\udcdd"}</div>)}</div>
        : <div style={{ display: "flex", flexDirection: "column", alignItems: "center", padding: "60px 20px", textAlign: "center" }}><div style={{ fontSize: 48, marginBottom: 16, opacity: 0.5 }}>\ud83d\udcf8</div><div style={{ fontSize: 15, color: "#666677" }}>no posts yet \u2014 tap + to create one</div></div>}
    </div>}
  </div>

  {/* Create Post */}
  {showCreate && <div style={c.overlay} onClick={e => e.target === e.currentTarget && setShowCreate(false)}><div style={c.sheet}>
    <div style={c.shHd}><div style={{ fontSize: 18, fontWeight: 700 }}>new post</div><button style={c.shClose} onClick={() => setShowCreate(false)}>{"\u2715"}</button></div>
    <textarea style={c.txa} placeholder="what's on your mind?" value={createTxt} onChange={e => setCreateTxt(e.target.value)} autoFocus />
    <div style={c.mRow}><button style={c.mBtn}>\ud83d\udcf7 photo</button><button style={c.mBtn}>\ud83c\udfac video</button><button style={c.mBtn}>\ud83d\udcce file</button></div>
    <button style={c.subBtn(!createTxt.trim())} disabled={!createTxt.trim()} onClick={doPost}>post it \u2192</button>
  </div></div>}

  {/* Create Story */}
  {showStoryCr && <div style={c.overlay} onClick={e => e.target === e.currentTarget && setShowStoryCr(false)}><div style={c.sheet}>
    <div style={c.shHd}><div style={{ fontSize: 18, fontWeight: 700 }}>new story</div><button style={c.shClose} onClick={() => setShowStoryCr(false)}>{"\u2715"}</button></div>
    <div style={c.storyBgGrid}>{STORY_BGS.map(bg => <div key={bg} style={{ ...c.storyBgOpt(sBg === bg), background: bg }} onClick={() => setSBg(bg)}>{"\u2726"}</div>)}</div>
    <textarea style={{ ...c.txa, minHeight: 80 }} placeholder="story text..." value={sTxt} onChange={e => setSTxt(e.target.value)} autoFocus />
    <button style={c.subBtn(!sTxt.trim())} disabled={!sTxt.trim()} onClick={doStory}>add story \u2192</button>
  </div></div>}

  {/* Bottom Nav */}
  <div style={c.bnav}>
    <button style={c.nv(tab === "feed")} onClick={() => setTab("feed")}><span style={{ fontSize: 22 }}>\ud83c\udfe0</span><span style={{ fontSize: 10, fontWeight: 600, letterSpacing: 0.5 }}>feed</span></button>
    <button style={c.nv(tab === "messages")} onClick={() => setTab("messages")}><span style={{ fontSize: 22 }}>\ud83d\udcac</span><span style={{ fontSize: 10, fontWeight: 600, letterSpacing: 0.5 }}>messages</span></button>
    <button style={c.nvCr} onClick={() => setShowCreate(true)}>+</button>
    <button style={c.nv(tab === "ai")} onClick={() => setTab("ai")}><span style={{ fontSize: 22 }}>{"\u2728"}</span><span style={{ fontSize: 10, fontWeight: 600, letterSpacing: 0.5 }}>tappy</span></button>
    <button style={c.nv(tab === "profile")} onClick={() => setTab("profile")}><span style={{ fontSize: 22 }}>\ud83d\udc64</span><span style={{ fontSize: 10, fontWeight: 600, letterSpacing: 0.5 }}>profile</span></button>
  </div>
</div>
```

);
}