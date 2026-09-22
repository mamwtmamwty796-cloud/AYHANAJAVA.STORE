const products = [
  {
    id: 1,
    name: "مدیریت کلینیک دامپزشکی",
    icon: "🐔",
    price: 4900000,
    desc: "مشتری، حیوان، ویزیت، سوابق و نسخه"
  },
  {
    id: 2,
    name: "مدیریت انبار و فروشگاه",
    icon: "📦",
    price: 3900000,
    desc: "کالا، خرید، فروش، موجودی، سود و گزارش"
  },
  {
    id: 3,
    name: "مدیریت مرغداری گوشتی",
    icon: "🐣",
    price: 5900000,
    desc: "سالن، جوجه، دان، تلفات، وزن، FCR و هزینه"
  },
  {
    id: 4,
    name: "فروشگاه موبایل",
    icon: "📱",
    price: 4500000,
    desc: "برند، مدل، حافظه، رنگ، IMEI، خرید و فروش"
  },
  {
    id: 5,
    name: "داروخانه دامپزشکی",
    icon: "💊",
    price: 5200000,
    desc: "بانک دارو، موجودی، انقضا، فروش و هشدار"
  }
];

let cart = JSON.parse(localStorage.getItem("ayhanCart") || "[]");

const $ = id => document.getElementById(id);

function money(number) {
  return new Intl.NumberFormat("fa-IR").format(number) + " تومان";
}

function renderProducts(query = "") {
  const grid = $("grid");
  if (!grid) return;

  const q = query.trim().toLowerCase();

  const list = products.filter(product =>
    (product.name + " " + product.desc)
      .toLowerCase()
      .includes(q)
  );

  if (list.length === 0) {
    grid.innerHTML = "<p>پروژه‌ای پیدا نشد.</p>";
    return;
  }

  grid.innerHTML = list.map(product => `
    <article class="card">
      <div class="icon">${product.icon}</div>

      <h3>${product.name}</h3>

      <p>${product.desc}</p>

      <div class="price">
        ${money(product.price)}
      </div>

      <div class="actions">
        <button class="btn outline"
          onclick="showDetails(${product.id})">
          جزئیات
        </button>

        <button class="btn"
          onclick="addToCart(${product.id})">
          افزودن
        </button>
      </div>
    </article>
  `).join("");
}

function showDetails(id) {
  const product = products.find(p => p.id === id);

  if (!product) return;

  alert(
    product.name +
    "\n\n" +
    "امکانات پروژه:\n" +
    "• مدیریت اطلاعات\n" +
    "• جستجو\n" +
    "• گزارش‌ها\n" +
    "• ذخیره اطلاعات\n\n" +
    "قیمت: " +
    money(product.price)
  );
}

function saveCart() {
  localStorage.setItem(
    "ayhanCart",
    JSON.stringify(cart)
  );

  renderCart();
}

function addToCart(id) {
  const item = cart.find(x => x.id === id);

  if (item) {
    item.qty++;
  } else {
    cart.push({
      id: id,
      qty: 1
    });
  }

  saveCart();
  openCart();
}

function changeQty(id, amount) {
  const item = cart.find(x => x.id === id);

  if (!item) return;

  item.qty += amount;

  if (item.qty <= 0) {
    cart = cart.filter(x => x.id !== id);
  }

  saveCart();
}

function renderCart() {
  const countElement = $("count");
  const itemsElement = $("items");
  const totalElement = $("total");

  if (!itemsElement || !totalElement) return;

  const count = cart.reduce(
    (sum, item) => sum + item.qty,
    0
  );

  if (countElement) {
    countElement.textContent =
      count.toLocaleString("fa-IR");
  }

  if (cart.length === 0) {
    itemsElement.innerHTML =
      "<p>سبد خرید خالی است.</p>";

    totalElement.textContent = "۰ تومان";
    return;
  }

  let total = 0;

  itemsElement.innerHTML = cart.map(item => {
    const product =
      products.find(p => p.id === item.id);

    if (!product) return "";

    total += product.price * item.qty;

    return `
      <div class="row">

        <span>
          ${product.icon} ${product.name}
          <br>
          <small>${money(product.price)}</small>
        </span>

        <span class="qty">

          <button
            onclick="changeQty(${product.id}, -1)">
            −
          </button>

          ${item.qty}

          <button
            onclick="changeQty(${product.id}, 1)">
            +
          </button>

        </span>

      </div>
    `;
  }).join("");

  totalElement.textContent = money(total);
}

function openCart() {
  const cartElement = $("cart");
  const shadeElement = $("shade");

  if (cartElement) {
    cartElement.classList.add("open");
  }

  if (shadeElement) {
    shadeElement.classList.add("show");
  }
}

function closeCart() {
  const cartElement = $("cart");
  const shadeElement = $("shade");

  if (cartElement) {
    cartElement.classList.remove("open");
  }

  if (shadeElement) {
    shadeElement.classList.remove("show");
  }
}

function checkout() {
  if (cart.length === 0) {
    alert("سبد خرید خالی است.");
    return;
  }

  const orderNumber =
    "AJ-" +
    Date.now().toString().slice(-8);

  const total = cart.reduce((sum, item) => {
    const product =
      products.find(p => p.id === item.id);

    return sum +
      (product
        ? product.price * item.qty
        : 0);
  }, 0);

  const order = {
    number: orderNumber,
    total: total,
    items: cart,
    date: new Date().toISOString()
  };

  localStorage.setItem(
    "ayhanLastOrder",
    JSON.stringify(order)
  );

  alert(
    "سفارش آماده شد.\n\n" +
    "شماره سفارش: " +
    orderNumber +
    "\n\n" +
    "مبلغ: " +
    money(total) +
    "\n\n" +
    "برای پرداخت واقعی باید درگاه پرداخت ایرانی به سرور امن متصل شود."
  );
}

document.addEventListener("DOMContentLoaded", () => {

  const cartButton = $("cartBtn");
  const closeButton = $("close");
  const shade = $("shade");
  const search = $("search");
  const checkoutButton = $("checkout");
  const form = $("form");

  if (cartButton) {
    cartButton.addEventListener(
      "click",
      openCart
    );
  }

  if (closeButton) {
    closeButton.addEventListener(
      "click",
      closeCart
    );
  }

  if (shade) {
    shade.addEventListener(
      "click",
      closeCart
    );
  }

  if (search) {
    search.addEventListener(
      "input",
      event => {
        renderProducts(event.target.value);
      }
    );
  }

  if (checkoutButton) {
    checkoutButton.addEventListener(
      "click",
      checkout
    );
  }

  if (form) {
    form.addEventListener(
      "submit",
      event => {

        event.preventDefault();

        const name =
          $("name")?.value.trim() || "";

        const contact =
          $("info")?.value.trim() || "";

        const message =
          $("msg")?.value.trim() || "";

        const request = {
          name: name,
          contact: contact,
          message: message,
          date: new Date().toISOString()
        };

        localStorage.setItem(
          "ayhanRequest_" + Date.now(),
          JSON.stringify(request)
        );

        const status = $("status");

        if (status) {
          status.textContent =
            "درخواست با موفقیت ذخیره شد.";
        }

        form.reset();
      }
    );
  }

  renderProducts();
  renderCart();
});
