# life-pilot
/*
GenesisHealthApp - Single-file React frontend scaffold
Technology stack (suggested):
- React + TypeScript (this file uses TSX style - remove types if using plain JS)
- Tailwind CSS for styling
- Recharts for charts
- shadcn/ui (optional) for UI primitives
- Framer Motion (optional) for subtle animations

What this file contains:
- Default export: App component
- In-file components: Auth (Login/Register), Sidebar, Navbar, Dashboard, Monitoring, Emergency, Profile, AddOns
- Mock authentication using localStorage (simple JWT-like token)
- Forms for creating Patient Profiles, Health Records, Emergency Contacts
- Medication reminder toggles (UI only)
- Health data processor: simple analyzer UI that flags trends
- Responsive, card-based layout with a vibrant medical theme

How to use:
1) Create a new React + Vite + Tailwind project (recommended):
   npm create vite@latest genesis-health --template react-ts
   cd genesis-health
   npm install

2) Install dependencies:
   npm install tailwindcss postcss autoprefixer
   npx tailwindcss init -p
   npm install recharts framer-motion @radix-ui/react-icons
   (or use shadcn/ui components if you prefer)

3) Configure Tailwind per their docs and add Tailwind classes to index.css.

4) Put this file content into src/App.tsx (or src/GenesisApp.tsx) and import into main.tsx.

5) Run:
   npm run dev

Notes on Authentication:
- The demo uses a simple localStorage token to simulate authentication.
- Recommended production options: Firebase Auth, Auth0, Clerk, or custom JWT backend.

This single-file scaffold is meant to be a clear, modular starting point. Replace mock functions and placeholder data with real APIs.
*/

import React, { useEffect, useMemo, useState } from "react";
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer } from "recharts";
import { motion } from "framer-motion";

// ---------- Types (remove if using plain JS) ----------
type User = {
  id: string;
  name: string;
  email: string;
};

// ---------- Mock helpers ----------
const fakeAuth = {
  login: (email: string, password: string) => {
    // naive validation
    if (!email.includes("@") || password.length < 4) return null;
    const token = btoa(email + ":token:" + Date.now());
    const user = { id: String(Date.now()), name: email.split("@")[0], email };
    localStorage.setItem("gh_token", token);
    localStorage.setItem("gh_user", JSON.stringify(user));
    return { token, user };
  },
  register: (name: string, email: string, password: string) => {
    // naive register
    return fakeAuth.login(email, password);
  },
  logout: () => {
    localStorage.removeItem("gh_token");
    localStorage.removeItem("gh_user");
  },
  getUser: (): User | null => {
    const s = localStorage.getItem("gh_user");
    return s ? JSON.parse(s) : null;
  },
};

// ---------- Sample data ----------
const sampleVitals = [
  { time: "2025-10-25", hr: 72, bp: 120, steps: 4500 },
  { time: "2025-10-26", hr: 75, bp: 124, steps: 6000 },
  { time: "2025-10-27", hr: 80, bp: 130, steps: 3000 },
  { time: "2025-10-28", hr: 78, bp: 128, steps: 7000 },
  { time: "2025-10-29", hr: 70, bp: 118, steps: 8000 },
  { time: "2025-10-30", hr: 68, bp: 116, steps: 10000 },
  { time: "2025-10-31", hr: 74, bp: 122, steps: 4000 },
];

// ---------- Styling tokens (Tailwind classes) ----------
const brand = {
  primary: "bg-gradient-to-br from-cyan-400 to-blue-600",
  card: "bg-white/90 dark:bg-gray-800/80",
  accentText: "text-blue-700",
};

// ---------- Small UI components ----------
function Logo() {
  return (
    <div className="flex items-center gap-2">
      <div className="w-10 h-10 rounded-2xl flex items-center justify-center shadow-md bg-gradient-to-br from-emerald-400 to-teal-500 text-white font-bold">GH</div>
      <div>
        <div className="text-lg font-semibold">Genesis</div>
        <div className="text-sm text-gray-500">Health Suite</div>
      </div>
    </div>
  );
}

function IconButton({ children, onClick }: any) {
  return (
    <button
      onClick={onClick}
      className="p-2 rounded-lg hover:shadow-md transition-shadow border border-transparent hover:border-blue-200"
    >
      {children}
    </button>
  );
}

