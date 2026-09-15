[obramarket-admin.html](https://github.com/user-attachments/files/32263329/obramarket-admin.html)
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>DepoMarket — Panel Interno</title>
<script src="https://cdn.jsdelivr.net/npm/react@18.2.0/umd/react.production.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/react-dom@18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@babel/standalone@7.24.7/babel.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.0/firebase-firestore-compat.js"></script>
<style>
  * { box-sizing: border-box; }
  body { margin: 0; }
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel" data-presets="react">

// ======================================================================
// CONFIGURACIÓN DE FIREBASE — mismo proyecto que usa el archivo cliente.
// ======================================================================
const firebaseConfig = {
  apiKey: "AIzaSyAzyoIoEgtFAGWjIUCLsBAjILTT0KPPGlg",
  authDomain: "obramarket-eb28f.firebaseapp.com",
  projectId: "obramarket-eb28f",
  storageBucket: "obramarket-eb28f.firebasestorage.app",
  messagingSenderId: "687534993083",
  appId: "1:687534993083:web:7ce08d8babace8039c039e",
};
firebase.initializeApp(firebaseConfig);
const db = firebase.firestore();
// ======================================================================

import React from "react";

// ---------- Paleta (misma que el cliente) ----------
const COLOR = {
  amarillo: "#F5B700",
  ladrillo: "#B5451B",
  fondo: "#EDE4D3",
  grafito: "#1B2E4B",
  grafitoSuave: "#5C6B80",
  tarjeta: "#FFFBF2",
  borde: "#DCCEB0",
};
const F_DISPLAY = "'Arial Black', 'Arial Narrow', sans-serif";
const F_BODY = "'Helvetica Neue', Arial, sans-serif";

function EstadoBadge({ estado }) {
  const config = {
    enviada: { color: COLOR.grafitoSuave, bg: "rgba(92,107,128,0.12)", label: "Enviada" },
    respondida: { color: COLOR.ladrillo, bg: "rgba(181,69,27,0.12)", label: "Respondida" },
    aceptada: { color: "#B08900", bg: "rgba(245,183,0,0.18)", label: "Aceptada" },
    pagada: { color: "#2E7D32", bg: "rgba(46,125,50,0.12)", label: "Pagada" },
    entregado: { color: "#1B5E20", bg: "rgba(27,94,32,0.15)", label: "✓ Entregado" },
  };
  const c = config[estado] || config.enviada;
  return (
    <span style={{ fontSize: 11, fontWeight: 800, padding: "3px 9px", borderRadius: 20, background: c.bg, color: c.color }}>
      {c.label}
    </span>
  );
}

// ---------- Control de facturación e IVA ----------
// Lista todas las facturas emitidas (cotizaciones marcadas como pagadas/entregadas,
// que son las que ya generaron número de factura) con su desglose, y suma los
// totales que necesitás para la declaración de IVA (modelo 303).
function FacturacionControl({ cotizaciones, onEditarCotizacion }) {
  const MESES = ["enero", "febrero", "marzo", "abril", "mayo", "junio", "julio", "agosto", "septiembre", "octubre", "noviembre", "diciembre"];

  const facturadas = cotizaciones
    .filter((c) => c.numeroFactura)
    .sort((a, b) => (a.numeroFactura < b.numeroFactura ? 1 : -1));

  // Agrupa por mes para armar las pestañas
  const grupos = {};
  facturadas.forEach((c) => {
    const fecha = new Date(c.fechaFactura || c.fecha);
    const clave = `${fecha.getFullYear()}-${String(fecha.getMonth()).padStart(2, "0")}`;
    if (!grupos[clave]) grupos[clave] = { label: `${MESES[fecha.getMonth()]} ${fecha.getFullYear()}`, facturas: [] };
    grupos[clave].facturas.push(c);
  });
  const clavesOrdenadas = Object.keys(grupos).sort((a, b) => (a < b ? 1 : -1));

  const [mesSeleccionado, setMesSeleccionado] = React.useState(clavesOrdenadas[0] || "todos");

  const listaMostrada = mesSeleccionado === "todos" ? facturadas : grupos[mesSeleccionado]?.facturas || [];

  const totales = listaMostrada.reduce(
    (acc, c) => {
      const d = c.desglose || { subtotal: 0, transporte: 0, iva: 0, total: c.precioTotal || 0 };
      acc.baseImponible += (d.subtotal || 0) + (d.transporte || 0);
      acc.iva += d.iva || 0;
      acc.total += d.total || c.precioTotal || 0;
      return acc;
    },
    { baseImponible: 0, iva: 0, total: 0 }
  );

  return (
    <div style={{ maxWidth: 900, margin: "0 auto", padding: "20px 16px" }}>
      <div style={{ fontFamily: F_DISPLAY, fontWeight: 900, fontSize: 16, marginBottom: 14 }}>Control de IVA y pagos</div>

      {facturadas.length > 0 && (
        <div style={{ display: "flex", gap: 6, marginBottom: 16, overflowX: "auto", paddingBottom: 4 }}>
          <button
            onClick={() => setMesSeleccionado("todos")}
            style={{ flexShrink: 0, padding: "8px 14px", borderRadius: 8, border: `1px solid ${mesSeleccionado === "todos" ? COLOR.grafito : COLOR.borde}`, background: mesSeleccionado === "todos" ? COLOR.grafito : COLOR.tarjeta, color: mesSeleccionado === "todos" ? COLOR.fondo : COLOR.grafito, fontWeight: 700, fontSize: 12.5, cursor: "pointer", whiteSpace: "nowrap", textTransform: "capitalize" }}
          >
            Todos los meses
          </button>
          {clavesOrdenadas.map((clave) => (
            <button
              key={clave}
              onClick={() => setMesSeleccionado(clave)}
              style={{ flexShrink: 0, padding: "8px 14px", borderRadius: 8, border: `1px solid ${mesSeleccionado === clave ? COLOR.grafito : COLOR.borde}`, background: mesSeleccionado === clave ? COLOR.grafito : COLOR.tarjeta, color: mesSeleccionado === clave ? COLOR.fondo : COLOR.grafito, fontWeight: 700, fontSize: 12.5, cursor: "pointer", whiteSpace: "nowrap", textTransform: "capitalize" }}
            >
              {grupos[clave].label} ({grupos[clave].facturas.length})
            </button>
          ))}
        </div>
      )}

      <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(140px, 1fr))", gap: 10, marginBottom: 20 }}>
        <div style={{ background: COLOR.tarjeta, border: `1px solid ${COLOR.borde}`, borderRadius: 10, padding: 14 }}>
          <div style={{ fontSize: 11, color: COLOR.grafitoSuave, marginBottom: 4 }}>
            Facturas emitidas {mesSeleccionado !== "todos" && <span style={{ textTransform: "capitalize" }}>· {grupos[mesSeleccionado]?.label}</span>}
          </div>
          <div style={{ fontSize: 20, fontWeight: 900 }}>{listaMostrada.length}</div>
        </div>
        <div style={{ background: COLOR.tarjeta, border: `1px solid ${COLOR.borde}`, borderRadius: 10, padding: 14 }}>
          <div style={{ fontSize: 11, color: COLOR.grafitoSuave, marginBottom: 4 }}>Base imponible</div>
          <div style={{ fontSize: 20, fontWeight: 900 }}>{totales.baseImponible.toFixed(2)} €</div>
        </div>
        <div style={{ background: COLOR.tarjeta, border: `1px solid ${COLOR.borde}`, borderRadius: 10, padding: 14 }}>
          <div style={{ fontSize: 11, color: COLOR.grafitoSuave, marginBottom: 4 }}>IVA repercutido (21%)</div>
          <div style={{ fontSize: 20, fontWeight: 900, color: COLOR.ladrillo }}>{totales.iva.toFixed(2)} €</div>
        </div>
        <div style={{ background: COLOR.grafito, border: `1px solid ${COLOR.grafito}`, borderRadius: 10, padding: 14 }}>
          <div style={{ fontSize: 11, color: "#CFC5B6", marginBottom: 4 }}>Total facturado</div>
          <div style={{ fontSize: 20, fontWeight: 900, color: COLOR.amarillo }}>{totales.total.toFixed(2)} €</div>
        </div>
      </div>

      <div style={{ fontSize: 11.5, color: COLOR.grafitoSuave, marginBottom: 14, lineHeight: 1.4 }}>
        Esta base imponible y este IVA repercutido (del mes o rango elegido arriba) son los números que le pasás a tu gestor para el modelo 303.
      </div>

      {facturadas.length === 0 ? (
        <div style={{ textAlign: "center", color: COLOR.grafitoSuave, fontSize: 13.5, padding: 40 }}>
          Todavía no se generó ninguna factura. Se genera sola apenas marcás una cotización como "pagada".
        </div>
      ) : (
        <div style={{ background: COLOR.tarjeta, border: `1px solid ${COLOR.borde}`, borderRadius: 10, overflow: "hidden" }}>
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1.3fr 0.9fr 0.9fr 0.9fr auto", gap: 8, padding: "10px 14px", background: COLOR.fondo, fontSize: 11, fontWeight: 800, color: COLOR.grafitoSuave }}>
            <div>Factura Nº</div>
            <div>Cliente</div>
            <div>Base</div>
            <div>IVA</div>
            <div>Total</div>
            <div></div>
          </div>
          {listaMostrada.map((c) => (
            <div key={c.id} style={{ display: "grid", gridTemplateColumns: "1fr 1.3fr 0.9fr 0.9fr 0.9fr auto", gap: 8, padding: "10px 14px", borderTop: `1px solid ${COLOR.borde}`, fontSize: 12.5, alignItems: "center" }}>
              <div style={{ fontWeight: 700 }}>{c.numeroFactura}</div>
              <div style={{ overflow: "hidden", textOverflow: "ellipsis", whiteSpace: "nowrap" }}>
                {c.contacto?.tipoCliente === "Empresa" ? c.contacto?.empresaNombre : c.contacto?.nombre}
              </div>
              <div>{((c.desglose?.subtotal || 0) + (c.desglose?.transporte || 0)).toFixed(2)} €</div>
              <div>{(c.desglose?.iva || 0).toFixed(2)} €</div>
              <div style={{ fontWeight: 800 }}>{c.precioTotal} €</div>
              <button
                onClick={() => onEditarCotizacion(c)}
                title="Ver y trabajar sobre esta factura"
                style={{ padding: "5px 9px", background: "transparent", border: `1px solid ${COLOR.borde}`, borderRadius: 6, fontSize: 11, fontWeight: 700, cursor: "pointer", whiteSpace: "nowrap" }}
              >
                ✏️ Abrir
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

// ---------- Directorio de clientes ----------
// Junta todas las cotizaciones por cliente (usando el número de cliente, o
// el email/celular si no lo tiene) para ver el historial completo de cada uno:
// cuántos pedidos hizo, cuánto gastó, y cuándo fue la última vez.
function ClientesDirectorio({ cotizaciones }) {
  const [busqueda, setBusqueda] = React.useState("");

  const clientesMap = {};
  cotizaciones.forEach((c) => {
    const clave = c.clienteNumero || c.contacto?.email || c.contacto?.celular || c.contacto?.telefono || c.id;
    if (!clientesMap[clave]) {
      clientesMap[clave] = {
        clave,
        clienteNumero: c.clienteNumero || "—",
        nombre: c.contacto?.tipoCliente === "Empresa" ? c.contacto?.empresaNombre || "Empresa" : c.contacto?.nombre || "Sin nombre",
        tipoCliente: c.contacto?.tipoCliente || "Particular",
        celular: c.contacto?.celular || c.contacto?.telefono || "",
        email: c.contacto?.email || "",
        pedidos: [],
        totalGastado: 0,
        ultimaFecha: c.fecha,
      };
    }
    clientesMap[clave].pedidos.push(c);
    clientesMap[clave].totalGastado += c.precioTotal || 0;
    if (new Date(c.fecha) > new Date(clientesMap[clave].ultimaFecha)) clientesMap[clave].ultimaFecha = c.fecha;
  });

  const clientes = Object.values(clientesMap)
    .filter((cl) => {
      const q = busqueda.trim().toLowerCase();
      if (!q) return true;
      return [cl.nombre, cl.clienteNumero, cl.email, cl.celular].some((campo) => (campo || "").toLowerCase().includes(q));
    })
    .sort((a, b) => new Date(b.ultimaFecha) - new Date(a.ultimaFecha));

  return (
    <div style={{ maxWidth: 900, margin: "0 auto", padding: "20px 16px" }}>
      <div style={{ fontFamily: F_DISPLAY, fontWeight: 900, fontSize: 16, marginBottom: 14 }}>Directorio de clientes</div>

      <input
        value={busqueda}
        onChange={(e) => setBusqueda(e.target.value)}
        placeholder="Buscar por nombre, número de cliente, email o celular..."
        style={{ width: "100%", padding: "10px 12px", border: `1px solid ${COLOR.borde}`, borderRadius: 8, fontSize: 13.5, marginBottom: 16, boxSizing: "border-box" }}
      />

      <div style={{ fontSize: 11.5, color: COLOR.grafitoSuave, marginBottom: 14 }}>
        {clientes.length} cliente{clientes.length !== 1 ? "s" : ""} registrado{clientes.length !== 1 ? "s" : ""}
      </div>

      {clientes.length === 0 ? (
        <div style={{ textAlign: "center", color: COLOR.grafitoSuave, fontSize: 13.5, padding: 40 }}>No hay clientes que coincidan con la búsqueda.</div>
      ) : (
        clientes.map((cl) => (
          <div
            key={cl.clave}
            style={{
              border: `1px solid ${COLOR.borde}`,
              borderLeft: `5px solid ${cl.tipoCliente === "Empresa" ? COLOR.grafito : COLOR.amarillo}`,
              borderRadius: 10,
              padding: 14,
              marginBottom: 10,
              background: COLOR.tarjeta,
            }}
          >
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 4 }}>
              <div>
                <div style={{ fontSize: 14, fontWeight: 800 }}>{cl.nombre}</div>
                <div style={{ fontSize: 11, color: COLOR.grafitoSuave }}>
                  {cl.tipoCliente === "Empresa" ? "🏢 Empresa" : "🙋 Particular"} · Nº {cl.clienteNumero}
                </div>
              </div>
              <div style={{ textAlign: "right" }}>
                <div style={{ fontSize: 15, fontWeight: 900, color: COLOR.ladrillo }}>{cl.totalGastado.toFixed(2)} €</div>
                <div style={{ fontSize: 10.5, color: COLOR.grafitoSuave }}>gastado en total</div>
              </div>
            </div>
            <div style={{ fontSize: 11.5, color: COLOR.grafitoSuave, marginBottom: 6 }}>
              {cl.celular && <>📱 {cl.celular} </>}
              {cl.email && <>· ✉️ {cl.email}</>}
            </div>
            <div style={{ fontSize: 11.5, color: COLOR.grafitoSuave }}>
              {cl.pedidos.length} pedido{cl.pedidos.length !== 1 ? "s" : ""} · Última actividad: {new Date(cl.ultimaFecha).toLocaleDateString()}
            </div>
          </div>
        ))
      )}
    </div>
  );
}

function PanelInterno() {
  const [autenticado, setAutenticado] = React.useState(false);
  const [claveIngresada, setClaveIngresada] = React.useState("");
  const [errorClave, setErrorClave] = React.useState(false);
  // ---------- Cotizaciones: base de datos real en la nube (Firebase) ----------
  const [cotizaciones, setCotizaciones] = React.useState([]);
  React.useEffect(() => {
    const dejarDeEscuchar = db
      .collection("cotizaciones")
      .orderBy("fecha", "desc")
      .onSnapshot(
        (snapshot) => {
          const datos = snapshot.docs.map((doc) => ({ id: doc.id, ...doc.data() }));
          setCotizaciones(datos);
        },
        (error) => console.error("Error conectando con Firebase:", error)
      );
    return () => dejarDeEscuchar();
  }, []);

  const [respuestaTemp, setRespuestaTemp] = React.useState({});
  const [editando, setEditando] = React.useState({}); // { [cotizacionId]: true } mientras se está corrigiendo
  const [filtroCliente, setFiltroCliente] = React.useState("todos"); // "todos" | "empresa" | "comun"
  const [filtroEstado, setFiltroEstado] = React.useState("nuevas"); // "nuevas" | "despacho" | "entregadas"
  const [vista, setVista] = React.useState("pedidos"); // "pedidos" | "facturacion"

  const CONTRASENA_TEMPORAL = "12345";

  function siguienteNumeroFactura() {
    // Busca el número más alto ya usado este año en las cotizaciones reales y sigue desde ahí
    const año = new Date().getFullYear();
    const prefijo = `${año}-`;
    let maximo = 0;
    cotizaciones.forEach((c) => {
      if (c.numeroFactura && c.numeroFactura.startsWith(prefijo)) {
        const n = parseInt(c.numeroFactura.slice(prefijo.length), 10);
        if (!isNaN(n) && n > maximo) maximo = n;
      }
    });
    return `${prefijo}${String(maximo + 1).padStart(4, "0")}`;
  }

  const IVA_ESPANA = 0.21; // IVA general — venta de materiales sin instalación siempre tributa al 21%

  function calcularTotales(cotizacionId, items) {
    const r = respuestaTemp[cotizacionId] || {};
    const preciosItems = r.preciosItems || {};
    const subtotal = (items || []).reduce((acc, it) => acc + (parseFloat(preciosItems[it.id]) || 0) * (it.cantidad || 1), 0);
    const transporte = parseFloat(r.transporte) || 0;
    const base = subtotal + transporte;
    const iva = base * IVA_ESPANA;
    const total = base + iva;
    return { subtotal, transporte, base, iva, total };
  }

  function responderCotizacion(id, items) {
    const totales = calcularTotales(id, items);
    const notas = respuestaTemp[id]?.notas || "";
    if (totales.total <= 0) return;
    db.collection("cotizaciones").doc(id).update({
      estado: "respondida",
      precioTotal: Math.round(totales.total * 100) / 100,
      desglose: { ...totales, preciosItems: respuestaTemp[id]?.preciosItems || {} },
      notasAdmin: notas,
    }).catch((error) => console.error("Error respondiendo cotización:", error));
  }

  function quitarItemDeCotizacion(cotizacionId, itemId) {
    const actual = cotizaciones.find((c) => c.id === cotizacionId);
    if (!actual) return;
    const nuevosItems = (actual.items || []).filter((it) => it.id !== itemId);
    db.collection("cotizaciones").doc(cotizacionId).update({ items: nuevosItems })
      .catch((error) => console.error("Error quitando producto:", error));
  }

  function empezarCorreccion(c) {
    // Precarga el formulario con lo que ya se había cobrado, para corregir sobre eso
    setRespuestaTemp((prev) => ({
      ...prev,
      [c.id]: {
        preciosItems: c.desglose?.preciosItems || {},
        transporte: c.desglose?.transporte || "",
        notas: c.notasAdmin || "",
      },
    }));
    setEditando((prev) => ({ ...prev, [c.id]: true }));
    setVista("pedidos");
    setFiltroEstado(c.estado === "entregado" ? "entregadas" : "despacho");
    setFiltroCliente("todos");
  }

  function guardarCorreccion(id, items) {
    const totales = calcularTotales(id, items);
    const notas = respuestaTemp[id]?.notas || "";
    db.collection("cotizaciones").doc(id).update({
      precioTotal: Math.round(totales.total * 100) / 100,
      desglose: { ...totales, preciosItems: respuestaTemp[id]?.preciosItems || {} },
      notasAdmin: notas,
    })
      .then(() => setEditando((prev) => ({ ...prev, [id]: false })))
      .catch((error) => console.error("Error guardando corrección:", error));
  }

  function intentarEntrar() {
    if (claveIngresada === CONTRASENA_TEMPORAL) {
      setAutenticado(true);
      setErrorClave(false);
    } else {
      setErrorClave(true);
    }
  }

  if (!autenticado) {
    return (
      <div style={{ fontFamily: F_BODY, background: COLOR.fondo, minHeight: "100vh", display: "flex", alignItems: "center", justifyContent: "center" }}>
        <div style={{ background: COLOR.tarjeta, border: `1px solid ${COLOR.borde}`, borderRadius: 12, padding: 30, width: "min(320px, 90vw)" }}>
          <div style={{ fontFamily: F_DISPLAY, fontWeight: 900, fontSize: 20, marginBottom: 4, textAlign: "center" }}>
            DEPO<span style={{ color: COLOR.ladrillo }}>MARKET</span>
          </div>
          <div style={{ fontSize: 13, color: COLOR.grafitoSuave, textAlign: "center", marginBottom: 18 }}>Panel Interno</div>
          <input
            type="password"
            placeholder={`Contraseña (vista previa: ${CONTRASENA_TEMPORAL})`}
            value={claveIngresada}
            onChange={(e) => setClaveIngresada(e.target.value)}
            onKeyDown={(e) => e.key === "Enter" && intentarEntrar()}
            style={{ width: "100%", padding: "10px 12px", border: `1px solid ${COLOR.borde}`, borderRadius: 8, fontSize: 14, marginBottom: 10, boxSizing: "border-box" }}
          />
          {errorClave && <div style={{ color: COLOR.ladrillo, fontSize: 12.5, marginBottom: 10 }}>Contraseña incorrecta.</div>}
          <button
            onClick={intentarEntrar}
            style={{ width: "100%", padding: "10px 12px", background: COLOR.amarillo, border: "none", borderRadius: 8, fontWeight: 800, fontSize: 14, cursor: "pointer" }}
          >
            Entrar
          </button>
        </div>
      </div>
    );
  }

  const pendientes = cotizaciones.filter((c) => c.estado === "enviada").length;

  return (
    <div style={{ fontFamily: F_BODY, background: COLOR.fondo, minHeight: "100vh", color: COLOR.grafito, paddingBottom: 40 }}>
      <header style={{ background: COLOR.grafito, color: COLOR.fondo, padding: "18px 20px" }}>
        <div style={{ maxWidth: 900, margin: "0 auto", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
          <div style={{ fontFamily: F_DISPLAY, fontWeight: 900, fontSize: 22 }}>
            DEPO<span style={{ color: COLOR.amarillo }}>MARKET</span> · Panel Interno
          </div>
          {pendientes > 0 && (
            <div style={{ background: COLOR.amarillo, color: COLOR.grafito, borderRadius: 20, padding: "4px 12px", fontSize: 12.5, fontWeight: 800 }}>
              {pendientes} pendiente{pendientes !== 1 ? "s" : ""}
            </div>
          )}
        </div>
      </header>

      <div style={{ maxWidth: 900, margin: "0 auto", padding: "14px 16px 0", display: "flex", gap: 8 }}>
        <button
          onClick={() => setVista("pedidos")}
          style={{ flex: 1, padding: "10px", borderRadius: 8, border: `1px solid ${vista === "pedidos" ? COLOR.grafito : COLOR.borde}`, background: vista === "pedidos" ? COLOR.grafito : COLOR.tarjeta, color: vista === "pedidos" ? COLOR.fondo : COLOR.grafito, fontWeight: 800, fontSize: 13, cursor: "pointer" }}
        >
          📋 Pedidos
        </button>
        <button
          onClick={() => setVista("facturacion")}
          style={{ flex: 1, padding: "10px", borderRadius: 8, border: `1px solid ${vista === "facturacion" ? COLOR.grafito : COLOR.borde}`, background: vista === "facturacion" ? COLOR.grafito : COLOR.tarjeta, color: vista === "facturacion" ? COLOR.fondo : COLOR.grafito, fontWeight: 800, fontSize: 13, cursor: "pointer" }}
        >
          🧾 Facturación
        </button>
        <button
          onClick={() => setVista("clientes")}
          style={{ flex: 1, padding: "10px", borderRadius: 8, border: `1px solid ${vista === "clientes" ? COLOR.grafito : COLOR.borde}`, background: vista === "clientes" ? COLOR.grafito : COLOR.tarjeta, color: vista === "clientes" ? COLOR.fondo : COLOR.grafito, fontWeight: 800, fontSize: 13, cursor: "pointer" }}
        >
          👥 Clientes
        </button>
      </div>

      {vista === "facturacion" ? (
        <FacturacionControl cotizaciones={cotizaciones} onEditarCotizacion={empezarCorreccion} />
      ) : vista === "clientes" ? (
        <ClientesDirectorio cotizaciones={cotizaciones} />
      ) : (
      <div style={{ maxWidth: 900, margin: "0 auto", padding: "20px 16px" }}>
        <div style={{ display: "flex", gap: 8, marginBottom: 18 }}>
          {[
            { id: "todos", label: "Todos", icono: "👥" },
            { id: "empresa", label: "Empresas", icono: "🏢" },
            { id: "comun", label: "Clientes comunes", icono: "🙋" },
          ].map((f) => {
            const cantidad =
              f.id === "todos"
                ? cotizaciones.length
                : cotizaciones.filter((c) => (c.contacto?.tipoCliente === "Empresa" ? "empresa" : "comun") === f.id).length;
            return (
              <button
                key={f.id}
                onClick={() => setFiltroCliente(f.id)}
                style={{
                  flex: 1,
                  padding: "10px 8px",
                  borderRadius: 8,
                  border: `1px solid ${filtroCliente === f.id ? COLOR.grafito : COLOR.borde}`,
                  background: filtroCliente === f.id ? COLOR.grafito : COLOR.tarjeta,
                  color: filtroCliente === f.id ? COLOR.fondo : COLOR.grafito,
                  fontWeight: 700,
                  fontSize: 12.5,
                  cursor: "pointer",
                }}
              >
                {f.icono} {f.label} ({cantidad})
              </button>
            );
          })}
        </div>

        <div style={{ display: "flex", gap: 8, marginBottom: 18, flexWrap: "wrap" }}>
          {[
            { id: "nuevas", label: "Nuevas — por responder", icono: "🆕" },
            { id: "despacho", label: "Pagadas — por despachar", icono: "📦" },
            { id: "entregadas", label: "Entregadas", icono: "✅" },
          ].map((f) => {
            const cantidad =
              f.id === "nuevas"
                ? cotizaciones.filter((c) => c.estado === "enviada").length
                : f.id === "despacho"
                ? cotizaciones.filter((c) => ["respondida", "aceptada", "pagada"].includes(c.estado)).length
                : cotizaciones.filter((c) => c.estado === "entregado").length;
            return (
              <button
                key={f.id}
                onClick={() => setFiltroEstado(f.id)}
                style={{
                  flex: "1 1 130px",
                  padding: "10px 8px",
                  borderRadius: 8,
                  border: `1px solid ${filtroEstado === f.id ? COLOR.grafito : COLOR.borde}`,
                  background: filtroEstado === f.id ? COLOR.grafito : COLOR.tarjeta,
                  color: filtroEstado === f.id ? COLOR.fondo : COLOR.grafito,
                  fontWeight: 700,
                  fontSize: 12.5,
                  cursor: "pointer",
                }}
              >
                {f.icono} {f.label} ({cantidad})
              </button>
            );
          })}
        </div>

        {(() => {
          const listaFiltrada = cotizaciones
            .filter((c) => filtroCliente === "todos" || (c.contacto?.tipoCliente === "Empresa" ? "empresa" : "comun") === filtroCliente)
            .filter((c) =>
              filtroEstado === "nuevas"
                ? c.estado === "enviada"
                : filtroEstado === "despacho"
                ? ["respondida", "aceptada", "pagada"].includes(c.estado)
                : c.estado === "entregado"
            );

          if (listaFiltrada.length === 0) {
            return (
              <div style={{ textAlign: "center", color: COLOR.grafitoSuave, fontSize: 13, padding: 30, border: `1px dashed ${COLOR.borde}`, borderRadius: 10 }}>
                No hay cotizaciones que combinen{" "}
                <strong>{filtroCliente === "todos" ? "cualquier cliente" : filtroCliente === "empresa" ? "Empresa" : "Particular"}</strong>
                {" "}+{" "}
                <strong>{filtroEstado === "nuevas" ? "Nuevas" : filtroEstado === "despacho" ? "Por despachar" : "Entregadas"}</strong>
                {" "}en este momento. Probá con otra combinación de pestañas arriba.
              </div>
            );
          }

          return listaFiltrada.map((c) => {
            const esEmpresa = c.contacto?.tipoCliente === "Empresa";
            return (
          <div
            key={c.id}
            style={{
              border: `1px solid ${COLOR.borde}`,
              borderLeft: `5px solid ${esEmpresa ? COLOR.grafito : COLOR.amarillo}`,
              borderRadius: 10,
              padding: 16,
              marginBottom: 14,
              background: COLOR.tarjeta,
            }}
          >
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 8 }}>
              {esEmpresa ? (
                <span style={{ fontSize: 12, fontWeight: 900, padding: "4px 10px", borderRadius: 6, background: COLOR.grafito, color: COLOR.fondo, letterSpacing: 0.3 }}>
                  🏢 EMPRESA
                </span>
              ) : (
                <span style={{ fontSize: 12, fontWeight: 900, padding: "4px 10px", borderRadius: 6, background: COLOR.amarillo, color: COLOR.grafito, letterSpacing: 0.3 }}>
                  🙋 PARTICULAR
                </span>
              )}
              <EstadoBadge estado={c.estado} />
            </div>
            <div style={{ fontSize: 11, color: COLOR.grafitoSuave, marginBottom: 6 }}>{new Date(c.fecha).toLocaleString()}</div>

            <div style={{ display: "flex", alignItems: "center", gap: 8, marginBottom: 2, flexWrap: "wrap" }}>
              <div style={{ fontSize: 14, fontWeight: 700 }}>
                {esEmpresa ? c.contacto?.empresaNombre || "Empresa" : c.contacto?.nombre || "Sin nombre"}
              </div>
              {c.clienteNumero && (
                <span style={{ fontSize: 10, fontWeight: 800, padding: "2px 7px", borderRadius: 10, background: "rgba(245,183,0,0.18)", color: COLOR.ladrillo }}>
                  {c.clienteNumero}
                </span>
              )}
            </div>
            {c.contacto?.tipoCliente === "Empresa" && c.contacto?.nombre && (
              <div style={{ fontSize: 12, color: COLOR.grafitoSuave, marginBottom: 2 }}>Contacto: {c.contacto.nombre}</div>
            )}

            <div style={{ fontSize: 12.5, color: COLOR.grafitoSuave, marginBottom: 2, lineHeight: 1.5 }}>
              {c.contacto?.telefono && <>📞 {c.contacto.telefono}{" "}</>}
              {c.contacto?.celular && <>📱 {c.contacto.celular}{" "}</>}
              {c.contacto?.email && <>✉️ {c.contacto.email}</>}
            </div>
            <div style={{ fontSize: 12.5, color: COLOR.grafitoSuave, marginBottom: 8 }}>
              📍 {c.contacto?.direccion || ""}{c.contacto?.direccion && c.contacto?.cp ? ", " : ""}{c.contacto?.cp || ""}
              {c.contacto?.medio && <> · Prefiere: {c.contacto.medio}</>}
            </div>

            <div style={{ fontSize: 13, marginBottom: 4 }}>
              <strong>{c.proyecto?.tipo || ""}</strong> — {c.proyecto?.zona || ""}
            </div>
            <div style={{ fontSize: 12.5, color: COLOR.grafitoSuave, marginBottom: 10 }}>
              Plazo: {c.plazo}
              {c.contacto?.diaRequerido && <> · Entrega deseada: {new Date(c.contacto.diaRequerido + "T00:00:00").toLocaleDateString()}</>}
            </div>

            {(c.reportes || []).length > 0 && (
              <div style={{ background: "rgba(181,69,27,0.08)", border: `1px solid ${COLOR.ladrillo}`, borderRadius: 8, padding: 10, marginBottom: 10 }}>
                <div style={{ fontSize: 11.5, fontWeight: 800, color: COLOR.ladrillo, marginBottom: 6 }}>
                  ⚠️ {c.reportes.length} mensaje{c.reportes.length !== 1 ? "s" : ""} del cliente — {c.clienteNumero || "sin número"}
                </div>
                {c.reportes.map((r, i) => (
                  <div key={i} style={{ fontSize: 12.5, marginBottom: 4 }}>
                    <span style={{ color: COLOR.grafitoSuave, fontSize: 10.5 }}>{new Date(r.fecha).toLocaleString()}</span>
                    <div>{r.texto}</div>
                  </div>
                ))}
              </div>
            )}

            <div style={{ borderTop: `1px solid ${COLOR.borde}`, paddingTop: 10, marginBottom: 10 }}>
              {(c.items || []).map((it) => {
                const mostrarFormulario = c.estado === "enviada" || editando[c.id];
                return (
                  <div key={it.id} style={{ display: "flex", alignItems: "center", justifyContent: "space-between", gap: 8, padding: "5px 0" }}>
                    <div style={{ fontSize: 13, flex: 1, minWidth: 0 }}>
                      {it.marca} — {it.nombre} <span style={{ color: COLOR.grafitoSuave }}>×{it.cantidad}</span>
                    </div>
                    {mostrarFormulario && (
                      <>
                        <input
                          type="number"
                          placeholder="€/ud"
                          value={respuestaTemp[c.id]?.preciosItems?.[it.id] || ""}
                          onChange={(e) =>
                            setRespuestaTemp((prev) => ({
                              ...prev,
                              [c.id]: { ...prev[c.id], preciosItems: { ...prev[c.id]?.preciosItems, [it.id]: e.target.value } },
                            }))
                          }
                          style={{ width: 70, padding: "6px 8px", border: `1px solid ${COLOR.borde}`, borderRadius: 6, fontSize: 12.5, flexShrink: 0 }}
                        />
                        <button
                          onClick={() => quitarItemDeCotizacion(c.id, it.id)}
                          title="Sacar este producto (el cliente no lo quiere)"
                          style={{ background: "none", border: "none", color: COLOR.ladrillo, fontSize: 15, cursor: "pointer", flexShrink: 0, padding: "2px 4px" }}
                        >
                          🗑
                        </button>
                      </>
                    )}
                  </div>
                );
              })}
              {(c.items || []).length === 0 && (
                <div style={{ fontSize: 12, color: COLOR.grafitoSuave, fontStyle: "italic" }}>Sin productos de catálogo (solo fuera de catálogo, si hay).</div>
              )}
              {(c.extras || []).map((e, i) => (
                <div key={i} style={{ fontSize: 13, fontStyle: "italic", padding: "3px 0" }}>
                  {e} (fuera de catálogo — cotizar aparte)
                </div>
              ))}
            </div>

            {c.estado === "enviada" || editando[c.id] ? (
              <div>
                <div style={{ display: "flex", gap: 8, alignItems: "center", marginBottom: 8 }}>
                  <label style={{ fontSize: 12.5, color: COLOR.grafitoSuave }}>Transporte:</label>
                  <input
                    type="number"
                    placeholder="0"
                    value={respuestaTemp[c.id]?.transporte || ""}
                    onChange={(e) => setRespuestaTemp((prev) => ({ ...prev, [c.id]: { ...prev[c.id], transporte: e.target.value } }))}
                    style={{ width: 90, padding: "7px 9px", border: `1px solid ${COLOR.borde}`, borderRadius: 6, fontSize: 12.5 }}
                  />
                  <span style={{ fontSize: 12, color: COLOR.grafitoSuave }}>€</span>
                </div>

                {(() => {
                  const t = calcularTotales(c.id, c.items);
                  return t.subtotal > 0 || t.transporte > 0 ? (
                    <div style={{ background: COLOR.fondo, border: `1px solid ${COLOR.borde}`, borderRadius: 8, padding: "10px 12px", marginBottom: 8, fontSize: 12.5 }}>
                      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 2 }}>
                        <span>Subtotal materiales</span><span>{t.subtotal.toFixed(2)} €</span>
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 2 }}>
                        <span>Transporte</span><span>{t.transporte.toFixed(2)} €</span>
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 2, color: COLOR.grafitoSuave }}>
                        <span>IVA (21%)</span><span>{t.iva.toFixed(2)} €</span>
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", fontWeight: 800, fontSize: 14, borderTop: `1px solid ${COLOR.borde}`, marginTop: 6, paddingTop: 6 }}>
                        <span>Total</span><span>{t.total.toFixed(2)} €</span>
                      </div>
                    </div>
                  ) : null;
                })()}

                <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                  <input
                    placeholder="Nota (opcional)"
                    value={respuestaTemp[c.id]?.notas || ""}
                    onChange={(e) => setRespuestaTemp((prev) => ({ ...prev, [c.id]: { ...prev[c.id], notas: e.target.value } }))}
                    style={{ flex: 1, minWidth: 150, padding: "9px 10px", border: `1px solid ${COLOR.borde}`, borderRadius: 8, fontSize: 13.5 }}
                  />
                  {editando[c.id] ? (
                    <>
                      <button
                        onClick={() => setEditando((prev) => ({ ...prev, [c.id]: false }))}
                        style={{ padding: "9px 14px", background: "transparent", border: `1px solid ${COLOR.borde}`, borderRadius: 8, fontWeight: 700, fontSize: 13.5, cursor: "pointer" }}
                      >
                        Cancelar
                      </button>
                      <button
                        onClick={() => guardarCorreccion(c.id, c.items)}
                        style={{ padding: "9px 16px", background: "#2E7D32", color: "#fff", border: "none", borderRadius: 8, fontWeight: 800, fontSize: 13.5, cursor: "pointer" }}
                      >
                        Guardar corrección
                      </button>
                    </>
                  ) : (
                    <button
                      onClick={() => responderCotizacion(c.id, c.items)}
                      style={{ padding: "9px 16px", background: COLOR.amarillo, border: "none", borderRadius: 8, fontWeight: 800, fontSize: 13.5, cursor: "pointer" }}
                    >
                      Enviar cotización
                    </button>
                  )}
                </div>
              </div>
            ) : (
              <div>
                {c.numeroFactura && (
                  <div style={{ display: "inline-block", fontSize: 11.5, fontWeight: 800, padding: "3px 9px", borderRadius: 6, background: "rgba(46,125,50,0.12)", color: "#2E7D32", marginBottom: 6 }}>
                    🧾 Factura Nº {c.numeroFactura}
                  </div>
                )}
                {c.desglose && (
                  <div style={{ fontSize: 12, color: COLOR.grafitoSuave, marginBottom: 4 }}>
                    Subtotal {c.desglose.subtotal.toFixed(2)}€ · Transporte {c.desglose.transporte.toFixed(2)}€ · IVA (21%) {c.desglose.iva.toFixed(2)}€
                  </div>
                )}
                <div style={{ fontSize: 15, fontWeight: 800, color: COLOR.ladrillo, marginBottom: 10 }}>
                  {c.precioTotal} € <span style={{ fontSize: 11, fontWeight: 400, color: COLOR.grafitoSuave }}>(IVA incluido)</span>{" "}
                  {c.notasAdmin && <span style={{ fontWeight: 400, fontSize: 12.5, color: COLOR.grafitoSuave }}>· {c.notasAdmin}</span>}
                </div>
                <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                  <button
                    onClick={() => empezarCorreccion(c)}
                    style={{ padding: "8px 14px", background: "transparent", border: `1px solid ${COLOR.grafito}`, borderRadius: 8, fontWeight: 700, fontSize: 12.5, cursor: "pointer" }}
                  >
                    ✏️ Corregir cotización
                  </button>
                  {(c.estado === "respondida" || c.estado === "aceptada") && (
                    <button
                      onClick={() => {
                        const numeroFactura = siguienteNumeroFactura();
                        db.collection("cotizaciones").doc(c.id).update({
                          estado: "pagada",
                          numeroFactura,
                          fechaFactura: new Date().toISOString(),
                        }).catch((error) => console.error("Error marcando como pagada:", error));
                      }}
                      style={{ padding: "8px 14px", background: "#2E7D32", color: "#fff", border: "none", borderRadius: 8, fontWeight: 800, fontSize: 12.5, cursor: "pointer" }}
                    >
                      💳 Marcar como pagada (genera factura)
                    </button>
                  )}
                </div>
                {c.estado === "pagada" && (
                  <button
                    onClick={() =>
                      db.collection("cotizaciones").doc(c.id).update({ estado: "entregado" })
                        .catch((error) => console.error("Error marcando como entregado:", error))
                    }
                    style={{ padding: "8px 14px", background: "#1B5E20", color: "#fff", border: "none", borderRadius: 8, fontWeight: 800, fontSize: 12.5, cursor: "pointer" }}
                  >
                    ✅ Marcar como entregado
                  </button>
                )}
              </div>
            )}
          </div>
            );
          });
        })()}
      </div>
      )}
    </div>
  );
}

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<PanelInterno />);
</script>
</body>
</html>
