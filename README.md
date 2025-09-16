# Vid.ai.gen
This app is a viral video generator that uses ai
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>AI Viral Content Creator</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body { font-family: Arial, sans-serif; margin: 0; background: #f5f5f5; }
.tabs { display: flex; background: #333; flex-wrap: wrap; }
.tab { flex: 1; min-width: 110px; padding: 15px; text-align: center; cursor: pointer; color: #fff; border: none; background: #333; transition: background 0.2s;}
.tab.active { background: #444; font-weight: bold; }
.tab-content { display: none; padding: 20px; }
.tab-content.active { display: block; }
.btn { padding: 10px 20px; margin-top: 10px; background: #007bff; color: white; border: none; cursor: pointer; border-radius:5px; font-size: 1em;}
.btn[disabled], .btn:disabled { opacity: 0.5; cursor: not-allowed; }
.btn:hover:enabled { background: #0056b3; }
.trend-item { background: #fff; margin: 10px 0; padding: 15px; border-radius: 5px; box-shadow:0 2px 6px #0001;}
.metric { margin-right: 15px; font-size: 0.92em; padding:2px 7px; border-radius:5px;}
.metric.virality { background:#ffe4e1; color:#c41d00;}
.metric.growth { background:#e6ffe6; color:#006b1b;}
.metric.engagement { background:#e6f3ff; color:#008;}
.loading { text-align: center; padding: 20px; font-size:1.1em}
.toast {position:fixed;left:50%;transform:translateX(-50%);top:30px;min-width:180px;background:#007bff;color:white;padding:10px 22px;border-radius:8px;z-index:1300;font-size:1.06em;box-shadow:0 2px 8px #0003; opacity:0; transition:opacity 0.2s;}
.toast.show{opacity:1;}
label { display:block; margin:8px 0 3px 0;font-weight:600;}
input, select {margin:0 0 10px 0; padding:6px 8px; border-radius:5px; border:1px solid #ccc; width:250px; max-width:95%;}
@media (max-width:700px) {
.tabs { flex-direction: column;}
.tab { border-bottom: 1px solid #222; text-align: left;}
.tab-content { padding: 12px;}
input, select { width:99%;}
#engagementChart { width:100%!important; max-width:99vw;}
}
</style>
</head>
<body>

<!-- Toast Notification -->
<div id="toast" class="toast" role="alert" aria-live="assertive"></div>

<!-- Navigation Tabs -->
<div class="tabs" role="tablist">
<button class="tab" role="tab" aria-controls="trends" tabindex="0" onclick="showTab('trends', event)">📈 Trend Analysis</button>
<button class="tab" role="tab" aria-controls="generator" tabindex="0" onclick="showTab('generator', event)">📝 Content Generator</button>
<button class="tab" role="tab" aria-controls="scheduler" tabindex="0" onclick="showTab('scheduler', event)">⏰ Smart Scheduler</button>
<button class="tab" role="tab" aria-controls="analytics" tabindex="0" onclick="showTab('analytics', event)">📊 Analytics</button>
</div>

<!-- Trend Analysis -->
<div id="trends" class="tab-content" role="tabpanel">
<h2>📈 Trend Analysis</h2>
<label for="trendPlatform">Platform:</label>
<select id="trendPlatform">
<option value="tiktok">TikTok</option>
<option value="instagram">Instagram</option>
</select>
<label for="trendCategory">Category:</label>
<select id="trendCategory">
<option value="all">All</option>
<option value="dance">Dance</option>
<option value="lifestyle">Lifestyle</option>
<option value="tech">Tech</option>
</select>
<button class="btn" id="trendsBtn" onclick="analyzeTrends()">Analyze Trends</button>
<div id="trendsResult" aria-live="polite"></div>
<button class="btn" onclick="copyToClipboard('trendsResult')">📋 Copy</button>
</div>

<!-- Content Generator -->
<div id="generator" class="tab-content" role="tabpanel">
<h2>📝 Content Generator</h2>
<form id="contentForm" onsubmit="event.preventDefault(); generateContent();">
<label for="userNiche">Your Niche</label>
<input id="userNiche" autocomplete="off" required placeholder="e.g. fitness, cooking">
<label for="contentType">Type</label>
<select id="contentType">
<option value="shorts">Short Form</option>
<option value="carousel">Carousel</option>
<option value="tutorial">Tutorial</option>
</select>
<label for="targetAudience">Target Audience <span style="font-weight:400">(optional)</span></label>
<input id="targetAudience" autocomplete="off" placeholder="Teens, new moms, etc.">
<button class="btn" id="contentBtn" type="submit">Generate Content Ideas</button>
</form>
<div id="contentResult" aria-live="polite"></div>
<button class="btn" onclick="copyToClipboard('contentResult')">📋 Copy</button>
</div>

<!-- Smart Scheduler -->
<div id="scheduler" class="tab-content" role="tabpanel">
<h2>⏰ Smart Scheduler</h2>
<label for="timezone">Timezone:</label>
<select id="timezone">
<option value="EST">EST</option>
<option value="CST">CST</option>
<option value="MST">MST</option>
<option value="PST">PST</option>
</select>
<label for="scheduleContentType">Content Type:</label>
<select id="scheduleContentType">
<option value="shorts">Short Form</option>
<option value="carousel">Carousel</option>
<option value="tutorial">Tutorial</option>
</select>
<button class="btn" id="scheduleBtn" onclick="getOptimalTimes()">Find Best Times</button>
<div id="schedulerResult" aria-live="polite"></div>
<button class="btn" onclick="copyToClipboard('schedulerResult')">📋 Copy</button>
</div>

<!-- Analytics -->
<div id="analytics" class="tab-content" role="tabpanel">
<h2>📊 Engagement Analytics</h2>
<canvas id="engagementChart" width="600" height="300" aria-label="Bar chart of Instagram and TikTok engagement"></canvas>
<div id="chartPlaceholder" class="loading" style="display:none;">No data yet! Run "Smart Scheduler" to view engagement analytics.</div>
</div>

<!-- Chart.js (with fallback detection) -->
<script src="https://cdn.jsdelivr.net/npm/chart.js" onerror="chartJsFailed = true;"></script>
<script>
/* ===== Utils ===== */
function setLoading(el, show = true, msg = "Loading...") {
if (!el) return;
el.innerHTML = show ? `<div class="loading">${msg}</div>` : "";
}

function showToast(msg, ms = 1500) {
const toast = document.getElementById('toast');
if (!toast) return;
toast.innerText = msg;
toast.classList.add('show');
setTimeout(() => { toast.classList.remove('show'); }, ms);
}

// Safe localStorage wrapper
function safeGet(key) { try { return localStorage.getItem(key); } catch { return null; } }
function safeSet(key, val) { try { localStorage.setItem(key, val); } catch {} }

// Accessibility for tabs
document.querySelectorAll('.tab').forEach(el => {
el.addEventListener('keydown', function(e){
if(e.key === "Enter" || e.key === " "){
e.preventDefault();
el.click();
}
});
});

/* ===== Tab Handling ===== */
function showTab(tabName, event) {
document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
const content = document.getElementById(tabName);
if (content) content.classList.add('active');
if (event && event.target) event.target.classList.add('active');
else {
document.querySelectorAll('.tab').forEach(t => {
if (t.getAttribute('aria-controls') === tabName) t.classList.add('active');
});
}
if(tabName==="analytics"){ chartPlaceholderDisplay(); }
safeSet('activeTab', tabName);
}

function chartPlaceholderDisplay() {
document.getElementById('chartPlaceholder').style.display =
engagementChart && engagementChart.data ? 'none' : '';
}

// Restore tab after load
document.addEventListener("DOMContentLoaded", () => {
const saved = safeGet('activeTab') || 'trends';
showTab(saved);
[...document.querySelectorAll('.tab')].forEach(t =>
t.getAttribute("aria-controls") === saved && t.classList.add("active")
);
chartPlaceholderDisplay();
if (typeof Chart === "undefined") {
chartJsFailed = true;
chartPlaceholderDisplay();
}
});

/* ===== Trend Analysis ===== */
function analyzeTrends() {
const btn = document.getElementById('trendsBtn');
if(btn) btn.disabled = true;
const platform = document.getElementById('trendPlatform')?.value;
const category = document.getElementById('trendCategory')?.value;
const resultBox = document.getElementById('trendsResult');
setLoading(resultBox, true, "Analyzing current trends...");
setTimeout(() => {
const trends = generateTrendData(platform, category);
let html;
if(!trends.length){
html = `<div class="loading">No trends found for this category.</div>`;
} else {
html = '<h3>🔥 Current Viral Trends</h3>';
trends.forEach(t => {
html += `<div class="trend-item">
<h4>${t.title}</h4>
<p>${t.description}</p>
<div>
<span class="metric virality">🔥 ${t.virality}</span>
<span class="metric engagement">👀 ${t.engagement}</span>
<span class="metric growth">📈 ${t.growth}</span>
</div>
</div>`;
});
}
resultBox.innerHTML = html;
if(btn) btn.disabled = false;
}, 1200);
}

function generateTrendData(platform, category) {
const trends = [
{ title: "Micro-Dancing Challenges", baseVirality: 75, baseGrowth: 300, niche: "dance", description: "5-second dances with trending audio." },
{ title: "Before & After Reveals", baseVirality: 85, baseGrowth: 420, niche: "lifestyle", description: "Transformation content: rooms, outfits, recipes." },
{ title: "Quick Hacks & Tips", baseVirality: 70, baseGrowth: 250, niche: "tech", description: "15–30s clips with a single hack." }
];
return trends
.filter(t => category === "all" || t.niche === category)
.map(t => {
const noise = Math.floor(Math.random() * 10) - 5;
const virality = Math.min(100, t.baseVirality + noise);
const growth = t.baseGrowth + Math.floor(Math.random() * 100);
return {
title: t.title,
description: t.description,
virality: virality + "%",
engagement: (60 + Math.random() * 35).toFixed(1) + "%",
growth: `+${growth}%`
};
});
}

/* ===== Content Generator ===== */
function generateContent() {
const btn = document.getElementById('contentBtn');
if(btn) btn.disabled = true;
const niche = document.getElementById('userNiche')?.value.trim();
const type = document.getElementById('contentType')?.value;
const audience = document.getElementById('targetAudience')?.value.trim();
const resultBox = document.getElementById('contentResult');
if (!niche) {
showToast('⚠️ Please enter your niche!');
if(btn) btn.disabled = false;
return;
}
setLoading(resultBox, true, "Generating viral content ideas...");
setTimeout(() => {
const ideas = generateContentIdeas(niche, type, audience);
let html = ideas.length ? '<h3>💡 AI-Generated Content Ideas</h3>' : "";
ideas.forEach(i => {
html += `<div class="trend-item">
<h4>${i.title}</h4>
<p><strong>Hook:</strong> ${i.hook}</p>
<p><strong>Content:</strong> ${i.content}</p>
<p><strong>CTA:</strong> ${i.cta}</p>
<div>
<span class="metric virality">🎯 ${i.viralScore}/10</span>
<span class="metric engagement">⏱️ ${i.duration}</span>
<span class="metric growth">🎬 ${i.difficulty}</span>
</div>
</div>`;
});
resultBox.innerHTML = html || `<div class="loading">No ideas produced. Try adjusting your search.</div>`;
if(btn) btn.disabled = false;
}, 1500);
}

function generateContentIdeas(niche, type, audience) {
return [
{ title: `${niche} Mistakes Everyone Makes`, hook: `Stop doing this ${niche} mistake...`, content: `Reveal a common misconception with tips`, cta: `Follow for more ${niche} secrets!`, viralScore: 7 + Math.floor(Math.random()*3), duration: "15-30s", difficulty: "Easy" },
{ title: `${niche} Before & After`, hook: `This ${niche} transformation will blow your mind...`, content: `Show dramatic before/after`, cta: `Double tap if this motivated you!`, viralScore: 8 + Math.floor(Math.random()*2), duration: "10-20s", difficulty: "Medium" },
{ title: `POV: You're the ${niche} Expert`, hook: `POV: You finally mastered ${niche}...`, content: `Show expertise in first-person style`, cta: `Save this if you want this too!`, viralScore: 6 + Math.floor(Math.random()*4), duration: "20-45s", difficulty: "Easy" }
];
}

/* ===== Smart Scheduler and Analytics ===== */
let engagementChart;
let chartJsFailed = false;

function getOptimalTimes() {
const btn = document.getElementById('scheduleBtn');
if(btn) btn.disabled = true;
const tz = document.getElementById('timezone')?.value;
const type = document.getElementById('scheduleContentType')?.value;
const resultBox = document.getElementById('schedulerResult');
setLoading(resultBox, true, "Analyzing best times...");
setTimeout(() => {
const times = generateOptimalTimes(tz, type);
let html = '<h3>⏰ Best Posting Times</h3>';
times.forEach(t => {
html += `<div class="trend-item">
<h4>${t.day}</h4>
<p><strong>Instagram:</strong> ${t.instagram}</p>
<p><strong>TikTok:</strong> ${t.tiktok}</p>
<span class="metric engagement">📈 ${t.engagement}% Engagement</span>
<span class="metric growth">👀 ${t.reach} Reach</span>
</div>`;
});
resultBox.innerHTML = html;
if(btn) btn.disabled = false;
if (!chartJsFailed && typeof Chart !== "undefined") {
updateEngagementChart(times);
} else {
chartPlaceholderDisplay();
}
}, 1300);
}

function generateOptimalTimes(timezone, contentType) {
const days = ["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday","Sunday"];
return days.map(day => {
let igMorning = `${7 + Math.floor(Math.random()*2)}:00 AM`;
let igEvening = `${6 + Math.floor(Math.random()*2)}:00 PM`;
let tkMorning = `${8 + Math.floor(Math.random()*3)}:00 AM`;
let tkEvening = `${7 + Math.floor(Math.random()*2)}:00 PM`;
return {
day,
instagram: `${igMorning}, ${igEvening}`,
tiktok: `${tkMorning}, ${tkEvening}`,
engagement: Math.floor(70 + Math.random()*20),
reach: ["Medium","High","Very High","Peak"][Math.floor(Math.random()*4)]
};
});
}

function updateEngagementChart(times) {
document.getElementById('chartPlaceholder').style.display = 'none';
const canvas = document.getElementById('engagementChart');
if (!canvas) return;
