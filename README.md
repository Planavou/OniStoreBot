# OniStoreBot
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Магазин</title>
  <script src="https://telegram.org/js/telegram-web-app.js"></script>
  <style>
    :root {
      --bg: var(--tg-theme-bg-color, #ffffff);
      --text: var(--tg-theme-text-color, #000000);
      --hint: var(--tg-theme-hint-color, #999999);
      --button: var(--tg-theme-button-color, #2481cc);
      --button-text: var(--tg-theme-button-text-color, #ffffff);
      --secondary: var(--tg-theme-secondary-bg-color, #f0f0f0);
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
      background: var(--bg);
      color: var(--text);
      padding-bottom: 80px;
    }

    header {
      padding: 16px;
      background: var(--secondary);
      position: sticky;
      top: 0;
      z-index: 10;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    header h1 { font-size: 18px; }

    .tabs {
      display: flex;
      gap: 8px;
    }
    .tab {
      padding: 6px 14px;
      border-radius: 20px;
      background: transparent;
      border: 1px solid var(--hint);
      color: var(--text);
      font-size: 14px;
    }
    .tab.active {
      background: var(--button);
      color: var(--button-text);
      border-color: var(--button);
    }

    .products {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      padding: 16px;
    }

    .product {
      background: var(--secondary);
      border-radius: 16px;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    .product img {
      width: 100%;
      height: 140px;
      object-fit: cover;
      background: #ddd;
    }
    .product-info {
      padding: 10px;
      flex: 1;
      display: flex;
      flex-direction: column;
    }
    .product-name { font-weight: 600; font-size: 14px; margin-bottom: 4px; }
    .product-price { color: var(--button); font-weight: 700; margin-bottom: 8px; }
    .add-btn {
      margin-top: auto;
      background: var(--button);
      color: var(--button-text);
      border: none;
      padding: 8px;
      border-radius: 10px;
      font-size: 13px;
    }

    /* Корзина */
    .cart-item {
      display: flex;
      gap: 12px;
      padding: 12px 16px;
      border-bottom: 1px solid var(--secondary);
      align-items: center;
    }
    .cart-item img {
      width: 60px;
      height: 60px;
      object-fit: cover;
      border-radius: 10px;
    }
    .cart-info { flex: 1; }
    .qty {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .qty button {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      border: none;
      background: var(--button);
      color: white;
      font-size: 16px;
    }

    .total {
      position: fixed;
      bottom: 0;
      left: 0;
      right: 0;
      background: var(--secondary);
      padding: 16px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      box-shadow: 0 -4px 20px rgba(0,0,0,0.1);
    }
    .checkout-btn {
      background: var(--button);
      color: var(--button-text);
      border: none;
      padding: 12px 24px;
      border-radius: 12px;
      font-weight: 600;
    }

    /* Форма добавления товара */
    .form {
      padding: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }
    .form input, .form textarea {
      padding: 12px;
      border-radius: 12px;
      border: 1px solid var(--hint);
      background: var(--bg);
      color: var(--text);
      font-size: 15px;
    }
    .form button {
      background: var(--button);
      color: var(--button-text);
      border: none;
      padding: 14px;
      border-radius: 12px;
      font-size: 16px;
      font-weight: 600;
    }
    .preview


{
      width: 100%;
      max-height: 200px;
      object-fit: contain;
      border-radius: 12px;
      display: none;
    }

    .empty {
      text-align: center;
      padding: 60px 20px;
      color: var(--hint);
    }
  </style>
</head>
<body>
  <header>
    <h1 id="page-title">Витрина</h1>
    <div class="tabs">
      <button class="tab active" data-page="catalog">Товары</button>
      <button class="tab" data-page="cart">Корзина <span id="cart-count"></span></button>
      <button class="tab" data-page="add">+ Добавить</button>
    </div>
  </header>

  <!-- Витрина -->
  <div id="catalog" class="page">
    <div class="products" id="products-list"></div>
  </div>

  <!-- Корзина -->
  <div id="cart" class="page" style="display:none">
    <div id="cart-list"></div>
    <div class="total" id="cart-total" style="display:none">
      <div>
        <div style="font-size:13px;color:var(--hint)">Итого</div>
        <div style="font-size:20px;font-weight:700" id="total-price">0 ₽</div>
      </div>
      <button class="checkout-btn" onclick="checkout()">Оформить заказ</button>
    </div>
  </div>

  <!-- Добавление товара -->
  <div id="add" class="page" style="display:none">
    <div class="form">
      <input type="text" id="name" placeholder="Название товара" required>
      <input type="number" id="price" placeholder="Цена (₽)" required>
      <textarea id="description" placeholder="Описание (необязательно)" rows="3"></textarea>
      <input type="file" id="photo" accept="image/*" onchange="previewPhoto(event)">
      <img id="photo-preview" class="preview">
      <button onclick="addProduct()">Добавить товар</button>
    </div>
  </div>

  <script>
    const tg = window.Telegram.WebApp;
    tg.ready();
    tg.expand();

    // ========== ХРАНИЛИЩЕ ==========
    let products = JSON.parse(localStorage.getItem('products') || '[]');
    let cart = JSON.parse(localStorage.getItem('cart') || '[]');

    // Если товаров нет — добавим демо
    if (products.length === 0) {
      products = [
        {
          id: 1,
          name: "Кроссовки Nike",
          price: 8990,
          description: "Удобные беговые кроссовки",
          photo: "https://via.placeholder.com/300x300?text=Nike"
        },
        {
          id: 2,
          name: "Футболка",
          price: 1990,
          description: "Хлопок 100%",
          photo: "https://via.placeholder.com/300x300?text=T-Shirt"
        }
      ];
      saveProducts();
    }

    function saveProducts() {
      localStorage.setItem('products', JSON.stringify(products));
    }
    function saveCart() {
      localStorage.setItem('cart', JSON.stringify(cart));
      updateCartCount();
    }

    // ========== НАВИГАЦИЯ ==========
    document.querySelectorAll('.tab').forEach(btn => {
      btn.addEventListener('click', () => {
        document.querySelectorAll('.tab').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        
        const page = btn.dataset.page;
        document.querySelectorAll('.page').forEach(p => p.style.display = 'none');
        document.getElementById(page).style.display = 'block';
        
        document.getElementById('page-title').textContent = 
          page === 'catalog' ? 'Витрина' : 
          page === 'cart' ? 'Корзина' : 'Добавить товар';

        if (page === 'catalog') renderProducts();
        if (page === 'cart') renderCart();
      });
    });

    // ========== ВИТРИНА ==========
    function renderProducts() {
      const list = document.getElementById('products-list');
      if (products.length === 0) {
        list.innerHTML = '<div class="empty">Товаров пока нет.
Добавьте первый!</div>';
        return;
      }

      list.innerHTML = products.map(p => `
        <div class="product">
          <img src="\( {p.photo}" alt=" \){p.name}" onerror="this.src='https://via.placeholder.com/300'">
          <div class="product-info">
            <div class="product-name">${p.name}</div>
            <div class="p


roduct-price">${p.price.toLocaleString()} ₽</div>
            <button class="add-btn" onclick="addToCart(${p.id})">В корзину</button>
          </div>
        </div>
      `).join('');
    }

    // ========== КОРЗИНА ==========
    function addToCart(id) {
      const product = products.find(p => p.id === id);
      if (!product) return;

      const existing = cart.find(item => item.id === id);
      if (existing) {
        existing.qty += 1;
      } else {
        cart.push({ ...product, qty: 1 });
      }
      saveCart();
      tg.HapticFeedback.impactOccurred('light');
      tg.showPopup({ title: 'Добавлено', message: product.name + ' добавлен в корзину', buttons: [{type: 'ok'}] });
    }

    function changeQty(id, delta) {
      const item = cart.find(i => i.id === id);
      if (!item) return;
      item.qty += delta;
      if (item.qty <= 0) {
        cart = cart.filter(i => i.id !== id);
      }
      saveCart();
      renderCart();
    }

    function renderCart() {
      const list = document.getElementById('cart-list');
      const totalBlock = document.getElementById('cart-total');

      if (cart.length === 0) {
        list.innerHTML = '<div class="empty">Корзина пуста</div>';
        totalBlock.style.display = 'none';
        return;
      }

      list.innerHTML = cart.map(item => `
        <div class="cart-item">
          <img src="${item.photo}" alt="">
          <div class="cart-info">
            <div style="font-weight:600">${item.name}</div>
            <div style="color:var(--button)">${item.price.toLocaleString()} ₽</div>
          </div>
          <div class="qty">
            <button onclick="changeQty(${item.id}, -1)">−</button>
            <span>${item.qty}</span>
            <button onclick="changeQty(${item.id}, 1)">+</button>
          </div>
        </div>
      `).join('');

      const total = cart.reduce((sum, i) => sum + i.price * i.qty, 0);
      document.getElementById('total-price').textContent = total.toLocaleString() + ' ₽';
      totalBlock.style.display = 'flex';
    }

    function updateCartCount() {
      const count = cart.reduce((s, i) => s + i.qty, 0);
      document.getElementById('cart-count').textContent = count > 0 ? `(${count})` : '';
    }

    function checkout() {
      if (cart.length === 0) return;

      const order = {
        items: cart,
        total: cart.reduce((s, i) => s + i.price * i.qty, 0),
        user: tg.initDataUnsafe?.user || null,
        date: new Date().toISOString()
      };

      // Отправляем заказ боту (если открыто через Keyboard Button)
      // tg.sendData(JSON.stringify(order));

      // Или на свой бэкенд:
      /*
      fetch('https://your-api.com/api/order', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `tma ${tg.initData}`
        },
        body: JSON.stringify(order)
      });
      */

      tg.showPopup({
        title: 'Заказ оформлен!',
        message: `Сумма: ${order.total.toLocaleString()} ₽\nМы свяжемся с вами.`,
        buttons: [{type: 'ok'}]
      });

      cart = [];
      saveCart();
      renderCart();
    }

    // ========== ДОБАВЛЕНИЕ ТОВАРА ==========
    let photoBase64 = null;

    function previewPhoto(e) {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = (ev) => {
        photoBase64 = ev.target.result;
        const preview = document.getElementById('photo-preview');
        preview.src = photoBase64;
        preview.style.display = 'block';
      };
      reader.readAsDataURL(file);
    }

    function addProduct() {
      const name = document.getElementById('name').value.trim();
      const price = parseInt(document.getElementById('price').value);
      const description = document.getElementById('description').value.trim();

      if (!name || !price || !photoBase64) {
        tg.showAlert('Заполните название, цену и загрузите фото');
        return;
}
      const newProduct = {
        id: Date.now(),
        name,
        price,
        description,
        photo: photoBase64
      };

      products.unshift(newProduct);
      saveProducts();

      // Очистка формы
      document.getElementById('name').value = '';
      document.getElementById('price').value = '';
      document.getElementById('description').value = '';
      document.getElementById('photo').value = '';
      document.getElementById('photo-preview').style.display = 'none';
      photoBase64 = null;

      tg.showAlert('Товар добавлен!');
      document.querySelector('[data-page="catalog"]').click();
    }

    // Старт
    renderProducts();
    updateCartCount();
  </script>
</body>
</html>
