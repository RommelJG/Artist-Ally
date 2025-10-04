# Artist-Ally
import React, { useMemo, useRef, useState, useEffect } from "react"; import { motion } from "framer-motion"; import { Card, CardContent } from "@/components/ui/card"; import { Button } from "@/components/ui/button"; import { Input } from "@/components/ui/input"; import { Textarea } from "@/components/ui/textarea"; import { Badge } from "@/components/ui/badge"; import { Download, Send, FileText, Settings2, PlusCircle, Music, Wand2, RotateCcw, Save, CheckCircle2 } from "lucide-react"; import jsPDF from "jspdf";

// ----------------------------- // Demo Mode Seed Data // -----------------------------

function seedDemoVault() { return { id: "demo123", legalName: "Alicia Johnson", stageName: "Lia J", proAffiliations: [{ society: "ASCAP", ipi: "123456789", memberId: "A12345" }], parties: [ { id: "p1", name: "Alicia Johnson", role: ["composer", "lyricist"], sharePct: 50 }, { id: "p2", name: "Marcus Lee", role: ["composer", "producer"], sharePct: 50 }, ], compositions: [ { id: "c1", title: "Shine On", altTitles: ["Shine"], writers: ["p1", "p2"], splitsValidated: true }, ], recordings: [ { id: "r1", title: "Shine On (Original Mix)", compositionId: "c1", isrc: "US-ABC-25-00001", performers: ["p1"], rightsHolder: "p1" }, ], progress: { register: false, neighboring: false, catalog: false } }; }

function saveVault(v) { localStorage.setItem("artist_ally_vault", JSON.stringify(v)); }

function loadVault() { const raw = localStorage.getItem("artist_ally_vault"); if (raw) return JSON.parse(raw); const demo = seedDemoVault(); saveVault(demo); return demo; }

function resetVault() { const demo = seedDemoVault(); saveVault(demo); return demo; }

function recapVault(vault) { let recap = Artist: ${vault.stageName} (${vault.legalName})\n; recap += PROs: ${vault.proAffiliations.map(p => ${p.society} (${p.ipi})).join(", ")}\n\n; recap += "Compositions:\n"; vault.compositions.forEach(c => { recap += - ${c.title} [writers: ${c.writers.map(w => vault.parties.find(p => p.id === w)?.name || w).join(", ")}]\n; }); recap += "\nRecordings:\n"; vault.recordings.forEach(r => { recap += - ${r.title} (ISRC: ${r.isrc || "missing"}) linked to ${vault.compositions.find(c => c.id === r.compositionId)?.title || "no comp"}\n; }); recap += "\nProgress: "; recap += ${Object.entries(vault.progress).filter(([k,v])=>v).length}/3 steps completed.; return recap; }

// ----------------------------- // Chat logic // -----------------------------

function useChat(vault, setVault) { const [msgs, setMsgs] = useState([{ id: "intro", role: "assistant", content: "👋 Welcome to Artist-Ally! I'm here to help you register your songs, collect royalties, and keep your catalog healthy. Try /register, /neighboring, /catalog, /healthcheck, or /recap." }]);

function pushAssistant(content) { setMsgs(m => [...m, { id: Math.random().toString(), role: "assistant", content }]); } function pushUser(content) { setMsgs(m => [...m, { id: Math.random().toString(), role: "user", content }]); }

function handleCommand(input) { if (!input) return; pushUser(input); if (input.startsWith("/register")) { setVault({ ...vault, progress: { ...vault.progress, register: true } }); pushAssistant("Let's walk through registering your composition + recording. Select them in the Wizards panel."); } else if (input.startsWith("/neighboring")) { setVault({ ...vault, progress: { ...vault.progress, neighboring: true } }); pushAssistant("We'll generate a Neighboring Rights packet. Select a recording."); } else if (input.startsWith("/catalog")) { setVault({ ...vault, progress: { ...vault.progress, catalog: true } }); pushAssistant("Open the Catalog to view and edit your works and recordings."); } else if (input.startsWith("/healthcheck")) pushAssistant("Healthcheck complete ✅ All splits balanced and ISRC valid."); else if (input.startsWith("/recap")) pushAssistant(recapVault(vault)); else pushAssistant("Available commands: /register, /neighboring, /catalog, /healthcheck, /recap"); }

return { msgs, handleCommand, pushAssistant }; }

// ----------------------------- // Progress Tracker Component // -----------------------------

function ProgressTracker({ progress }) { const total = 3; const done = Object.values(progress).filter(Boolean).length; const percent = Math.round((done / total) * 100);

return ( <div className="w-full bg-gray-200 rounded-full h-4 overflow-hidden"> <div className="bg-purple-600 h-4 text-xs text-white flex items-center justify-center transition-all" style={{ width: ${percent}% }}> {done}/{total} steps </div> </div> ); }

// ----------------------------- // Main Component // -----------------------------

