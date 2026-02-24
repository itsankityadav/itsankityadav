import { useState, useEffect } from "react";

const SKILLS = [
  { id: "react", label: "React.js", color: "#61DAFB" },
  { id: "nextjs", label: "Next.js", color: "#ffffff" },
  { id: "php", label: "PHP (Advanced)", color: "#a78bfa" },
  { id: "laravel", label: "Laravel", color: "#f87171" },
  { id: "wpbackend", label: "WordPress Backend", color: "#38bdf8" },
  { id: "nodejs", label: "Node.js", color: "#4ade80" },
  { id: "gsap", label: "GSAP Animations", color: "#88CE02" },
  { id: "dsa", label: "DSA", color: "#fb923c" },
];

const MONTHS = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"];
const DAYS = ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"];

const STATUS_MAP = {
  0:  { label: "Not Started", icon: "○", color: "#475569" },
  25: { label: "Started",     icon: "⚡", color: "#38bdf8" },
  50: { label: "Learning",    icon: "🔥", color: "#fb923c" },
  75: { label: "Practiced",   icon: "💪", color: "#a78bfa" },
  100:{ label: "Mastered",    icon: "🏆", color: "#4ade80" },
};

function getStatus(pct) {
  if (pct >= 100) return STATUS_MAP[100];
  if (pct >= 75)  return STATUS_MAP[75];
  if (pct >= 50)  return STATUS_MAP[50];
  if (pct >= 25)  return STATUS_MAP[25];
  return STATUS_MAP[0];
}

const DEFAULT_STATE = {
  skills: Object.fromEntries(SKILLS.map(s => [s.id, 0])),
  months: Array(12).fill(false),
  days:   Array(7).fill(false),
  weekFocus: "React.js Components & Hooks",
  nextFocus: "PHP OOP & Laravel Routing",
};

