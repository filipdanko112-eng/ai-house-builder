import React, { useState, useEffect } from "react";
import { Canvas } from "@react-three/fiber";
import { OrbitControls } from "@react-three/drei";
import jsPDF from "jspdf";

/*
AI HOUSE BUILDER - STARTUP WEB MVP

Pages included:
1 Landing page (marketing)
2 Login / account creation
3 3D house configurator
4 Lead capture
5 Company dashboard

This is a FULL startup web prototype that can be deployed.
*/

// ======================================================
// UTILITIES
// ======================================================

function normalizeFloors(value) {
  const n = parseInt(value, 10);
  if (Number.isNaN(n) || n < 1) return 1;
  if (n > 3) return 3;
  return n;
}

function normalizeNumber(value, fallback = 0) {
  const n = Number(value);
  if (Number.isNaN(n) || n < 0) return fallback;
  return n;
}

// ======================================================
// PRICE ENGINE
// ======================================================

function calculateHousePrice({ size, type, floors, garage, solar, garden, pool }) {
  const basePerM2 = {
    modern: 1400,
    classic: 1200,
    luxury: 2200
  };

  const floorMultiplier = { 1: 1, 2: 1.6, 3: 2.2 };

  const extras = {
    garage: 15000,
    solar: 12000,
    garden: 8000,
    pool: 25000
  };

  const floorsNum = normalizeFloors(floors);

  const base = size * (basePerM2[type] || basePerM2.modern) * floorMultiplier[floorsNum];

  let extrasTotal = 0;

  if (garage) extrasTotal += extras.garage;
  if (solar) extrasTotal += extras.solar;
  if (garden) extrasTotal += extras.garden;
  if (pool) extrasTotal += extras.pool;

  return Math.round(base + extrasTotal);
}

// ======================================================
// 3D HOUSE
// ======================================================

function House3D({ floors, color, garage, solar, pool }) {
  const height = normalizeFloors(floors) * 1.2;

  return (
    <group>
      <mesh position={[0, height / 2, 0]}>
        <boxGeometry args={[2, height, 2]} />
        <meshStandardMaterial color={color} />
      </mesh>

      {garage && (
        <mesh position={[2, 0.5, 0]}>
          <boxGeometry args={[1.5, 1, 2]} />
          <meshStandardMaterial color="#777" />
        </mesh>
      )}

      {solar && (
        <mesh position={[0, height + 0.2, 0]}>
          <boxGeometry args={[1.5, 0.1, 1]} />
          <meshStandardMaterial color="#0ea5e9" />
        </mesh>
      )}

      {pool && (
        <mesh position={[0, -0.1, 3]}>
          <boxGeometry args={[2, 0.2, 1.5]} />
          <meshStandardMaterial color="#38bdf8" />
        </mesh>
      )}
    </group>
  );
}

function HouseViewer(props) {
  return (
    <div style={{ height: 420 }}>
      <Canvas camera={{ position: [5, 4, 5] }}>
        <ambientLight intensity={0.7} />
        <directionalLight position={[5, 5, 5]} />
        <House3D {...props} />
        <OrbitControls />
      </Canvas>
    </div>
  );
}

// ======================================================
// PDF OFFER
// ======================================================

function exportPDF(config, price) {
  const doc = new jsPDF();

  doc.text("House Offer", 20, 20);
  doc.text(`Type: ${config.type}`, 20, 40);
  doc.text(`Size: ${config.size} m2`, 20, 50);
  doc.text(`Floors: ${config.floors}`, 20, 60);
  doc.text(`Price: ${price.toLocaleString()} EUR`, 20, 80);

  doc.save("house-offer.pdf");
}

// ======================================================
// LANDING PAGE
// ======================================================

function Landing({ start }) {
  return (
    <div style={{ fontFamily: "sans-serif", padding: 60, textAlign: "center" }}>
      <h1>AI House Builder</h1>
      <p>3D konfigurátor domov pre developerov a stavebné firmy</p>

      <button onClick={start} style={{ padding: 15, fontSize: 18 }}>
        Spustiť konfigurátor
      </button>

      <div style={{ marginTop: 60 }}>
        <h2>Prečo tento nástroj?</h2>
        <p>• klient si navrhne dom v 3D</p>
        <p>• okamžitá cenová ponuka</p>
        <p>• automatické leady pre firmu</p>
        <p>• PDF ponuka pre zákazníka</p>
      </div>
    </div>
  );
}

// ======================================================
// LOGIN
// ======================================================