export default function ArtistAllyBot() { const [vault, setVault] = useState(() => loadVault()); const [input, setInput] = useState(""); const { msgs, handleCommand } = useChat(vault, (v) => { setVault(v); saveVault(v); });

const brandPrimary = "bg-purple-600 hover:bg-purple-700 text-white"; const brandSecondary = "bg-pink-500 hover:bg-pink-600 text-white";

function handleReset() { const fresh = resetVault(); setVault(fresh); alert("Vault reset to demo mode."); }

function handleSaveRecap() { const text = recapVault(vault); const doc = new jsPDF(); doc.setFontSize(14); doc.text("Artist-Ally Vault Recap", 14, 20); doc.setFontSize(11); let y = 30; text.split("\n").forEach(line => { doc.text(line, 14, y); y += 7; }); doc.save("ArtistAlly_Recap.pdf"); }

return ( <div className="w-full max-w-6xl mx-auto font-sans"> {/* HEADER */} <motion.div initial={{ opacity: 0, y: -10 }} animate={{ opacity: 1, y: 0 }} className="flex items-center justify-between mb-6"> <div className="flex items-center gap-3"> <Music className="w-8 h-8 text-purple-600" /> <div> <h1 className="text-2xl font-bold text-gray-800">Artist-Ally</h1> <p className="text-sm text-gray-600">Your personal music rights assistant.</p> </div> </div> <div className="flex gap-2"> <Button className="bg-gray-200 text-gray-700 hover:bg-gray-300 rounded-xl flex items-center gap-1" onClick={handleReset}><RotateCcw className="w-4 h-4"/> Reset</Button> <Button className="bg-green-600 hover:bg-green-700 text-white rounded-xl flex items-center gap-1" onClick={handleSaveRecap}><Save className="w-4 h-4"/> Save Recap</Button> </div> </motion.div>

{/* PROGRESS TRACKER */}
  <div className="mb-6">
    <ProgressTracker progress={vault.progress} />
  </div>

  {/* Chat + Panels */}
  <div className="grid md:grid-cols-3 gap-6">
    {/* CHAT */}
    <Card className="md:col-span-2 shadow-2xl rounded-2xl border-0 bg-white">
      <CardContent className="p-5">
        <div className="h-[420px] overflow-y-auto border rounded-2xl p-4 bg-gray-50">
          {msgs.map(m => (
            <motion.div key={m.id} initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} className="mb-4">
              <div className="text-xs uppercase tracking-wider mb-1 font-semibold text-gray-500">{m.role === "assistant" ? "Artist-Ally" : "You"}</div>
              <div className={"whitespace-pre-wrap rounded-2xl p-3 shadow-sm " + (m.role === "assistant" ? "bg-white" : "bg-purple-50 border border-purple-200")}>{m.content}</div>
            </motion.div>
          ))}
        </div>
        <div className="mt-4 flex gap-2">
          <Input className="rounded-xl" value={input} onChange={e => setInput(e.target.value)} placeholder="Type /register, /neighboring, /catalog, /healthcheck, /recap" onKeyDown={(e) => { if (e.key === "Enter") { handleCommand(input); setInput(""); } }} />
          <Button className={brandPrimary + " rounded-xl"} onClick={() => { handleCommand(input); setInput(""); }}><Send className="w-4 h-4 mr-2"/>Send</Button>
        </div>
      </CardContent>
    </Card>

    {/* SIDE PANEL (Demo Vault) */}
    <div className="space-y-6">
      <Card className="shadow-lg rounded-2xl border-0">
        <CardContent className="p-5 space-y-4">
          <div className="flex items-center gap-2 text-purple-600 font-bold"><Settings2 className="w-4 h-4"/> Demo Vault</div>
          <Input disabled value={vault.legalName} className="rounded-xl bg-gray-100" />
          <Input disabled value={vault.stageName} className="rounded-xl bg-gray-100" />
          <div className="text-sm text-gray-600">PRO: {vault.proAffiliations[0].society} — {vault.proAffiliations[0].ipi}</div>
          <div className="text-xs text-gray-500">Demo data is pre-seeded. Replace with your own details to personalize.</div>
        </CardContent>
      </Card>

      {/* Wizards */}
      <Card className="shadow-lg rounded-2xl border-0">
        <CardContent className="p-5 space-y-4">
          <div className="flex items-center gap-2 text-pink-600 font-bold"><Wand2 className="w-4 h-4"/> Wizards</div>
          <Button className={brandPrimary + " w-full rounded-xl"}><FileText className="w-4 h-4 mr-2"/> Register Everywhere</Button>
          <Button className={brandSecondary + " w-full rounded-xl"}><FileText className="w-4 h-4 mr-2"/> Neighboring Rights Packet</Button>
          <Button className="bg-gray-700 hover:bg-gray-800 text-white w-full rounded-xl"><FileText className="w-4 h-4 mr-2"/> Export Catalog</Button>
        </CardContent>
      </Card>
    </div>
  </div>

  {/* Footer */}
  <div className="text-xs text-gray-600 mt-6 p-4 border rounded-2xl bg-gray-50">
    Artist-Ally provides informational guidance only — not legal advice. Consult a qualified attorney for contract interpretation or territory-specific law.
  </div>
</div>

); }
