<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Escort — демо с оплатой и админкой</title>
<style>
  :root{
    --accent:#ff2d78;
    --accent-dark:#d02362;
    --bg:linear-gradient(180deg,#ffe6f0,#ffd6ec);
    --card-bg:white;
    --muted:#666;
  }
  *{box-sizing:border-box}
  body{
    margin:0;
    font-family:Inter, system-ui, Arial, sans-serif;
    background:var(--bg);
    color:#111;
    -webkit-font-smoothing:antialiased;
  }
  header{
    display:flex;
    justify-content:space-between;
    align-items:center;
    padding:18px 20px;
    background:linear-gradient(90deg,var(--accent),var(--accent-dark));
    color:#fff;
  }
  header h1{font-size:1.2rem;margin:0}
  header .controls button{
    background:rgba(255,255,255,0.15);
    border:1px solid rgba(255,255,255,0.12);
    color:#fff;
    padding:8px 12px;
    margin-left:8px;
    border-radius:10px;
    cursor:pointer;
  }
  main{padding:20px;max-width:1100px;margin:0 auto}
  .grid{
    display:grid;
    grid-template-columns:repeat(auto-fit,minmax(240px,1fr));
    gap:18px;
  }
  .card{
    background:var(--card-bg);
    border-radius:14px;
    padding:12px;
    box-shadow:0 8px 20px rgba(17,17,17,0.08);
    text-align:center;
  }
  .card img{width:100%;height:170px;object-fit:cover;border-radius:10px}
  .card h3{margin:10px 0 6px}
  .card p{margin:0;color:var(--muted);font-size:14px}
  .price{font-weight:700;margin-top:8px}
  .actions{margin-top:10px;display:flex;gap:8px;justify-content:center}
  .btn{
    padding:8px 12px;border-radius:10px;border:none;cursor:pointer;font-weight:600
  }
  .btn-primary{background:var(--accent);color:#fff}
  .btn-ghost{background:transparent;border:1px solid #ddd;color:#333}
  footer{padding:18px;text-align:center;color:#333;margin-top:28px}
  /* Modal */
  .modal-backdrop{position:fixed;inset:0;background:rgba(0,0,0,0.45);display:none;align-items:center;justify-content:center;z-index:60}
  .modal{background:white;border-radius:12px;padding:16px;min-width:320px;max-width:720px;max-height:90vh;overflow:auto}
  .modal h2{margin:0 0 8px}
  .form-row{display:flex;flex-direction:column;margin-bottom:10px}
  .form-row label{font-size:13px;margin-bottom:6px;color:#444}
  .form-row input,.form-row textarea{padding:8px;border-radius:8px;border:1px solid #ddd;font-size:14px}
  .small{font-size:13px;color:var(--muted)}
  .admin-btn{background:#222;color:#fff;padding:8px 10px;border-radius:8px;border:none;cursor:pointer}
  /* admin panel content */
  .admin-grid{display:grid;grid-template-columns:1fr 360px;gap:12px}
  .product-list{max-height:60vh;overflow:auto;padding-right:6px}
  .product-item{display:flex;gap:8px;align-items:center;padding:8px;border-bottom:1px dashed #eee}
  .product-item img{width:60px;height:60px;border-radius:8px;object-fit:cover}
  .muted{color:var(--muted)}
  .order-row{display:flex;justify-content:space-between;gap:8px;padding:8px;border-bottom:1px solid #f0f0f0}
  @media(max-width:900px){
    .admin-grid{grid-template-columns:1fr}
  }
</style>
</head>
<body>

<header>
  <h1>Escort (демо)</h1>
  <div class="controls">
    <button onclick="openCheckout(null)" class="btn btn-ghost">Корзина</button>
    <button onclick="openAdminLogin()" class="btn btn-ghost">Админ</button>
  </div>
</header>

<main>
  <section>
    <p class="small">Выбери услугу и нажми «Оплатить». В демо оплата имитируется — вводи ник, email и получишь чек. Все данные хранятся в браузере.</p>
  </section>

  <section class="grid" id="catalog"></section>

  <footer>
    &copy; 2025 Demo. Это шаблон — реальную платёжную интеграцию подключай через безопасный сервер (Stripe/PayPal и т.д.).
  </footer>
</main>

<!-- Checkout modal -->
<div id="modalCheckout" class="modal-backdrop" onclick="backdropClick(event,'modalCheckout')">
  <div class="modal" onclick="event.stopPropagation()">
    <h2 id="checkoutTitle">Оплата</h2>
    <div id="checkoutForm">
      <div class="form-row">
        <label>Услуга</label>
        <input id="checkout_product" readonly />
      </div>
      <div class="form-row">
        <label>Цена (в DEMO)</label>
        <input id="checkout_price" readonly />
      </div>
      <div class="form-row">
        <label>Ник (обязательно)</label>
        <input id="checkout_nick" placeholder="ваш ник в игре/телеграм" />
      </div>
      <div class="form-row">
        <label>Email / Контакт (рекомендуется)</label>
        <input id="checkout_contact" placeholder="email или тел. для подтверждения" />
      </div>
      <div class="form-row">
        <label>Комментарий</label>
        <textarea id="checkout_note" rows="3" placeholder="Доп. информация"></textarea>
      </div>
      <div style="display:flex;gap:8px;justify-content:flex-end">
        <button class="btn btn-ghost" onclick="closeModal('modalCheckout')">Отмена</button>
        <button class="btn btn-primary" onclick="submitOrder()">Оплатить (DEMO)</button>
      </div>
    </div>
    <div id="checkoutSuccess" style="display:none">
      <h3>Оплата успешно создана</h3>
      <p class="small">Это демо-чеки. Нажми, чтобы скопировать данные для отправки/оплаты.</p>
      <pre id="successData" style="background:#f8f8f8;padding:8px;border-radius:8px;overflow:auto"></pre>
      <div style="display:flex;gap:8px;justify-content:flex-end;margin-top:8px">
        <button class="btn btn-ghost" onclick="closeModal('modalCheckout')">Закрыть</button>
        <button class="btn btn-primary" onclick="copySuccess()">Скопировать чек</button>
      </div>
    </div>
  </div>
</div>

<!-- Admin modal -->
<div id="modalAdmin" class="modal-backdrop" onclick="backdropClick(event,'modalAdmin')">
  <div class="modal" onclick="event.stopPropagation()">
    <div id="adminLoginView">
      <h2>Вход в админку</h2>
      <div class="form-row">
        <label>Пароль</label>
        <input id="admin_password" type="password" placeholder="Введите пароль" />
      </div>
      <div style="display:flex;gap:8px;justify-content:flex-end">
        <button class="btn btn-ghost" onclick="closeModal('modalAdmin')">Отмена</button>
        <button class="btn admin-btn" onclick="adminLogin()">Войти</button>
      </div>
      <p class="small" style="margin-top:10px">Пароль для демо: <strong>159</strong></p>
    </div>

    <div id="adminPanelView" style="display:none">
      <h2>Панель администратора</h2>
      <div class="admin-grid">
        <div>
          <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px">
            <strong>Товары</strong>
            <div>
              <button class="btn btn-ghost" onclick="populateDemo()">Загрузить demo</button>
              <button class="btn admin-btn" onclick="openAddProduct()">Добавить</button>
            </div>
          </div>

          <div class="product-list" id="adminProductList"></div>
        </div>

        <div>
          <strong>Заказы</strong>
          <div id="ordersList" style="margin-top:8px;max-height:55vh;overflow:auto;border-radius:8px;padding:6px;background:#fff"></div>
        </div>
      </div>

      <div style="display:flex;justify-content:flex-end;margin-top:10px">
        <button class="btn btn-ghost" onclick="logoutAdmin()">Выйти</button>
        <button class="btn btn-primary" onclick="closeModal('modalAdmin')">Закрыть</button>
      </div>
    </div>

    <!-- Add/Edit product modal inside admin -->
    <div id="productEdit" style="display:none;margin-top:12px;background:#fafafa;padding:10px;border-radius:8px">
      <h3 id="productEditTitle">Добавить товар</h3>
      <div class="form-row">
        <label>Название</label>
        <input id="p_name" />
      </div>
      <div class="form-row">
        <label>Описание</label>
        <textarea id="p_desc" rows="3"></textarea>
      </div>
      <div class="form-row">
        <label>Цена (например 1000)</label>
        <input id="p_price" type="number" />
      </div>
      <div class="form-row">
        <label>Ссылка на изображение (скин Minecraft)</label>
        <input id="p_img" />
      </div>
      <div style="display:flex;gap:8px;justify-content:flex-end">
        <button class="btn btn-ghost" onclick="closeProductEdit()">Отмена</button>
        <button class="btn btn-primary" onclick="saveProduct()">Сохранить</button>
      </div>
    </div>

  </div>
</div>

<script>
/* ---------- Data layer (localStorage) ---------- */
const LS_PRODUCTS = 'demo_products_v1';
const LS_ORDERS = 'demo_orders_v1';
let products = [];
let orders = [];

/* ---------- Initial demo products (skins may change, replace URLs if needed) ---------- */
const demoProducts = [
  {
    id: genId(), name: 'Companion Alex', desc: 'Проводник и собеседник — 60 мин.', price: 1200,
    img: 'https://visage.surgeplay.com/face/512/Steve.png'
  },
  {
    id: genId(), name: 'Night Lily', desc: 'Элегантный вечер — 90 мин.', price: 2000,
    img: 'https://visage.surgeplay.com/face/512/Alex.png'
  },
  {
    id: genId(), name: 'Shadow', desc: 'Динамичный и надежный — 45 мин.', price: 800,
    img: 'https://visage.surgeplay.com/face/512/Notch.png'
  }
];

function genId(){ return 'id_' + Math.random().toString(36).slice(2,9) }

/* load/save */
function loadState(){
  const p = localStorage.getItem(LS_PRODUCTS);
  const o = localStorage.getItem(LS_ORDERS);
  products = p ? JSON.parse(p) : demoProducts.slice();
  orders = o ? JSON.parse(o) : [];
  renderCatalog();
}
function saveProducts(){ localStorage.setItem(LS_PRODUCTS, JSON.stringify(products)); renderCatalog(); renderAdminProducts(); }
function saveOrders(){ localStorage.setItem(LS_ORDERS, JSON.stringify(orders)); renderOrders(); }

/* ---------- Rendering catalog ---------- */
function renderCatalog(){
  const el = document.getElementById('catalog');
  el.innerHTML = '';
  products.forEach(p=>{
    const card = document.createElement('div'); card.className='card';
    card.innerHTML = `
      <img src="${escapeHtml(p.img)}" alt="${escapeHtml(p.name)}" onerror="this.src='https://via.placeholder.com/400x300?text=skin'"/>
      <h3>${escapeHtml(p.name)}</h3>
      <p>${escapeHtml(p.desc)}</p>
      <div class="price">${escapeHtml(String(p.price))} DEMO</div>
      <div class="actions">
        <button class="btn btn-ghost" onclick="openCheckout('${p.id}')">Подробнее</button>
        <button class="btn btn-primary" onclick="openCheckout('${p.id}')">Оплатить</button>
      </div>`;
    el.appendChild(card);
  });
}

/* ---------- Checkout flow (DEMO) ---------- */
function openCheckout(productId){
  const modal = document.getElementById('modalCheckout');
  document.getElementById('checkoutSuccess').style.display='none';
  document.getElementById('checkoutForm').style.display='block';
  if(productId){
    const p = products.find(x=>x.id===productId);
    document.getElementById('checkout_product').value = p ? p.name : '';
    document.getElementById('checkout_price').value = p ? p.price + ' DEMO' : '';
    modal.style.display='flex';
    modal.dataset.productId = productId;
  } else {
    // open empty (cart) - for demo, show message
    document.getElementById('checkout_product').value = 'Корзина (DEMO)';
    document.getElementById('checkout_price').value = '';
    modal.style.display='flex';
    modal.dataset.productId = '';
  }
}
function closeModal(id){ document.getElementById(id).style.display='none' }
function backdropClick(e,id){ if(e.target.id === id) closeModal(id) }

function submitOrder(){
  const pid = document.getElementById('modalCheckout').dataset.productId;
  const nick = document.getElementById('checkout_nick').value.trim();
  const contact = document.getElementById('checkout_contact').value.trim();
  const note = document.getElementById('checkout_note').value.trim();
  if(!nick){ alert('Укажи ник'); return; }
  let product = pid ? products.find(x=>x.id===pid) : {name: document.getElementById('checkout_product').value, price: 0};
  const order = {
    id: genId(),
    productId: pid || null,
    productName: product.name,
    price: product.price || 0,
    nick, contact, note,
    createdAt: new Date().toISOString(),
    status: 'pending' // демо
  };
  orders.unshift(order);
  saveOrders();

  // show simulated success + "чек"
  document.getElementById('checkoutForm').style.display='none';
  document.getElementById('checkoutSuccess').style.display='block';
  const data = {
    orderId: order.id, product: order.productName, price: order.price, nick: order.nick, contact: order.contact, note: order.note, createdAt: order.createdAt
  };
  document.getElementById('successData').textContent = JSON.stringify(data, null, 2);
  renderOrders();
}

/* copy success data */
function copySuccess(){ 
  const t = document.getElementById('successData').textContent;
  navigator.clipboard?.writeText(t).then(()=>{
    alert('Чек скопирован в буфер обмена');
  },()=>{ alert('Не удалось скопировать — скопируй вручную'); })
}

/* ---------- Admin login & panel ---------- */
function openAdminLogin(){ document.getElementById('modalAdmin').style.display='flex'; document.getElementById('admin_password').value=''; document.getElementById('admin_password').focus(); }
function adminLogin(){
  const pass = document.getElementById('admin_password').value;
  if(pass === '159'){
    document.getElementById('adminLoginView').style.display='none';
    document.getElementById('adminPanelView').style.display='block';
    document.getElementById('productEdit').style.display='none';
    renderAdminProducts();
    renderOrders();
  } else {
    alert('Неверный пароль');
  }
}
function logoutAdmin(){
  document.getElementById('adminLoginView').style.display='block';
  document.getElementById('adminPanelView').style.display='none';
  document.getElementById('productEdit').style.display='none';
  // don't remove data — just hide
}
function closeProductEdit(){ document.getElementById('productEdit').style.display='none'; }

/* Admin product management */
let editingProductId = null;
function renderAdminProducts(){
  const el = document.getElementById('adminProductList');
  el.innerHTML = '';
  if(products.length===0){ el.innerHTML = '<p class="small">Нет товаров. Нажмите "Загрузить demo" или "Добавить".</p>'; return; }
  products.forEach(p=>{
    const row = document.createElement('div'); row.className='product-item';
    row.innerHTML = `
      <img src="${escapeHtml(p.img)}" onerror="this.src='https://via.placeholder.com/150x150?text=skin'"/>
      <div style="flex:1">
        <div style="font-weight:700">${escapeHtml(p.name)}</div>
        <div class="small">${escapeHtml(p.desc)}</div>
        <div class="small muted">${escapeHtml(String(p.price))} DEMO</div>
      </div>
      <div style="display:flex;flex-direction:column;gap:6px">
        <button class="btn btn-ghost" onclick="startEditProduct('${p.id}')">Ред.</button>
        <button class="btn btn-ghost" onclick="deleteProduct('${p.id}')">Удал.</button>
      </div>`;
    el.appendChild(row);
  });
}
function openAddProduct(){
  editingProductId = null;
  document.getElementById('productEditTitle').textContent = 'Добавить товар';
  document.getElementById('p_name').value=''; document.getElementById('p_desc').value=''; document.getElementById('p_price').value='1000';
  document.getElementById('p_img').value='https://visage.surgeplay.com/face/512/Steve.png';
  document.getElementById('productEdit').style.display='block';
}
function startEditProduct(id){
  const p = products.find(x=>x.id===id); if(!p) return;
  editingProductId = id;
  document.getElementById('productEditTitle').textContent = 'Редактировать товар';
  document.getElementById('p_name').value=p.name; document.getElementById('p_desc').value=p.desc; document.getElementById('p_price').value=p.price; document.getElementById('p_img').value=p.img;
  document.getElementById('productEdit').style.display='block';
}
function saveProduct(){
  const name = document.getElementById('p_name').value.trim();
  const desc = document.getElementById('p_desc').value.trim();
  const price = parseInt(document.getElementById('p_price').value)||0;
  const img = document.getElementById('p_img').value.trim();
  if(!name){ alert('Введите название'); return; }
  if(editingProductId){
    const p = products.find(x=>x.id===editingProductId);
    p.name=name; p.desc=desc; p.price=price; p.img=img;
  } else {
    products.unshift({id:genId(), name, desc, price, img});
  }
  saveProducts();
  document.getElementById('productEdit').style.display='none';
}
function deleteProduct(id){
  if(!confirm('Удалить товар?')) return;
  products = products.filter(x=>x.id!==id);
  saveProducts();
}

/* ---------- Orders rendering ---------- */
function renderOrders(){
  const el = document.getElementById('ordersList');
  el.innerHTML = '';
  if(orders.length===0){ el.innerHTML='<p class="small">Заказов ещё нет</p>'; return; }
  orders.forEach(o=>{
    const div = document.createElement('div'); div.className='order-row';
    div.innerHTML = `
      <div style="flex:1">
        <div style="font-weight:700">${escapeHtml(o.productName)}</div>
        <div class="small muted">Ник: ${escapeHtml(o.nick)} • ${new Date(o.createdAt).toLocaleString()}</div>
        <div class="small">${escapeHtml(o.note||'')}</div>
      </div>
      <div style="min-width:120px;text-align:right">
        <div style="font-weight:700">${escapeHtml(String(o.price||0))} DEMO</div>
        <div style="margin-top:6px;display:flex;gap:6px;justify-content:flex-end">
          <button class="btn btn-ghost" onclick="copyOrder('${o.id}')">Коп.</button>
          <button class="btn btn-ghost" onclick="removeOrder('${o.id}')">Удал.</button>
        </div>
      </div>`;
    el.appendChild(div);
  });
}
function copyOrder(id){
  const o = orders.find(x=>x.id===id); if(!o) return;
  navigator.clipboard?.writeText(JSON.stringify(o,null,2)).then(()=>alert('Заказ скопирован'),()=>alert('Не удалось скопировать'));
}
function removeOrder(id){
  if(!confirm('Удалить заказ?')) return;
  orders = orders.filter(x=>x.id!==id); saveOrders();
}

/* ---------- Helpers ---------- */
function escapeHtml(s){ if(!s && s!==0) return ''; return String(s).replace(/[&<>"']/g, c=> ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c])); }

/* clicking open add/edit product via admin */
function openAddProduct(){ editingProductId=null; document.getElementById('productEditTitle').textContent='Добавить товар'; document.getElementById('p_name').value=''; document.getElementById('p_desc').value=''; document.getElementById('p_price').value=1000; document.getElementById('p_img').value='https://visage.surgeplay.com/face/512/Steve.png'; document.getElementById('productEdit').style.display='block'; }

/* populate demo */
function populateDemo(){ if(!confirm('Заменить товары демо-набором?')) return; products = demoProducts.map(x=>Object.assign({},x,{id:genId()})); saveProducts(); }

/* open edit/update created earlier conflict fix (function redeclare safe) */
window.openAddProduct = openAddProduct;

/* copy order helper already defined*/

/* small UI helpers */
function start(){ loadState(); }
function closeModal(id){ document.getElementById(id).style.display='none' }
function backdropClick(e,id){ if(e.target.id===id) closeModal(id) }

/* remove order earlier */
function removeOrder(id){
  if(!confirm('Удалить заказ?')) return;
  orders = orders.filter(x=>x.id!==id);
  saveOrders();
}

/* copy order defined earlier maybe overwritten, define safe */
function copyOrder(id){
  const o = orders.find(x=>x.id===id); if(!o) return;
  navigator.clipboard?.writeText(JSON.stringify(o,null,2)).then(()=>alert('Заказ скопирован'),()=>alert('Не удалось скопировать'));
}

/* start */
start();
</script>
</body>
</html>