export default function GoalTracker() {
  const [data, setData] = useState(DEFAULT_STATE);
  const [loaded, setLoaded] = useState(false);
  const [editFocus, setEditFocus] = useState(null);
  const [focusVal, setFocusVal] = useState("");
  const [toast, setToast] = useState(null);

  useEffect(() => {
    async function load() {
      try {
        const res = await window.storage.get("ankit-goals-v1");
        if (res) setData(JSON.parse(res.value));
      } catch (_) {}
      setLoaded(true);
    }
    load();
  }, []);

  async function save(newData) {
    setData(newData);
    try {
      await window.storage.set("ankit-goals-v1", JSON.stringify(newData));
    } catch (_) {}
  }

  function showToast(msg) {
    setToast(msg);
    setTimeout(() => setToast(null), 2000);
  }

  function setSkill(id, val) {
    const updated = { ...data, skills: { ...data.skills, [id]: val } };
    save(updated);
    showToast(`${SKILLS.find(s=>s.id===id).label} → ${val}%`);
  }

  function toggleMonth(i) {
    const months = [...data.months];
    months[i] = !months[i];
    save({ ...data, months });
  }

  function toggleDay(i) {
    const days = [...data.days];
    days[i] = !days[i];
    save({ ...data, days });
  }

  function saveFocus(field) {
    save({ ...data, [field]: focusVal });
    setEditFocus(null);
    showToast("Focus updated!");
  }

  const streak = data.months.reduce((acc, v, i, arr) => {
    if (!v) return acc;
    let s = 0, j = i;
    while (j >= 0 && arr[j]) { s++; j--; }
    return Math.max(acc, s);
  }, 0);

  const currentStreak = (() => {
    let s = 0;
    for (let i = data.months.length - 1; i >= 0; i--) {
      if (data.months[i]) s++; else break;
    }
    return s;
  })();

  const daysActive = data.days.filter(Boolean).length;
  const overall = Math.round(Object.values(data.skills).reduce((a,b)=>a+b,0) / SKILLS.length);

  if (!loaded) return (
    <div style={{background:"#0d1117",height:"100vh",display:"flex",alignItems:"center",justifyContent:"center",color:"#38bdf8",fontFamily:"'Fira Code',monospace",fontSize:18}}>
      Loading tracker...
    </div>
  );

  return (
    <div style={{
      background: "#0d1117",
      minHeight: "100vh",
      fontFamily: "'Fira Code', 'Courier New', monospace",
      color: "#e2e8f0",
      padding: "24px 16px",
      position: "relative",
    }}>
      {/* Toast */}
      {toast && (
        <div style={{
          position:"fixed", top:20, right:20, zIndex:999,
          background:"#1e293b", border:"1px solid #38bdf8",
          color:"#38bdf8", padding:"8px 18px", borderRadius:6,
          fontSize:13, boxShadow:"0 0 20px #38bdf820",
          animation:"fadeIn .2s ease",
        }}>{toast}</div>
      )}

      <div style={{ maxWidth: 680, margin: "0 auto" }}>

        {/* Header */}
        <div style={{ textAlign:"center", marginBottom:36 }}>
          <div style={{ color:"#38bdf8", fontSize:11, letterSpacing:6, textTransform:"uppercase", marginBottom:8 }}>
            ◈ ANKIT YADAV ◈
          </div>
          <div style={{ fontSize:22, fontWeight:700, color:"#f1f5f9", letterSpacing:1 }}>
            2026 GOAL TRACKER
          </div>
          <div style={{ color:"#475569", fontSize:11, marginTop:6, letterSpacing:3 }}>
            FULL STACK WEB DEVELOPER · WESTONIK
          </div>
        </div>

        {/* ── SKILL PROGRESS ── */}
        <Section title="SKILL PROGRESS BOARD — 2026 ROADMAP">
          {SKILLS.map(skill => {
            const pct = data.skills[skill.id];
            const status = getStatus(pct);
            return (
              <div key={skill.id} style={{ marginBottom: 14 }}>
                <div style={{ display:"flex", justifyContent:"space-between", alignItems:"center", marginBottom:5 }}>
                  <span style={{ fontSize:13, color:"#cbd5e1", minWidth:180 }}>{skill.label}</span>
                  <span style={{ fontSize:12, color: status.color, display:"flex", alignItems:"center", gap:5 }}>
                    <span>{status.icon}</span> {status.label}
                  </span>
                </div>
                <div style={{ display:"flex", alignItems:"center", gap:10 }}>
                  {/* Bar slider */}
                  <div style={{ flex:1, height:8, background:"#1e293b", borderRadius:4, position:"relative", cursor:"pointer" }}
                    onClick={e => {
                      const rect = e.currentTarget.getBoundingClientRect();
                      const raw = Math.round(((e.clientX - rect.left) / rect.width) * 100);
                      setSkill(skill.id, Math.min(100, Math.max(0, raw)));
                    }}
                  >
                    <div style={{
                      height:"100%", borderRadius:4,
                      width: `${pct}%`,
                      background: `linear-gradient(90deg, ${skill.color}88, ${skill.color})`,
                      transition:"width .3s ease",
                      boxShadow: pct > 0 ? `0 0 8px ${skill.color}55` : "none",
                    }}/>
                  </div>
                  {/* Stepper */}
                  <div style={{ display:"flex", gap:4, alignItems:"center" }}>
                    <button onClick={() => setSkill(skill.id, Math.max(0, pct-5))}
                      style={btnStyle}>−</button>
                    <span style={{ fontSize:12, color:"#94a3b8", width:34, textAlign:"center" }}>{pct}%</span>
                    <button onClick={() => setSkill(skill.id, Math.min(100, pct+5))}
                      style={btnStyle}>+</button>
                  </div>
                </div>
              </div>
            );
          })}
          {/* Overall */}
          <div style={{ marginTop:20, paddingTop:16, borderTop:"1px solid #1e293b" }}>
            <div style={{ display:"flex", justifyContent:"space-between", fontSize:12, color:"#64748b", marginBottom:6 }}>
              <span style={{ color:"#94a3b8", fontWeight:700 }}>OVERALL 2026 PROGRESS</span>
              <span style={{ color:"#38bdf8", fontWeight:700 }}>{overall}% — {overall < 30 ? "🚀 LET'S GO!" : overall < 60 ? "💪 KEEP GOING!" : "🔥 CRUSHING IT!"}</span>
            </div>
            <div style={{ height:10, background:"#1e293b", borderRadius:5 }}>
              <div style={{
                height:"100%", borderRadius:5,
                width:`${overall}%`,
                background:"linear-gradient(90deg,#1d4ed8,#38bdf8,#4ade80)",
                transition:"width .4s ease",
                boxShadow:"0 0 12px #38bdf855",
              }}/>
            </div>
          </div>
        </Section>

        {/* ── MONTHLY STREAK ── */}
        <Section title="MONTHLY CONSISTENCY STREAK — 2026">
          <div style={{ display:"grid", gridTemplateColumns:"repeat(6,1fr)", gap:10, marginBottom:20 }}>
            {MONTHS.map((m, i) => (
              <button key={m} onClick={() => toggleMonth(i)}
                style={{
                  background: data.months[i] ? "#0f3460" : "#1e293b",
                  border: `1px solid ${data.months[i] ? "#38bdf8" : "#334155"}`,
                  borderRadius:6, padding:"8px 4px", cursor:"pointer",
                  display:"flex", flexDirection:"column", alignItems:"center", gap:4,
                  transition:"all .2s", boxShadow: data.months[i] ? "0 0 12px #38bdf840" : "none",
                }}
              >
                <span style={{ fontSize:11, color: data.months[i] ? "#38bdf8" : "#475569" }}>{m}</span>
                <span style={{ fontSize:16 }}>{data.months[i] ? "✅" : "🔲"}</span>
              </button>
            ))}
          </div>
          <div style={{ display:"grid", gridTemplateColumns:"repeat(3,1fr)", gap:10, fontSize:12 }}>
            <StatBox icon="🔥" label="Current Streak" value={`${currentStreak} months`} />
            <StatBox icon="🏆" label="Best Streak" value={`${streak} months`} />
            <StatBox icon="📅" label="Goal" value={`${data.months.filter(Boolean).length} / 12 months`} />
          </div>
        </Section>

        {/* ── WEEKLY LOG ── */}
        <Section title="WEEKLY LEARNING LOG">
          <div style={{ display:"grid", gridTemplateColumns:"repeat(7,1fr)", gap:8, marginBottom:20 }}>
            {DAYS.map((d, i) => (
              <button key={d} onClick={() => toggleDay(i)}
                style={{
                  background: data.days[i] ? "#14532d" : "#1e293b",
                  border: `1px solid ${data.days[i] ? "#4ade80" : "#334155"}`,
                  borderRadius:6, padding:"8px 4px", cursor:"pointer",
                  display:"flex", flexDirection:"column", alignItems:"center", gap:4,
                  transition:"all .2s", boxShadow: data.days[i] ? "0 0 10px #4ade8040" : "none",
                }}
              >
                <span style={{ fontSize:10, color: data.days[i] ? "#4ade80" : "#475569" }}>{d}</span>
                <span style={{ fontSize:15 }}>{data.days[i] ? "✅" : "🔲"}</span>
              </button>
            ))}
          </div>

          <div style={{ fontSize:12, color:"#64748b", marginBottom:14, textAlign:"center" }}>
            {daysActive} / 7 days active this week
          </div>

          {/* Focus fields */}
          {[
            { key:"weekFocus", icon:"📌", label:"This week's focus" },
            { key:"nextFocus", icon:"📌", label:"Next week's focus" },
          ].map(({ key, icon, label }) => (
            <div key={key} style={{ display:"flex", alignItems:"center", gap:10, marginBottom:10, background:"#0f172a", borderRadius:8, padding:"10px 14px" }}>
              <span style={{ fontSize:14 }}>{icon}</span>
              <span style={{ fontSize:12, color:"#64748b", minWidth:130 }}>{label}</span>
              <span style={{ fontSize:12, color:"#38bdf8" }}>→</span>
              {editFocus === key ? (
                <input
                  autoFocus
                  value={focusVal}
                  onChange={e => setFocusVal(e.target.value)}
                  onKeyDown={e => e.key === "Enter" && saveFocus(key)}
                  onBlur={() => saveFocus(key)}
                  style={{
                    flex:1, background:"#1e293b", border:"1px solid #38bdf8",
                    color:"#e2e8f0", padding:"4px 10px", borderRadius:4, fontSize:12,
                    outline:"none",
                  }}
                />
              ) : (
                <span
                  onClick={() => { setEditFocus(key); setFocusVal(data[key]); }}
                  style={{ flex:1, fontSize:12, color:"#e2e8f0", cursor:"pointer", borderBottom:"1px dashed #334155" }}
                >
                  {data[key]}
                </span>
              )}
            </div>
          ))}
        </Section>

        {/* Footer */}
        <div style={{ textAlign:"center", marginTop:32, color:"#334155", fontSize:11, letterSpacing:2 }}>
          ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
          <div style={{ marginTop:10, color:"#475569" }}>
            ANKIT YADAV · WESTONIK · INDIA 🇮🇳
          </div>
        </div>

      </div>

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;600;700&display=swap');
        * { box-sizing: border-box; margin:0; padding:0; }
        body { background: #0d1117; }
        @keyframes fadeIn { from { opacity:0; transform:translateY(-6px); } to { opacity:1; transform:none; } }
      `}</style>
    </div>
  );
}

function Section({ title, children }) {
  return (
    <div style={{ marginBottom:28 }}>
      <div style={{ fontSize:11, color:"#64748b", letterSpacing:3, marginBottom:12, textTransform:"uppercase" }}>
        {title}
      </div>
      <div style={{ height:1, background:"linear-gradient(90deg,#38bdf8,transparent)", marginBottom:18, opacity:.4 }}/>
      <div style={{ background:"#0f172a", borderRadius:10, padding:"18px 20px", border:"1px solid #1e293b" }}>
        {children}
      </div>
    </div>
  );
}

function StatBox({ icon, label, value }) {
  return (
    <div style={{ background:"#0d1117", borderRadius:8, padding:"10px 12px", border:"1px solid #1e293b", textAlign:"center" }}>
      <div style={{ fontSize:18, marginBottom:4 }}>{icon}</div>
      <div style={{ fontSize:10, color:"#475569", marginBottom:4 }}>{label}</div>
      <div style={{ fontSize:12, color:"#94a3b8", fontWeight:600 }}>{value}</div>
    </div>
  );
}

const btnStyle = {
  background:"#1e293b", border:"1px solid #334155", color:"#94a3b8",
  width:22, height:22, borderRadius:4, cursor:"pointer", fontSize:14,
  display:"flex", alignItems:"center", justifyContent:"center", lineHeight:1,
};
