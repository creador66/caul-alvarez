<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Sistema Modular Veta </title>
  <style>
    * { box-sizing: border-box; margin:0; padding:0 }
    body { font-family: Arial, sans-serif; padding:20px; background:#f5f5f5 }
    h1 { margin-bottom:20px }
    .tabs { display:flex; border-bottom:2px solid #ddd; margin-bottom:20px }
    .tabs button {
      background:none; border:none; padding:10px 20px; cursor:pointer;
      font-size:16px; border-bottom:3px solid transparent;
      transition:border-color .2s, color .2s;
    }
    .tabs button.active { border-color:#3f51b5; color:#3f51b5 }
    .section { display:none }
    .section.active { display:block }
    .toolbar { display:flex; justify-content:space-between; margin-bottom:10px }
    .toolbar button {
      padding:6px 12px; font-size:14px;
      background:#3f51b5; color:#fff; border:none; border-radius:4px;
    }
    .toolbar button:hover { background:#354496 }
    table {
      width:100%; border-collapse:collapse; margin-top:10px;
      background:#fff; display:block; max-height:300px; overflow-y:auto;
    }
    th, td { border:1px solid #ddd; padding:8px; text-align:left }
    th { background:#fafafa }
    .small-btn { font-size:0.9em; padding:2px 6px; margin-left:8px }
    .overlay {
      position:fixed; top:0; left:0; width:100%; height:100%;
      background:rgba(0,0,0,0.5); display:none;
      align-items:center; justify-content:center;
    }
    .overlay.active { display:flex }
    .modal {
      background:#fff; border-radius:4px; overflow:hidden;
      width:90%; max-width:500px; display:flex; flex-direction:column;
    }
    .modal header, .modal footer { background:#f1f1f1; padding:12px }
    .modal header { display:flex; justify-content:space-between; align-items:center }
    .modal .body {
      padding:16px; max-height:70vh; overflow-y:auto;
    }
    .modal input, .modal select { 
      width:100%; padding:6px; margin-bottom:12px;
      border:1px solid #ccc; border-radius:4px; font-size:16px;
    }
    .ingred-row { display:flex; gap:8px; align-items:center; margin-bottom:8px; }
    .ingred-row select, .ingred-row input { flex:1 }
    .ingred-row .costo { min-width:60px; text-align:right }
    #totalCosto { font-weight:bold; margin-top:8px }
    /* Compras */
    #ComprasSection .compras-container {
      display: flex;
      gap: 20px;
    }
    #ComprasSection .compras-container > div {
      width: 50%;
      background: #fff;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
      max-height: 300px;
      overflow-y: auto;
    }
    #ComprasSection h4 {
      margin-bottom: 8px;
      font-size: 1em;
      border-bottom: 1px solid #ccc;
      padding-bottom: 4px;
    }
    .alerta-nivel1 { color: #f39c12; }
    .alerta-nivel2 { color: #e67e22; }
    .alerta-nivel3 { color: #c0392b; }
    .historial-entry { display:flex; justify-content:space-between; align-items:center; margin-bottom:6px; }
    .historial-entry span { flex:1; }
  </style>
</head>
<body>

  <h1>Sistema Modular Veta IAEgo</h1>
  <div class="tabs">
    <button class="active" data-target="productosSection">Productos</button>
    <button data-target="recetasSection">Recetas</button>
    <button data-target="cajaSection">Caja</button>
    <button data-target="ComprasSection">Compras</button>
  </div>

  <!-- Productos -->
  <div id="productosSection" class="section active">
    <div class="toolbar">
      <button onclick="openModal('producto')">Agregar Producto</button>
    </div>
    <table id="productosTabla">
      <thead>
        <tr>
          <th>Producto</th><th>Precio</th><th>Cantidad comprada</th><th>Unidad</th>
          <th>Stock total</th><th>Ventas</th><th>Stock real</th>
          <th>Precio unit.</th><th>Valor stock</th><th>Acciones</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <!-- Recetas -->
  <div id="recetasSection" class="section">
    <div class="toolbar">
      <button onclick="openModal('receta')">Agregar Receta</button>
    </div>
    <table id="recetasTabla">
      <thead>
        <tr>
          <th>Receta</th><th>Ingredientes</th><th>Costo total</th><th>Acciones</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <!-- Caja -->
  <div id="cajaSection" class="section">
    <div class="toolbar">
      <button onclick="openModal('venta')">Registrar Venta</button>
    </div>
    <table id="ventasTabla">
      <thead>
        <tr>
          <th>Fecha</th><th>Receta</th><th>Cantidad</th><th>Total Venta</th><th>Acciones</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <!-- Compras -->
  <div id="ComprasSection" class="section">
    <div class="compras-container">
      <div id="alertas">
        <h4>Alertas de Stock</h4>
      </div>
      <div id="listaCompras">
        <h4>Lista de Compras</h4>
      </div>
    </div>
    <div style="margin-top:20px;">
      <h4>Historial de Surtidos</h4>
      <div id="historialSurtidos"></div>
    </div>
  </div>

  <!-- Modal genérico -->
  <div id="modalOverlay" class="overlay">
    <div class="modal">
      <header>
        <h3 id="modalTitle"></h3>
        <button onclick="closeModal()">✖</button>
      </header>
      <div class="body" id="modalBody"></div>
      <footer>
        <button id="modalSaveBtn">Guardar</button>
        <button onclick="closeModal()">Cancelar</button>
      </footer>
    </div>
  </div>

  <script>
    // IDs y storage keys
    const makeId = () => Date.now().toString(36) + Math.random().toString(36).slice(2,5);
    const PK_PROD     = 'miAlmacenUnidades';
    const PK_RECETAS  = 'miAlmacenRecetas';
    const PK_VENTAS   = 'miAlmacenVentas';
    const PK_SURTIDOS = 'miAlmacenSurtidos';

    // Datos iniciales
    let products = JSON.parse(localStorage.getItem(PK_PROD)    || '[]');
    let recipes  = JSON.parse(localStorage.getItem(PK_RECETAS) || '[]');
    let ventas   = JSON.parse(localStorage.getItem(PK_VENTAS)  || '[]');
    let surtidos = JSON.parse(localStorage.getItem(PK_SURTIDOS)|| '[]');

    // Asegurar surtidoAcumulado
    products = products.map(p => ({ ...p, surtidoAcumulado: p.surtidoAcumulado||0 }));

    function saveAllProducts() {
      localStorage.setItem(PK_PROD, JSON.stringify(products));
    }
    function saveSurtidos() {
      localStorage.setItem(PK_SURTIDOS, JSON.stringify(surtidos));
    }
    function calcStockReal(p) {
  return p.stockTotal - p.ventas + p.surtidoAcumulado;
}


    // — CRUD Productos —
    function saveProducto(editId) {
      const n  = document.getElementById('nombre').value.trim();
      const pr = parseFloat(document.getElementById('precio').value);
      const px = parseFloat(document.getElementById('porx').value);
      const ex = parseFloat(document.getElementById('stock').value);
      const un = document.getElementById('unidad').value;
      if (!n || isNaN(pr)||isNaN(px)||isNaN(ex)) return;

      let ventasPrev = 0, surtPrev = 0;
      if (editId) {
        const old = products.find(p=>p.id===editId);
        ventasPrev = old?old.ventas:0;
        surtPrev   = old?old.surtidoAcumulado:0;
      }

      const item = {
        id: editId||makeId(),
        nombre: n,
        precio: pr,
        porx: px,
        unidad: un,
        stockTotal: ex + ventasPrev - surtPrev,
        ventas: ventasPrev,
        surtidoAcumulado: surtPrev
      };

      products = editId
        ? products.map(p=>p.id===editId?item:p)
        : [...products,item];
      saveAllProducts();
      renderTable();
      closeModal();
    }

    function deleteProduct(id) {
      products = products.filter(p=>p.id!==id);
      saveAllProducts();
      renderTable();
    }

    // — CRUD Recetas —
    function saveReceta(editId) {
      const name = document.getElementById('recetaNombre').value.trim();
      if (!name) return;
      const rows = Array.from(document.querySelectorAll('.ingred-row'));
      const ingred = rows.map(r=>{
        const pid = r.querySelector('select').value;
        const qty = parseFloat(r.querySelector('input').value)||0;
        const ud  = r.querySelector('.ingredUnidad').value;
        return pid?{ productoId: pid, cantidad: qty, unidad: ud }:null;
      }).filter(x=>x);
      const total = rows.reduce((sum,r)=>
        sum + parseFloat(r.querySelector('.costo').textContent.replace('$','')),0);
      const obj = { id:editId||makeId(), nombre: name, ingredientes: ingred, costoTotal: total };
      recipes = editId
        ? recipes.map(r=>r.id===editId?obj:r)
        : [obj,...recipes];
      localStorage.setItem(PK_RECETAS, JSON.stringify(recipes));
      renderRecipes();
      closeModal();
    }

    function deleteRecipe(id) {
      recipes = recipes.filter(r=>r.id!==id);
      localStorage.setItem(PK_RECETAS, JSON.stringify(recipes));
      renderRecipes();
    }

    // — CRUD Ventas —
    function saveVenta(editId=null) {
      const sel = document.getElementById('ventaReceta');
      const qty = parseFloat(document.getElementById('ventaCantidad').value)||0;
      if (!sel.value||qty<=0) return;
      const rec = recipes.find(r=>r.id===sel.value);
      if (!rec) return;

      if (editId) {
        const antigua = ventas.find(v=>v.id===editId);
        if (antigua) {
          const recA = recipes.find(r=>r.nombre===antigua.recetaNombre);
          recA.ingredientes.forEach(i=>{
            const p = products.find(x=>x.id===i.productoId);
            if (!p) return;
            const key = `${p.unidad}->${i.unidad}`;
            const f   = {'kg->g':1000,'g->kg':1/1000,'l->ml':1000,'ml->l':1/1000,'pz->pz':1}[key]||1;
            p.ventas = Math.max(p.ventas - i.cantidad*antigua.cantidad*(1/f),0);
          });
        }
      }

      rec.ingredientes.forEach(i=>{
        const p = products.find(x=>x.id===i.productoId);
        if (!p) return;
        const key = `${p.unidad}->${i.unidad}`;
        const f   = {'kg->g':1000,'g->kg':1/1000,'l->ml':1000,'ml->l':1/1000,'pz->pz':1}[key]||1;
        p.ventas += i.cantidad*qty*(1/f);
      });
      saveAllProducts();
renderTable()
      const total = rec.costoTotal * qty;
      const ventaObj = {
        id: editId||makeId(),
        fecha: Date.now(),
        recetaNombre: rec.nombre,
        cantidad: qty,
        total: total
      };
      if (editId) {
        const idx = ventas.findIndex(v=>v.id===editId);
        if (idx!==-1) ventas[idx]=ventaObj;
      } else ventas.unshift(ventaObj);
      localStorage.setItem(PK_VENTAS, JSON.stringify(ventas));
      renderVentas();
      closeModal();
    }

    function deleteVenta(id) {
      const idx = ventas.findIndex(v=>v.id===id);
      if (idx===-1) return;
      const venta = ventas[idx];
      const rec   = recipes.find(r=>r.nombre===venta.recetaNombre);
      rec.ingredientes.forEach(i=>{
        const p = products.find(x=>x.id===i.productoId);
        if (!p) return;
        const key = `${p.unidad}->${i.unidad}`;
        const f   = {'kg->g':1000,'g->kg':1/1000,'l->ml':1000,'ml->l':1/1000,'pz->pz':1}[key]||1;
        p.ventas = Math.max(p.ventas - i.cantidad*venta.cantidad*(1/f),0);
      });
      saveAllProducts();
      ventas.splice(idx,1);
      localStorage.setItem(PK_VENTAS, JSON.stringify(ventas));
      renderTable();
      renderVentas();
    }

    // — Renderizar tablas —
    function renderTable() {
      const tbody = document.querySelector('#productosTabla tbody');
      tbody.innerHTML = products.map(p => {
        const stockReal      = calcStockReal(p);
        const precioUnitario = p.porx>0?(p.precio/p.porx).toFixed(2):'0.00';
        const valorStock     = (precioUnitario*stockReal).toFixed(2);
        return `
          <tr>
            <td>${p.nombre}</td>
            <td>$${p.precio.toFixed(2)}</td>
            <td>${p.porx}</td>
            <td>${p.unidad}</td>
            <td>${p.stockTotal} ${p.unidad}</td>
            <td>${p.ventas}</td>
            <td>${stockReal} ${p.unidad}</td>
            <td>$${precioUnitario}</td>
            <td>$${valorStock}</td>
            <td>
              <button class="small-btn" onclick="openModal('producto','${p.id}')">Editar</button>
              <button class="small-btn" onclick="deleteProduct('${p.id}')">Eliminar</button>
            </td>
          </tr>`;
      }).join('');
    }

    function renderRecipes() {
      const tbody = document.querySelector('#recetasTabla tbody');
      tbody.innerHTML = recipes.map(r => {
        const ingList = r.ingredientes.map(i => {
          const prod = products.find(p=>p.id===i.productoId);
          return prod?`${prod.nombre}(${i.cantidad}${i.unidad})`:'';
        }).join(', ');
        return `
          <tr>
            <td>${r.nombre}</td>
            <td>${ingList}</td>
            <td>$${r.costoTotal.toFixed(2)}</td>
            <td>
              <button class="small-btn" onclick="openModal('receta','${r.id}')">Editar</button>
              <button class="small-btn" onclick="deleteRecipe('${r.id}')">Eliminar</button>
            </td>
          </tr>`;
      }).join('');
    }

    function renderVentas() {
      const tbody = document.querySelector('#ventasTabla tbody');
      tbody.innerHTML = ventas.map(v => `
        <tr>
          <td>${new Date(v.fecha).toLocaleString()}</td>
          <td>${v.recetaNombre}</td>
          <td>${v.cantidad}</td>
          <td>$${v.total.toFixed(2)}</td>
          <td>
            <button class="small-btn" onclick="openModal('venta','${v.id}')">Editar</button>
            <button class="small-btn" onclick="deleteVenta('${v.id}')">Eliminar</button>
          </td>
        </tr>
      `).join('');
    }

    // — Compras y surtidos —
    function renderCompras() {
      const alertasDiv = document.getElementById('alertas');
      const listaDiv   = document.getElementById('listaCompras');
      alertasDiv.innerHTML = '<h4>Alertas de Stock</h4>';
      listaDiv.innerHTML   = '<h4>Lista de Compras</h4>';

      const compras = [];
      products.forEach(p => {
        const stockReal = calcStockReal(p);
        const pct       = stockReal / p.stockTotal;
        let nivel = 0;
        if (pct <= 0.40) nivel = 3;
        else if (pct <= 0.50) nivel = 2;
        else if (pct <= 0.70) nivel = 1;

        if (nivel > 0) {
          const div = document.createElement('div');
          div.innerHTML = `
            <strong>${p.nombre}</strong>: Quedan ${stockReal} ${p.unidad}
            <span class="alerta-nivel${nivel}">[Nivel ${nivel}]</span>
            <button class="small-btn" onclick="openSurtirModal('${p.id}')">Surtir</button>
          `;
          alertasDiv.append(div);
        }
        if (nivel === 3) {
          compras.push({ id:p.id, nombre: p.nombre, cantidad: p.stockTotal - calcStockReal(p), unidad: p.unidad });
        }
      });

      if (compras.length) {
        const ul = document.createElement('ul');
        compras.forEach(c => {
          const li = document.createElement('li');
          li.textContent = `${c.nombre}: pedir ${c.cantidad} ${c.unidad}`;
          ul.append(li);
        });
        listaDiv.append(ul);
      } else {
        const p = document.createElement('p');
        p.textContent = 'No hay productos en nivel crítico (≤ 40%).';
        listaDiv.append(p);
      }

      renderHistorialSurtidos();
    }

    function openSurtirModal(prodId) {
      const p = products.find(x=>x.id===prodId);
      if (!p) return;
      const faltante = p.stockTotal - calcStockReal(p);
      document.getElementById('modalTitle').innerText = `Surtir: ${p.nombre}`;
      document.getElementById('modalBody').innerHTML = `
        <p>Faltan <strong>${faltante} ${p.unidad}</strong> para llegar a stockTotal (${p.stockTotal}).</p>
        <label>Cantidad surtida:</label>
        <input type="number" id="surtirCantidad" placeholder="Ingresa cantidad comprada" />
      `;
      document.getElementById('modalSaveBtn').onclick = ()=>saveSurtir(prodId);
      document.getElementById('modalOverlay').classList.add('active');
    }

    function saveSurtir(prodId) {
      const inp = parseFloat(document.getElementById('surtirCantidad').value)||0;
      if (inp<=0) return alert('Ingresa una cantidad válida.');
      products = products.map(p=>{
        if (p.id===prodId) p.surtidoAcumulado += inp;
        return p;
      });
      saveAllProducts();
      surtidos.unshift({ id:makeId(), prodId, fecha:Date.now(), cantidad:inp });
      saveSurtidos();
      closeModal();
      renderTable(); renderVentas(); renderCompras();
    }

    function renderHistorialSurtidos() {
      const cont = document.getElementById('historialSurtidos');
      cont.innerHTML = '';
      if (!surtidos.length) {
        cont.innerHTML = '<p>No hay registros de surtidos.</p>';
        return;
      }
      surtidos.forEach(s => {
        const p = products.find(x=>x.id===s.prodId);
        const date = new Date(s.fecha).toLocaleString();
        const div = document.createElement('div');
        div.className = 'historial-entry';
        div.innerHTML = `
          <span>${date} — ${p? p.nombre : 'N/D'}: ${s.cantidad} ${p? p.unidad : ''}</span>
          <div>
            <button class="small-btn" onclick="openEditSurtidoModal('${s.id}')">Editar</button>
            <button class="small-btn" onclick="deleteSurtido('${s.id}')">Eliminar</button>
          </div>
        `;
        cont.append(div);
      });
    }

    function openEditSurtidoModal(surtId) {
      const s = surtidos.find(x=>x.id===surtId);
      if (!s) return;
      const p = products.find(x=>x.id===s.prodId);
      document.getElementById('modalTitle').innerText = `Editar surtido: ${p? p.nombre : ''}`;
      document.getElementById('modalBody').innerHTML = `
        <label>Cantidad surtida:</label>
        <input type="number" id="editSurtCantidad" value="${s.cantidad}" />
      `;
      document.getElementById('modalSaveBtn').onclick = ()=>saveEditSurtido(surtId);
      document.getElementById('modalOverlay').classList.add('active');
    }

    function saveEditSurtido(surtId) {
      const idx = surtidos.findIndex(x=>x.id===surtId);
      if (idx<0) return;
      const old = surtidos[idx];
      const nueva = parseFloat(document.getElementById('editSurtCantidad').value)||0;
      if (nueva<=0) return alert('Ingresa una cantidad válida.');
      products = products.map(p=>{
        if (p.id===old.prodId) {
          p.surtidoAcumulado = p.surtidoAcumulado - old.cantidad + nueva;
        }
        return p;
      });
      saveAllProducts();
      surtidos[idx].cantidad = nueva;
      saveSurtidos();
      closeModal();
      renderTable(); renderVentas(); renderCompras();
    }

    function deleteSurtido(surtId) {
      const idx = surtidos.findIndex(x=>x.id===surtId);
      if (idx<0) return;
      const s = surtidos[idx];
      products = products.map(p=>{
        if (p.id===s.prodId) p.surtidoAcumulado -= s.cantidad;
        return p;
      });
      saveAllProducts();
      surtidos.splice(idx,1);
      saveSurtidos();
      renderTable(); renderVentas(); renderCompras();
    }

    // — Modal genérico —
    const FORM_CONFIG = {
      producto: { title:'Producto', fields:[
        { id:'nombre', type:'text', placeholder:'Nombre del producto' },
        { id:'precio', type:'number', placeholder:'Precio total pagado' },
        { id:'porx',   type:'number', placeholder:'Cantidad comprada (solo costeo)' },
        { id:'stock',  type:'number', placeholder:'Stock existente (total real)' },
        { id:'unidad', type:'select', optionsKey:'unidadesBasicas' }
      ], save: saveProducto },
      receta:   { title:'Receta', fields:[
        { id:'recetaNombre', type:'text', placeholder:'Nombre de la receta' },
        { id:'ingredientes', type:'ingredientes' }
      ], save: saveReceta },
      venta:    { title:'Venta', fields:[
        { id:'ventaReceta',   type:'select', optionsKey:'recetas' },
        { id:'ventaCantidad', type:'number', placeholder:'Cantidad' }
      ], save: saveVenta }
    };
    const SELECT_OPTIONS = {
      unidadesBasicas: ['pz','g','kg','ml','l'],
      recetas: ()=>recipes.map(r=>({ value:r.id, text:`${r.nombre} ($${r.costoTotal.toFixed(2)})` }))
    };
    let currentForm = null;

    function openModal(type, editId=null) {
      currentForm = { type, editId };
      const cfg = FORM_CONFIG[type];
      document.getElementById('modalTitle').innerText =
        `${editId?'Editar':'Agregar'} ${cfg.title}`;
      const body = document.getElementById('modalBody');
      body.innerHTML = '';
      cfg.fields.forEach(f=>{
        if (f.type==='ingredientes') {
          const cont = document.createElement('div');
          cont.id = 'ingredCont';
          const addBtn = document.createElement('button');
          addBtn.innerText = '+ Ingrediente';
          addBtn.style.marginBottom = '12px';
          addBtn.onclick = ()=>addIngredRow();
          body.append(cont, addBtn);
          updateTotalCosto();
        } else {
          const el = f.type==='select' ? document.createElement('select') : document.createElement('input');
          el.id = f.id;
          if (f.type!=='select') el.type = f.type;
          if (f.placeholder) el.placeholder = f.placeholder;
          if (f.type==='select') {
            const opts = typeof SELECT_OPTIONS[f.optionsKey]==='function'
              ? SELECT_OPTIONS[f.optionsKey]() : SELECT_OPTIONS[f.optionsKey];
            el.innerHTML = opts.map(o=>
              typeof o==='string'
                ? `<option value="${o}">${o}</option>`
                : `<option value="${o.value}">${o.text}</option>`
            ).join('');
          }
          body.append(el);
        }
      });
      // Prefill edit
      if (type==='producto'&&editId) {
        const p = products.find(x=>x.id===editId);
        if (p) {
          document.getElementById('nombre').value = p.nombre;
          document.getElementById('precio').value = p.precio;
          document.getElementById('porx').value   = p.porx;
          document.getElementById('stock').value  = calcStockReal(p);
          document.getElementById('unidad').value = p.unidad;
        }
      }
      if (type==='receta'&&editId) {
        const r = recipes.find(x=>x.id===editId);
        if (r) {
          document.getElementById('recetaNombre').value = r.nombre;
          r.ingredientes.forEach(i=>addIngredRow(i.productoId,i.cantidad,i.unidad));
        }
      }
      if (type==='venta'&&editId) {
        const v = ventas.find(x=>x.id===editId);
        if (v) {
          document.getElementById('ventaCantidad').value = v.cantidad;
          const sel = document.getElementById('ventaReceta');
          const rec = recipes.find(r=>r.nombre===v.recetaNombre);
          if (rec) sel.value = rec.id;
        }
      }
      document.getElementById('modalSaveBtn').onclick = ()=>cfg.save(editId);
      document.getElementById('modalOverlay').classList.add('active');
    }

    function closeModal() {
      document.getElementById('modalOverlay').classList.remove('active');
      currentForm = null;
    }

    // — Ingredientes dinámicos —
    function addIngredRow(selId=null, qty=null, selUnidad=null) {
      const rowId = makeId();
      const div = document.createElement('div');
      div.className = 'ingred-row'; div.id = `ingr_${rowId}`;
      const selProd = document.createElement('select');
      selProd.innerHTML = '<option value="">-- Producto --</option>' +
        products.map(p=>`<option value="${p.id}" ${p.id===selId?'selected':''}>${p.nombre}</option>`).join('');
      selProd.onchange = ()=>{ populateUnidadOptions(div.id); updateRowCosto(div.id); };

      const selUn = document.createElement('select');
      selUn.className = 'ingredUnidad';
      selUn.onchange = ()=>updateRowCosto(div.id);

      const inp = document.createElement('input');
      inp.type = 'number'; inp.placeholder = 'Cantidad';
      if (qty) inp.value = qty;
      inp.oninput = ()=>updateRowCosto(div.id);

      const sp = document.createElement('div');
      sp.className = 'costo'; sp.textContent = '$0.00';

      const btn = document.createElement('button');
      btn.textContent = '✖';
      btn.onclick = ()=>{ div.remove(); updateTotalCosto(); };

      div.append(selProd, selUn, inp, sp, btn);
      document.getElementById('ingredCont').append(div);
      populateUnidadOptions(div.id, selUnidad);
      updateRowCosto(div.id);
    }

    function populateUnidadOptions(rowId, selectedUnidad=null) {
      const div = document.getElementById(rowId);
      const pid = div.querySelector('select').value;
      const selUn = div.querySelector('.ingredUnidad');
      selUn.innerHTML = '';
      if (!pid) return;
      const prod = products.find(p=>p.id===pid);
      const mapa = { 'kg':['kg','g'], 'g':['g','kg'], 'l':['l','ml'], 'ml':['ml','l'], 'pz':['pz'] };
      (mapa[prod.unidad]||[prod.unidad]).forEach(u=>{
        const o = document.createElement('option');
        o.value = u; o.textContent = u;
        if (u===selectedUnidad) o.selected = true;
        selUn.append(o);
      });
    }

    function updateRowCosto(rowId) {
      const div = document.getElementById(rowId);
      const pid = div.querySelector('select').value;
      const unidadSel = div.querySelector('.ingredUnidad').value;
      const cantidad = parseFloat(div.querySelector('input').value)||0;
      const prod = products.find(p=>p.id===pid);
      let costo = 0;
      if (prod) {
        const puBase = prod.precio/prod.porx;
        const key = `${prod.unidad}->${unidadSel}`;
        const factors = {'kg->g':1000,'g->kg':1/1000,'l->ml':1000,'ml->l':1/1000,'pz->pz':1};
        const f = factors[key]||1;
        const cantEnBase = cantidad*(1/f);
        costo = puBase*cantEnBase;
      }
      div.querySelector('.costo').textContent = `$${costo.toFixed(2)}`;
      updateTotalCosto();
    }

    function updateTotalCosto() {
      let total = 0;
      document.querySelectorAll('.ingred-row').forEach(div=>{
        total += parseFloat(div.querySelector('.costo').textContent.replace('$',''))||0;
      });
      const old = document.getElementById('totalCosto');
      if (old) old.remove();
      const txt = document.createElement('div');
      txt.id = 'totalCosto';
      txt.textContent = `Costo total: $${total.toFixed(2)}`;
      document.getElementById('ingredCont').append(txt);
    }

    // — Pestañas e inicialización —
    document.querySelectorAll('.tabs button').forEach(btn=>
      btn.addEventListener('click',()=>{
        document.querySelectorAll('.tabs button').forEach(b=>b.classList.remove('active'));
        btn.classList.add('active');
        const tgt = btn.dataset.target;
        document.querySelectorAll('.section').forEach(sec=>
          sec.id===tgt?sec.classList.add('active'):sec.classList.remove('active')
        );
      })
    );

    // Monkey‑patch para refrescar Compras tras tabla/ventas
    const origRT  = renderTable;
    renderTable  = ()=>{ origRT(); renderCompras(); };
    const origRV = renderVentas;
    renderVentas = ()=>{ origRV(); renderCompras(); };

    // Inicializar vistas
    renderTable();
    renderRecipes();
    renderVentas();
    renderCompras();
  </script>

</body>
</html>