// ---------- Auth UI ----------
function AuthPanel({ onAuth }: { onAuth: (u: User) => void }) {
  const [mode, setMode] = useState<"login" | "register">("login");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [name, setName] = useState("");
  const [error, setError] = useState("");

  const submit = () => {
    setError("");
    if (mode === "login") {
      const res = fakeAuth.login(email, password);
      if (!res) return setError("Invalid credentials (demo). Use a valid email and password >=4 chars.");
      onAuth(res.user);
    } else {
      const res = fakeAuth.register(name, email, password);
      if (!res) return setError("Unable to register (demo).");
      onAuth(res.user);
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center p-6 bg-gradient-to-b from-slate-50 to-white">
      <div className="max-w-3xl w-full grid grid-cols-1 md:grid-cols-2 gap-6 items-center">
        <motion.div
          initial={{ opacity: 0, x: -30 }}
          animate={{ opacity: 1, x: 0 }}
          className="p-6 rounded-2xl shadow-lg"
          style={{ background: "linear-gradient(180deg,#f8fbff, #ffffff)" }}
        >
          <Logo />
          <h2 className="mt-6 text-2xl font-bold">Welcome back</h2>
          <p className="text-gray-500 mt-2">Sign in to manage patients, vitals and reminders.</p>

          <div className="mt-6 space-y-4">
            {mode === "register" && (
              <input
                className="w-full p-3 rounded-xl border"
                placeholder="Full name"
                value={name}
                onChange={(e) => setName(e.target.value)}
              />
            )}
            <input className="w-full p-3 rounded-xl border" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
            <input
              className="w-full p-3 rounded-xl border"
              placeholder="Password"
              type="password"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
            />
            <div className="flex items-center justify-between">
              <button onClick={submit} className="px-4 py-2 rounded-xl font-semibold text-white " style={{ background: "linear-gradient(90deg,#06b6d4,#3b82f6)" }}>
                {mode === "login" ? "Sign in" : "Create account"}
              </button>
              <button className="text-sm text-gray-500" onClick={() => setMode(mode === "login" ? "register" : "login")}>{mode === "login" ? "Create account" : "Have an account?"}</button>
            </div>
            {error && <div className="text-red-500 text-sm">{error}</div>}
          </div>
        </motion.div>

        <motion.div initial={{ opacity: 0, x: 30 }} animate={{ opacity: 1, x: 0 }} className="p-6 rounded-2xl shadow-lg flex flex-col gap-4 items-start justify-center" style={{ background: "linear-gradient(180deg,#ffffff,#f3f8ff)" }}>
          <h3 className="text-lg font-semibold">Genesis Health — what you'll get</h3>
          <ul className="list-disc pl-5 text-gray-600 space-y-2">
            <li>Patient profiles & emergency contacts</li>
            <li>Vitals monitoring and visualizations</li>
            <li>Medication reminders & adherence tracking</li>
            <li>One-tap Emergency SOS</li>
            <li>AI Health Assistant (chat) — connect later</li>
          </ul>
          <div className="w-full mt-4">
            <div className="text-sm text-gray-500">Demo credentials</div>
            <div className="mt-2 text-xs text-gray-600">Email: demo@example.com • Password: demo1234</div>
          </div>
        </motion.div>
      </div>
    </div>
  );
}

// ---------- Navigation & Layout ----------
function Navbar({ user, onLogout }: { user: User | null; onLogout: () => void }) {
  return (
    <div className="flex items-center justify-between px-4 py-3 border-b">
      <div className="flex items-center gap-4">
        <Logo />
      </div>
      <div className="flex items-center gap-3">
        <div className="hidden md:flex flex-col text-right">
          <div className="text-sm font-medium">{user?.name}</div>
          <div className="text-xs text-gray-500">{user?.email}</div>
        </div>
        <button onClick={onLogout} className="px-3 py-2 rounded-lg bg-red-50 text-red-600 border border-red-100">Logout</button>
      </div>
    </div>
  );
}

function Sidebar({ active, setActive }: { active: string; setActive: (s: string) => void }) {
  const items = ["Dashboard", "Monitoring", "Patients", "Emergency", "Add-ons", "Settings"];
  return (
    <div className="w-64 hidden md:block p-4">
      <div className="space-y-2">
        {items.map((it) => (
          <button key={it} onClick={() => setActive(it)} className={`w-full text-left p-3 rounded-lg ${active === it ? "bg-blue-50 border border-blue-100" : "hover:bg-gray-50"}`}>
            {it}
          </button>
        ))}
      </div>
    </div>
  );
}

// ---------- Dashboard ----------
function StatCard({ title, value, children }: any) {
  return (
    <div className="p-4 rounded-2xl shadow-sm border h-full">
      <div className="text-sm text-gray-500">{title}</div>
      <div className="text-2xl font-bold mt-2">{value}</div>
      <div className="mt-3 text-sm text-gray-600">{children}</div>
    </div>
  );
}

function MiniVitalsChart({ data }: { data: any[] }) {
  return (
    <div className="h-40">
      <ResponsiveContainer width="100%" height="100%">
        <AreaChart data={data}>
          <defs>
            <linearGradient id="colorHr" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor="#06b6d4" stopOpacity={0.8} />
              <stop offset="95%" stopColor="#06b6d4" stopOpacity={0.05} />
            </linearGradient>
          </defs>
          <XAxis dataKey="time" hide />
          <YAxis hide />
          <Tooltip />
          <Area type="monotone" dataKey="hr" stroke="#06b6d4" fillOpacity={1} fill="url(#colorHr)" />
        </AreaChart>
      </ResponsiveContainer>
    </div>
  );
}

function Dashboard({ vitals }: { vitals: any[] }) {
  const avgHr = Math.round(vitals.reduce((s, v) => s + v.hr, 0) / vitals.length);
  const steps = vitals.reduce((s, v) => s + v.steps, 0);
  return (
    <div className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div className="md:col-span-2">
          <div className="p-4 rounded-2xl shadow-sm">
            <div className="flex items-center justify-between">
              <div>
                <div className="text-sm text-gray-500">Health Overview</div>
                <div className="text-xl font-semibold">Weekly snapshot</div>
              </div>
              <div className="text-sm text-gray-500">Last 7 days</div>
            </div>
            <div className="mt-4">
              <MiniVitalsChart data={vitals} />
            </div>
          </div>
        </div>

        <div className="space-y-4">
          <StatCard title="Avg Heart Rate" value={`${avgHr} bpm`}>
            <div>Stable — keep up with light exercise</div>
          </StatCard>
          <StatCard title="Total Steps" value={`${steps.toLocaleString()}`}>
            <div>Great — daily steps trend improving</div>
          </StatCard>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div className="md:col-span-2 p-4 rounded-2xl shadow-sm">
          <h4 className="font-semibold">Medication adherence</h4>
          <div className="mt-3 grid grid-cols-1 md:grid-cols-3 gap-3">
            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-500">Morning</div>
              <div className="font-medium">Amlodipine</div>
              <div className="text-xs text-gray-500 mt-2">Taken: 6/7</div>
            </div>
            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-500">Afternoon</div>
              <div className="font-medium">Metformin</div>
              <div className="text-xs text-gray-500 mt-2">Taken: 5/7</div>
            </div>
            <div className="p-3 rounded-lg border">
              <div className="text-sm text-gray-500">Night</div>
              <div className="font-medium">Aspirin</div>
              <div className="text-xs text-gray-500 mt-2">Taken: 7/7</div>
            </div>
          </div>
        </div>

        <div className="p-4 rounded-2xl shadow-sm">
          <h4 className="font-semibold">Upcoming</h4>
          <div className="mt-3 space-y-3">
            <div className="p-3 rounded-lg border">Medication reminder • 8:00 AM</div>
            <div className="p-3 rounded-lg border">Doctor appointment • 2025-11-05</div>
            <div className="p-3 rounded-lg border">Routine check-up • 2025-12-01</div>
          </div>
        </div>
      </div>
    </div>
  );
}

// ---------- Monitoring page ----------
function MonitoringPage({ vitals, onFlag }: { vitals: any[]; onFlag: (s: string) => void }) {
  const [threshold, setThreshold] = useState(100);
  useEffect(() => {
    const high = vitals.some((v) => v.hr > threshold);
    if (high) onFlag(`Alert: heart rate exceeded ${threshold} bpm in last 7 days`);
  }, [threshold, vitals, onFlag]);

  return (
    <div className="space-y-6">
      <div className="p-4 rounded-2xl shadow-sm">
        <h3 className="font-semibold">Vitals Monitor</h3>
        <div className="mt-3">Adjust auto-flag threshold for heart rate</div>
        <div className="mt-4 flex items-center gap-4">
          <input type="range" min={60} max={140} value={threshold} onChange={(e) => setThreshold(Number(e.target.value))} />
          <div className="font-mono">{threshold} bpm</div>
        </div>
      </div>

      <div className="p-4 rounded-2xl shadow-sm">
        <h4 className="font-semibold">Raw data</h4>
        <div className="mt-3 overflow-x-auto">
          <table className="w-full text-sm">
            <thead className="text-left text-gray-500">
              <tr>
                <th className="p-2">Date</th>
                <th className="p-2">Heart rate</th>
                <th className="p-2">BP</th>
                <th className="p-2">Steps</th>
              </tr>
            </thead>
            <tbody>
              {vitals.map((v) => (
                <tr key={v.time} className="border-t">
                  <td className="p-2">{v.time}</td>
                  <td className="p-2">{v.hr} bpm</td>
                  <td className="p-2">{v.bp} mmHg</td>
                  <td className="p-2">{v.steps}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      </div>
    </div>
  );
}

// ---------- Patients & Emergency ----------
function PatientsPage() {
  const [patients, setPatients] = useState<any[]>(() => {
    try {
      const s = localStorage.getItem("gh_patients");
      return s ? JSON.parse(s) : [];
    } catch {
      return [];
    }
  });
  const [form, setForm] = useState({ name: "", age: "", disease: "" });

  useEffect(() => {
    localStorage.setItem("gh_patients", JSON.stringify(patients));
  }, [patients]);

  const add = () => {
    setPatients((p) => [...p, { id: Date.now(), ...form }]);
    setForm({ name: "", age: "", disease: "" });
  };

  return (
    <div className="space-y-6">
      <div className="p-4 rounded-2xl shadow-sm">
        <h3 className="font-semibold">Patient Profiles</h3>
        <div className="mt-3 grid grid-cols-1 md:grid-cols-3 gap-3">
          <input value={form.name} onChange={(e) => setForm({ ...form, name: e.target.value })} placeholder="Name" className="p-2 border rounded" />
          <input value={form.age} onChange={(e) => setForm({ ...form, age: e.target.value })} placeholder="Age" className="p-2 border rounded" />
          <input value={form.disease} onChange={(e) => setForm({ ...form, disease: e.target.value })} placeholder="Disease" className="p-2 border rounded" />
        </div>
        <div className="mt-3">
          <button onClick={add} className="px-4 py-2 rounded-lg text-white" style={{ background: "linear-gradient(90deg,#06b6d4,#3b82f6)" }}>
            Add patient
          </button>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
        {patients.map((p) => (
          <div key={p.id} className="p-4 rounded-lg border">
            <div className="font-medium">{p.name}</div>
            <div className="text-xs text-gray-500">Age: {p.age}</div>
            <div className="text-sm mt-2">Condition: {p.disease}</div>
          </div>
        ))}
      </div>
    </div>
  );
}

function EmergencyPage() {
  const [contacts, setContacts] = useState<any[]>(() => {
    const s = localStorage.getItem("gh_contacts");
    return s ? JSON.parse(s) : [];
  });
  const [form, setForm] = useState({ name: "", phone: "", relationship: "" });

  useEffect(() => {
    localStorage.setItem("gh_contacts", JSON.stringify(contacts));
  }, [contacts]);

  const add = () => {
    setContacts((c) => [...c, { id: Date.now(), ...form }]);
    setForm({ name: "", phone: "", relationship: "" });
  };

  const triggerSOS = () => {
    // In production: request user's geolocation, call backend to notify contacts via SMS / call, etc.
    alert("SOS activated — demo: will notify saved contacts and show instructions.");
  };

  return (
    <div className="space-y-6">
      <div className="p-4 rounded-2xl shadow-sm">
        <h3 className="font-semibold">Emergency Contacts</h3>
        <div className="mt-3 grid grid-cols-1 md:grid-cols-3 gap-3">
          <input value={form.name} onChange={(e) => setForm({ ...form, name: e.target.value })} placeholder="Name" className="p-2 border rounded" />
          <input value={form.phone} onChange={(e) => setForm({ ...form, phone: e.target.value })} placeholder="Phone" className="p-2 border rounded" />
          <input value={form.relationship} onChange={(e) => setForm({ ...form, relationship: e.target.value })} placeholder="Relationship" className="p-2 border rounded" />
        </div>
        <div className="mt-3 flex gap-3">
          <button onClick={add} className="px-4 py-2 rounded-lg text-white" style={{ background: "linear-gradient(90deg,#06b6d4,#3b82f6)" }}>
            Add contact
          </button>
          <button onClick={triggerSOS} className="px-4 py-2 rounded-lg bg-red-600 text-white shadow-lg">SOS • One Tap</button>
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-3">
        {contacts.map((c) => (
          <div key={c.id} className="p-3 rounded-lg border">
            <div className="font-medium">{c.name}</div>
            <div className="text-xs text-gray-500">{c.relationship}</div>
            <div className="text-sm mt-2">{c.phone}</div>
          </div>
        ))}
      </div>
    </div>
  );
}

// ---------- Addons & Settings ----------
function AddonsPage() {
  return (
    <div className="space-y-6">
      <div className="p-4 rounded-2xl shadow-sm">
        <h3 className="font-semibold">Optional Integrations</h3>
        <div className="mt-3 grid grid-cols-1 md:grid-cols-3 gap-3">
          <div className="p-3 rounded-lg border">Google Fit • Connect to sync steps & sleep</div>
          <div className="p-3 rounded-lg border">Voice Assistant • Conversational check-ins</div>
          <div className="p-3 rounded-lg border">Chatbot • AI Health Assistant (connect endpoint)</div>
        </div>
      </div>
    </div>
  );
}

function SettingsPage() {
  return (
    <div className="p-4 rounded-2xl shadow-sm">
      <h3 className="font-semibold">Settings</h3>
      <div className="mt-3 text-sm text-gray-600">Theme: Light (default) • Dark mode available via OS preference</div>
    </div>
  );
}

// ---------- Main App ----------
export default function App() {
  const [user, setUser] = useState<User | null>(() => fakeAuth.getUser());
  const [active, setActive] = useState<string>("Dashboard");
  const [vitals, setVitals] = useState<any[]>(sampleVitals);
  const [alerts, setAlerts] = useState<string[]>([]);

  useEffect(() => {
    const u = fakeAuth.getUser();
    setUser(u);
  }, []);

  const handleLogout = () => {
    fakeAuth.logout();
    setUser(null);
  };

  const onFlag = (msg: string) => setAlerts((a) => [msg, ...a].slice(0, 5));

  if (!user) return <AuthPanel onAuth={(u: User) => setUser(u)} />;

  return (
    <div className="min-h-screen bg-slate-50 text-slate-900">
      <div className="max-w-7xl mx-auto">
        <Navbar user={user} onLogout={handleLogout} />
        <div className="flex gap-6">
          <Sidebar active={active} setActive={setActive} />

          <main className="flex-1 p-6">
            <div className="flex items-start justify-between">
              <h1 className="text-2xl font-semibold">{active}</h1>
              <div className="flex items-center gap-3">
                {alerts.length > 0 && (
                  <div className="p-2 rounded-lg bg-yellow-50 border">{alerts[0]}</div>
                )}
                <div className="text-sm text-gray-500">November 1, 2025</div>
              </div>
            </div>

            <div className="mt-6">
              {active === "Dashboard" && <Dashboard vitals={vitals} />}
              {active === "Monitoring" && <MonitoringPage vitals={vitals} onFlag={onFlag} />}
              {active === "Patients" && <PatientsPage />}
              {active === "Emergency" && <EmergencyPage />}
              {active === "Add-ons" && <AddonsPage />}
              {active === "Settings" && <SettingsPage />}
            </div>
          </main>
        </div>
      </div>
    </div>
  );
}