function Login({ onLogin }) {
  const [email, setEmail] = useState("");

  function login() {
    if (!email) return;

    const company = { email };

    localStorage.setItem("company", JSON.stringify(company));

    onLogin(company);
  }

  return (
    <div style={{ padding: 60 }}>
      <h2>Login firmy</h2>

      <input
        placeholder="Email firmy"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />

      <br />
      <br />

      <button onClick={login}>Prihlásiť</button>
    </div>
  );
}

// ======================================================
// LEAD FORM
// ======================================================

function LeadForm({ price }) {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");

  function submit() {
    const lead = { name, email, price };

    const leads = JSON.parse(localStorage.getItem("leads") || "[]");

    leads.push(lead);

    localStorage.setItem("leads", JSON.stringify(leads));

    alert("Dopyt odoslaný");

    setName("");
    setEmail("");
  }

  return (
    <div style={{ marginTop: 30 }}>
      <h3>Získať ponuku</h3>

      <input placeholder="Meno" value={name} onChange={(e) => setName(e.target.value)} />

      <br />

      <input placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />

      <br />

      <button onClick={submit}>Odoslať</button>
    </div>
  );
}

// ======================================================
// DASHBOARD
// ======================================================

function Dashboard() {
  const [leads, setLeads] = useState([]);

  useEffect(() => {
    const stored = JSON.parse(localStorage.getItem("leads") || "[]");
    setLeads(stored);
  }, []);

  return (
    <div style={{ marginTop: 40 }}>
      <h2>Leady</h2>

      {leads.map((l, i) => (
        <div key={i} style={{ border: "1px solid #ddd", padding: 10, marginTop: 10 }}>
          <b>{l.name}</b>
          <div>{l.email}</div>
          <div>{l.price?.toLocaleString()} EUR</div>
        </div>
      ))}
    </div>
  );
}

// ======================================================
// CONFIGURATOR
// ======================================================

function Configurator() {
  const [type, setType] = useState("modern");
  const [size, setSize] = useState(120);
  const [floors, setFloors] = useState(1);

  const [garage, setGarage] = useState(false);
  const [solar, setSolar] = useState(false);
  const [garden, setGarden] = useState(false);
  const [pool, setPool] = useState(false);

  const [color, setColor] = useState("#e5e5e5");

  const price = calculateHousePrice({
    size: normalizeNumber(size),
    type,
    floors,
    garage,
    solar,
    garden,
    pool
  });

  const config = { type, size, floors };

  return (
    <div style={{ maxWidth: 1100, margin: "auto", padding: 30 }}>
      <h1>3D Konfigurátor domu</h1>

      <HouseViewer floors={floors} color={color} garage={garage} solar={solar} pool={pool} />

      <div style={{ display: "grid", gap: 10, marginTop: 20 }}>
        <label>
          Typ domu
          <select value={type} onChange={(e) => setType(e.target.value)}>
            <option value="modern">Moderný</option>
            <option value="classic">Klasický</option>
            <option value="luxury">Luxusný</option>
          </select>
        </label>

        <label>
          Veľkosť
          <input type="number" value={size} onChange={(e) => setSize(e.target.value)} />
        </label>

        <label>
          Poschodia
          <select value={floors} onChange={(e) => setFloors(parseInt(e.target.value))}>
            <option value={1}>1</option>
            <option value={2}>2</option>
            <option value={3}>3</option>
          </select>
        </label>

        <label>
          Farba
          <input type="color" value={color} onChange={(e) => setColor(e.target.value)} />
        </label>

        <label>
          <input type="checkbox" checked={garage} onChange={(e) => setGarage(e.target.checked)} /> Garaz
        </label>

        <label>
          <input type="checkbox" checked={solar} onChange={(e) => setSolar(e.target.checked)} /> Solar
        </label>

        <label>
          <input type="checkbox" checked={garden} onChange={(e) => setGarden(e.target.checked)} /> Zahrada
        </label>

        <label>
          <input type="checkbox" checked={pool} onChange={(e) => setPool(e.target.checked)} /> Bazen
        </label>

        <h2>Cena: {price.toLocaleString()} EUR</h2>

        <button onClick={() => exportPDF(config, price)}>PDF ponuka</button>
      </div>

      <LeadForm price={price} />

      <Dashboard />
    </div>
  );
}

// ======================================================
// APP ROUTER
// ======================================================

export default function App() {
  const [page, setPage] = useState("landing");
  const [company, setCompany] = useState(null);

  useEffect(() => {
    const saved = localStorage.getItem("company");
    if (saved) {
      setCompany(JSON.parse(saved));
      setPage("app");
    }
  }, []);

  if (page === "landing") return <Landing start={() => setPage("login")} />;

  if (page === "login") return <Login onLogin={(c) => { setCompany(c); setPage("app"); }} />;

  if (page === "app") return <Configurator />;

  return null;
}
