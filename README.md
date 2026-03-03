import { useState, useEffect } from "react";

const STORAGE_KEY = "ai-interview-qna";
const TAGS = ["ML Fundamentals","Deep Learning","NLP","Computer Vision","Reinforcement Learning","MLOps","Statistics","System Design","Ethics & Safety","General AI"];
const SEED = [{ id:"s1", question:"What is the difference between supervised and unsupervised learning?", answer:"Supervised learning uses labeled data to train a model to map inputs to outputs (e.g. classification, regression). Unsupervised learning finds hidden patterns in unlabeled data (e.g. clustering, dimensionality reduction). The key difference is whether ground-truth labels guide training.", date:"2026-03-03", tag:"ML Fundamentals" }];

export default function App() {
  const [qas, setQas] = useState([]);
  const [loading, setLoading] = useState(true);
  const [expanded, setExpanded] = useState(null);
  const [showForm, setShowForm] = useState(false);
  const [saving, setSaving] = useState(false);
  const [filterTag, setFilterTag] = useState("All");
  const [search, setSearch] = useState("");
  const [form, setForm] = useState({ question:"", answer:"", tag:TAGS[0] });
  const [editId, setEditId] = useState(null);
  const [toast, setToast] = useState(null);
  const [confirmDel, setConfirmDel] = useState(null);

  useEffect(() => { load(); }, []);

  const notify = (msg, type="ok") => { setToast({msg,type}); setTimeout(()=>setToast(null),3000); };

  const load = async () => {
    setLoading(true);
    try {
      const r = await window.storage.get(STORAGE_KEY);
      setQas(r?.value ? JSON.parse(r.value) : SEED);
      if (!r?.value) await window.storage.set(STORAGE_KEY, JSON.stringify(SEED));
    } catch { setQas(SEED); }
    setLoading(false);
  };

  const persist = async (list) => {
    try { await window.storage.set(STORAGE_KEY, JSON.stringify(list)); }
    catch { notify("Save failed","err"); }
  };

  const submit = async () => {
    if (!form.question.trim() || !form.answer.trim()) { notify("Fill in both fields","err"); return; }
    setSaving(true);
    let next;
    if (editId) {
      next = qas.map(q => q.id===editId ? {...q,...form} : q);
      notify("Updated!");
    } else {
      next = [{ id:Date.now()+"", ...form, date:new Date().toISOString().split("T")[0] }, ...qas];
      notify("Q&A saved!");
    }
    setQas(next); await persist(next);
    setForm({question:"",answer:"",tag:TAGS[0]}); setShowForm(false); setEditId(null); setSaving(false);
  };

  const startEdit = (qa) => { setForm({question:qa.question,answer:qa.answer,tag:qa.tag}); setEditId(qa.id); setShowForm(true); setExpanded(null); };

  const del = async (id) => {
    const next = qas.filter(q=>q.id!==id);
    setQas(next); await persist(next); setConfirmDel(null); setExpanded(null); notify("Deleted.");
  };

  const list = qas.filter(q => (filterTag==="All"||q.tag===filterTag) && (!search||q.question.toLowerCase().includes(search.toLowerCase())||q.answer.toLowerCase().includes(search.toLowerCase())));

  return (
    <div style={S.page}>
      <div style={S.grid}/>

      {toast && <div style={{...S.toast, background: toast.type==="err"?"#ff4d4d":"#00e5a0"}}>{toast.type==="err"?"⚠ ":"✓ "}{toast.msg}</div>}

      {confirmDel && (
        <div style={S.overlay}>
          <div style={S.modal}>
            <div style={{fontSize:40,marginBottom:12}}>🗑</div>
            <h3 style={{fontWeight:800,fontSize:20,marginBottom:8}}>Delete this entry?</h3>
            <p style={{color:"#6b7d9a",fontSize:14,marginBottom:24}}>This cannot be undone.</p>
            <div style={{display:"flex",gap:12,justifyContent:"center"}}>
              <button style={S.btnCancel} onClick={()=>setConfirmDel(null)}>Cancel</button>
              <button style={S.btnDel} onClick={()=>del(confirmDel)}>Delete</button>
            </div>
          </div>
        </div>
      )}

      {/* HEADER */}
      <header style={S.header}>
        <div>
          <div style={S.brand}>⬡ <span>AI<span style={{color:"#00e5a0"}}>Prep</span></span></div>
          <div style={S.brandSub}>Daily Interview Question Tracker</div>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:20}}>
          <div style={{textAlign:"center"}}>
            <div style={{fontSize:28,fontWeight:800,color:"#00e5a0",lineHeight:1}}>{qas.length}</div>
            <div style={{fontSize:10,color:"#4a6080",fontFamily:"monospace",letterSpacing:"0.1em"}}>TOTAL Q&As</div>
          </div>
          <button style={S.btnAdd} onClick={()=>{ setShowForm(!showForm); setEditId(null); setForm({question:"",answer:"",tag:TAGS[0]}); }}>
            {showForm ? "✕ Close" : "+ Add Q&A"}
          </button>
        </div>
      </header>

      {/* FORM */}
      {showForm && (
        <div style={S.formBox}>
          <h2 style={{fontSize:17,fontWeight:800,color:"#00e5a0",marginBottom:22}}>{editId?"✏ Edit Entry":"✦ New Question"}</h2>
          <div style={{marginBottom:18}}>
            <div style={S.lbl}>TOPIC</div>
            <div style={{display:"flex",flexWrap:"wrap",gap:7}}>
              {TAGS.map(t=><button key={t} style={{...S.chip,...(form.tag===t?S.chipOn:{})}} onClick={()=>setForm({...form,tag:t})}>{t}</button>)}
            </div>
          </div>
          <div style={{marginBottom:18}}>
            <div style={S.lbl}>QUESTION</div>
            <textarea style={S.ta} rows={3} placeholder="e.g. What is backpropagation?" value={form.question} onChange={e=>setForm({...form,question:e.target.value})}/>
          </div>
          <div style={{marginBottom:20}}>
            <div style={S.lbl}>ANSWER</div>
            <textarea style={{...S.ta,minHeight:110}} rows={5} placeholder="Write a clear, concise answer..." value={form.answer} onChange={e=>setForm({...form,answer:e.target.value})}/>
          </div>
          <div style={{display:"flex",justifyContent:"flex-end"}}>
            <button style={S.btnSave} onClick={submit} disabled={saving}>{saving?"Saving...":editId?"Update →":"Save Q&A →"}</button>
          </div>
        </div>
      )}

      {/* CONTROLS */}
      <div style={S.controls}>
        <div style={{position:"relative",maxWidth:460}}>
          <span style={{position:"absolute",left:13,top:"50%",transform:"translateY(-50%)",color:"#3d5270",fontSize:17,pointerEvents:"none"}}>⌕</span>
          <input style={S.srch} placeholder="Search questions or answers..." value={search} onChange={e=>setSearch(e.target.value)}/>
        </div>
        <div style={{display:"flex",flexWrap:"wrap",gap:7,marginTop:4}}>
          {["All",...TAGS].map(t=><button key={t} style={{...S.ftag,...(filterTag===t?S.ftagOn:{})}} onClick={()=>setFilterTag(t)}>{t}</button>)}
        </div>
        <div style={{fontSize:12,color:"#3d5270",fontFamily:"monospace",marginTop:4}}>
          {!loading && <>{list.length} entr{list.length===1?"y":"ies"}{filterTag!=="All"&&<span style={{color:"#00e5a0"}}> · {filterTag}</span>}{search&&<span style={{color:"#00e5a0"}}> · "{search}"</span>}</>}
        </div>
      </div>

      {/* LIST */}
      <div style={S.list}>
        {loading ? (
          <div style={S.empty}><div style={{fontSize:32,marginBottom:10}}>⟳</div>Loading...</div>
        ) : list.length===0 ? (
          <div style={S.empty}><div style={{fontSize:32,marginBottom:10}}>◇</div>No entries found. Add your first Q&A!</div>
        ) : list.map((qa,i)=>(
          <div key={qa.id} style={{...S.card,animationDelay:`${i*0.04}s`}}>
            <div style={S.cardTop} onClick={()=>setExpanded(expanded===qa.id?null:qa.id)}>
              <div style={{minWidth:108,display:"flex",flexDirection:"column",gap:5}}>
                <span style={S.tagPill}>{qa.tag}</span>
                <span style={{fontSize:10,color:"#3d5270",fontFamily:"monospace"}}>{qa.date}</span>
              </div>
              <div style={{flex:1,display:"flex",alignItems:"flex-start",gap:11}}>
                <span style={S.qBadge}>Q</span>
                <span style={{fontSize:14,fontWeight:600,color:"#c8d8ec",lineHeight:1.55}}>{qa.question}</span>
              </div>
              <span style={{fontSize:10,color:"#3d5270",flexShrink:0,marginLeft:8}}>{expanded===qa.id?"▲":"▼"}</span>
            </div>
            {expanded===qa.id && (
              <div style={S.cardBot}>
                <div style={{display:"flex",alignItems:"flex-start",gap:11,marginBottom:14}}>
                  <span style={S.aBadge}>A</span>
                  <p style={{fontSize:13,color:"#8aa8c8",lineHeight:1.75,fontFamily:"monospace"}}>{qa.answer}</p>
                </div>
                <div style={{display:"flex",gap:9,justifyContent:"flex-end"}}>
                  <button style={S.eBtn} onClick={()=>startEdit(qa)}>✏ Edit</button>
                  <button style={S.dBtn} onClick={()=>setConfirmDel(qa.id)}>🗑 Delete</button>
                </div>
              </div>
            )}
          </div>
        ))}
      </div>

      <footer style={{textAlign:"center",padding:"20px",color:"#2a3c54",fontSize:11,fontFamily:"monospace",borderTop:"1px solid #0a1020",marginTop:16,position:"relative",zIndex:1}}>
        AIPrep Dashboard · {new Date().getFullYear()}
      </footer>

      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Syne:wght@600;700;800&display=swap');
        *{box-sizing:border-box;margin:0;padding:0}
        @keyframes up{from{opacity:0;transform:translateY(14px)}to{opacity:1;transform:translateY(0)}}
        textarea:focus,input:focus{outline:none;border-color:#00e5a0!important}
        ::-webkit-scrollbar{width:5px}::-webkit-scrollbar-track{background:#0d1220}::-webkit-scrollbar-thumb{background:#1e2d4a;border-radius:3px}
      `}</style>
    </div>
  );
}

const S = {
  page:{fontFamily:"'Syne',sans-serif",background:"#070b14",minHeight:"100vh",color:"#e8edf5",paddingBottom:60,position:"relative",overflowX:"hidden"},
  grid:{position:"fixed",inset:0,backgroundImage:"linear-gradient(rgba(0,229,160,0.03) 1px,transparent 1px),linear-gradient(90deg,rgba(0,229,160,0.03) 1px,transparent 1px)",backgroundSize:"40px 40px",pointerEvents:"none",zIndex:0},
  toast:{position:"fixed",top:18,right:18,zIndex:1000,padding:"11px 18px",borderRadius:8,color:"#07111e",fontWeight:700,fontSize:13,fontFamily:"monospace",boxShadow:"0 4px 20px rgba(0,0,0,0.5)"},
  overlay:{position:"fixed",inset:0,background:"rgba(7,11,20,0.88)",zIndex:999,display:"flex",alignItems:"center",justifyContent:"center"},
  modal:{background:"#0d1220",border:"1px solid #1a2840",borderRadius:16,padding:32,textAlign:"center",maxWidth:320,width:"90%"},
  btnCancel:{padding:"9px 22px",borderRadius:8,border:"1px solid #1a2840",background:"transparent",color:"#e8edf5",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:600,fontSize:13},
  btnDel:{padding:"9px 22px",borderRadius:8,border:"none",background:"#ff4d4d",color:"#fff",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:13},
  header:{position:"relative",zIndex:1,display:"flex",alignItems:"center",justifyContent:"space-between",flexWrap:"wrap",gap:14,padding:"24px 28px",borderBottom:"1px solid #0e1828",background:"linear-gradient(180deg,#0a0f1e,transparent)"},
  brand:{display:"flex",alignItems:"center",gap:8,fontSize:24,fontWeight:800,color:"#00e5a0"},
  brandSub:{fontSize:12,color:"#4a6080",fontFamily:"monospace",letterSpacing:"0.05em",marginTop:4},
  btnAdd:{padding:"11px 22px",borderRadius:9,background:"linear-gradient(135deg,#00e5a0,#00b87a)",border:"none",color:"#07111e",fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:13,cursor:"pointer",boxShadow:"0 4px 18px rgba(0,229,160,0.3)"},
  formBox:{position:"relative",zIndex:1,margin:"18px 28px",background:"#0a1020",border:"1px solid #1a2840",borderRadius:14,padding:24,animation:"up 0.3s ease"},
  lbl:{fontSize:10,fontWeight:700,color:"#3d5270",letterSpacing:"0.12em",fontFamily:"monospace",marginBottom:9},
  chip:{padding:"5px 13px",borderRadius:6,border:"1px solid #1a2840",background:"transparent",color:"#4a6080",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:600,fontSize:11},
  chipOn:{border:"1px solid #00e5a0",color:"#00e5a0",background:"rgba(0,229,160,0.08)"},
  ta:{width:"100%",background:"#060c18",border:"1px solid #1a2840",borderRadius:9,padding:"12px 14px",color:"#e8edf5",fontFamily:"monospace",fontSize:13,resize:"vertical",lineHeight:1.6},
  btnSave:{padding:"12px 28px",borderRadius:9,background:"linear-gradient(135deg,#00e5a0,#00b87a)",border:"none",color:"#07111e",fontFamily:"'Syne',sans-serif",fontWeight:700,fontSize:13,cursor:"pointer",boxShadow:"0 4px 18px rgba(0,229,160,0.25)"},
  controls:{position:"relative",zIndex:1,padding:"16px 28px 0",display:"flex",flexDirection:"column",gap:10},
  srch:{width:"100%",background:"#0a1020",border:"1px solid #1a2840",borderRadius:9,padding:"11px 14px 11px 38px",color:"#e8edf5",fontFamily:"monospace",fontSize:12},
  ftag:{padding:"5px 13px",borderRadius:20,border:"1px solid #1a2840",background:"transparent",color:"#4a6080",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:600,fontSize:11},
  ftagOn:{border:"1px solid #00e5a0",color:"#00e5a0",background:"rgba(0,229,160,0.08)"},
  list:{position:"relative",zIndex:1,padding:"12px 28px",display:"flex",flexDirection:"column",gap:9},
  card:{background:"#080e1c",border:"1px solid #111d30",borderRadius:12,overflow:"hidden",animation:"up 0.4s ease both"},
  cardTop:{display:"flex",alignItems:"center",gap:12,padding:"15px 16px",cursor:"pointer"},
  tagPill:{fontSize:9,fontWeight:700,color:"#00e5a0",letterSpacing:"0.1em",fontFamily:"monospace",background:"rgba(0,229,160,0.08)",border:"1px solid rgba(0,229,160,0.2)",borderRadius:4,padding:"3px 7px",display:"inline-block"},
  qBadge:{flexShrink:0,width:24,height:24,background:"linear-gradient(135deg,#1a3a5c,#0e2438)",border:"1px solid #1e3d5e",borderRadius:5,display:"flex",alignItems:"center",justifyContent:"center",fontSize:11,fontWeight:800,color:"#5ba3e0",fontFamily:"monospace"},
  cardBot:{borderTop:"1px solid #111d30",padding:16,background:"#060b18"},
  aBadge:{flexShrink:0,width:24,height:24,background:"linear-gradient(135deg,#0e3326,#092419)",border:"1px solid rgba(0,229,160,0.2)",borderRadius:5,display:"flex",alignItems:"center",justifyContent:"center",fontSize:11,fontWeight:800,color:"#00e5a0",fontFamily:"monospace",marginTop:2},
  eBtn:{padding:"6px 13px",borderRadius:6,border:"1px solid #1a2840",background:"transparent",color:"#5ba3e0",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:600,fontSize:11},
  dBtn:{padding:"6px 13px",borderRadius:6,border:"1px solid rgba(255,77,77,0.2)",background:"transparent",color:"#ff6b6b",cursor:"pointer",fontFamily:"'Syne',sans-serif",fontWeight:600,fontSize:11},
  empty:{textAlign:"center",padding:"50px 20px",color:"#3d5270",fontFamily:"monospace"},
};
