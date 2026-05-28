# BestPrice 360 Installation Guide

## Table of Contents
- [Plugins](#plugins)
- [Custom Installation](#custom-installation)
- [Shopify Integration](#shopify-integration)
- [Useful Notes for All Integrations](#useful-notes-for-all-integrations)

---

## Plugins

The easiest way to install [BestPrice](https://www.bestprice.gr) 360º in your store is the plugin developed for the platform your store is based on.

Here are the platforms we support so far. Follow the link to install the plugin:

- [Woocommerce](https://wordpress.org/plugins/bestprice-analytics-integration/)
- Magento
  - [Marketplace](https://marketplace.magento.com/bestprice-bestpriceanalytics.html)
  - [Download](https://www.bestprice.gr/public/assets/360/magento_bestprice_bestpriceanalytics_2.x-1.0.6.zip)
- CsCart
  - [Marketplace](https://marketplace.cs-cart.com/bestprice-analytics-360.html)
  - [Download](https://www.bestprice.gr/public/360-plugins/cscart/4.x/cs-cart-bestpriceanalytics_4.x-1.0.5.zip)
- PrestaShop
  - [Unified Plugin (1.7.x, 8.x, 9.1) — v1.1.0](https://www.bestprice.gr/public/360-plugins/prestashop/unified/prestashop_bestprice360-unified-1.1.0.zip) *(recommended)*
  - [Legacy Plugin (1.6.x) — v1.0.6](https://www.bestprice.gr/public/360-plugins/prestashop/prestashop_bestpriceanalytics-1.6x-1.7x-8.x-1.0.6.zip)
  - [Source & Changelog on GitHub](https://github.com/TheBestCo/bestprice-360-plugins/tree/master/prestashop)

---

## Custom Installation

If your store is not based on any of the platforms listed above, you can install the script using the "manual" method that follows.

### Snippet

Place the following snippet right before closing the `</body>` tag (not inside `<head>`).

NOTE: The snippet must be added to the product and cart pages. Ideally, though, it should be added to all pages. Also, the snippet should always be served (not after accepting cookies, etc.).

Make sure you replace the `BESTPRICE_360_KEY_HERE` with the BestPrice 360 key provided by the BestPrice Team. If it hasn't been provided, you can find it [here](https://merchants.bestprice.gr/account/360).


```html
<script>
(function (a, b, c, d, s) {a.__bp360 = c;a[c] = a[c] || function (){(a[c].q = a[c].q || []).push(arguments);};
s = b.createElement('script'); s.async = true; s.src = d; s.charset = 'utf-8'; (b.body || b.head).appendChild(s);})
(window, document, 'bp', '//360.bestprice.gr/360.js');

bp('connect', 'BESTPRICE_360_KEY_HERE');
</script>
```

### Actions

The following two methods are to be used to track orders and products.

#### addOrder
```js
bp('addOrder', {
  orderId:  '123456',                        // Order ID (alias: order_id)           [Required] 
  revenue:  '1315.25',                       // Grand Total (Cost + Tax + Shipping)  [Required]
  shipping: '5.45',                          // Shipping Cost                        [Required]
  tax:      '301.25',                        // Tax                                  [Required]

  method: 'card|paypal|ondelivery|deposit|atstore',  // [Optional]
  currency: 'euro',                          // [Optional]

  // products: []                            // You can pass them here
});
```

#### addProduct(s?)
```js
bp('addProduct', {
  orderId:   '123456',                         // Order ID (alias: order_id)      [Required]
  productId: '111222',                         // Product ID (alias: product_id)  [Required]
  title:     'Apple IPhone 11 (64GB) Red EU',  // Product Title (alias: name)     [Required]
  price:     '654.90',                         // Price, should include tax       [Required]
  quantity:  '2'                               // Quantity                        [Required]
});
```
Or, you can pass an array like so:

```js
bp('addProduct', [products]);
```

#### debug

```js
bp('debug');
```

Will enable logging.

---

## Shopify Integration

### Step 1: Theme Integration

1. Go to **Shopify Admin > Online Store > Themes**
2. Click **...** next to the active theme → **Edit code**
3. Open **`theme.liquid`**
4. Paste this **before `</head>`**:

```html
<!-- BestPrice 360 Start -->
<script>
(function(w,d,s,u) {
    w['bp']=w['bp']||function(){(w['bp'].q=w['bp'].q||[]).push(arguments)};
    var a=d.createElement(s),m=d.getElementsByTagName(s)[0];
    a.async=1;a.src=u;m.parentNode.insertBefore(a,m);
})(window,document,'script','https://360.bestprice.gr/360.js');
bp('connect', 'BESTPRICE_360_KEY_HERE');
</script>
<!-- BestPrice 360 End -->
```
 
*Replace `BESTPRICE_360_KEY_HERE` with the merchant's key (e.g. `BP-XXXXX-XXXXXXXXXX`). If it hasn't been provided by the BestPrice Team, you can find it [here](https://merchants.bestprice.gr/account/360).*

5. Click **Save**

---

### Step 2: Custom Pixel

1. Go to **Settings > Customer Events**
2. Click **Add custom pixel** → name it `BestPrice 360` → **Add pixel**
3. Paste this code:

```javascript
analytics.subscribe('checkout_completed', async (event) => {
    try {
        const bestpriceKey = 'BESTPRICE_360_KEY_HERE';
        const cookieName = '__bp-gid';

        let bpSessionValue = await browser.cookie.get(cookieName);
        if (!bpSessionValue) {
            bpSessionValue = await browser.localStorage.getItem('__bpgid');
        }

        const bpQueue = [];
        bpQueue.push(["connect", bestpriceKey]);
        bpQueue.push(["addOrder", {
            orderId:          String(event.data.checkout.order.id),
            revenue:          String(event.data.checkout.totalPrice.amount),
            shipping:         event.data.checkout.shippingLine ? String(event.data.checkout.shippingLine.price.amount) : "0",
            tax:              event.data.checkout.totalTax ? String(event.data.checkout.totalTax.amount) : "0",
            currency:         event.data.checkout.currencyCode,
            bp_cookie_session: String(bpSessionValue || "")
        }]);
        bpQueue.push(["native", true]);

        if (event.data.checkout.lineItems?.length > 0) {
            event.data.checkout.lineItems.forEach(item => {
                bpQueue.push(["addProduct", {
                    orderId:   String(event.data.checkout.order.id),
                    productId: String(item.variant ? item.variant.id : item.id),
                    title:     item.title,
                    price:     String(item.variant?.price?.amount ?? 0),
                    quantity:  String(item.quantity)
                }]);
            });
        }

        fetch('https://www.bestprice.gr/api/order', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(bpQueue),
            keepalive: true
        });

    } catch (e) {
        console.error('BestPrice Tracking Error:', e);
    }
});
```

*Replace `BESTPRICE_360_KEY_HERE` with the merchant's key. If it hasn't been provided by the BestPrice Team, you can find it [here](https://merchants.bestprice.gr/account/360).*

4. Click **Save** and then **Connect**.

---

### Step 3: Verify the Cookie is Being Set

The Custom Pixel in Step 2 reads the `__bp-gid` cookie / `__bpgid` LocalStorage value
that **Step 1's theme snippet** sets on every page-view. If Step 1 is missing —
or only loaded on the homepage but not on product/cart pages — the cookie
never gets set, the Custom Pixel sends an empty `bp_cookie_session`, and
**every order arrives at BestPrice unattributed**.

To verify Step 1 is doing its job, open the storefront in a regular browser
window (not a Shopify Preview), visit a **product page**, then a **cart page**,
then open DevTools → Console and run:

```js
console.table({
  cookie:       document.cookie.match(/__bp-gid=([^;]+)/)?.[1] || '(missing)',
  localStorage: localStorage.getItem('__bpgid') || '(missing)'
});
```

Expected: at least one of `cookie` or `localStorage` shows a long base64-like
string (e.g. `MXNrbU0xLndVal8sb35QZE9oVzAqOA==`). If both show `(missing)`,
Step 1 is not loading on that page — re-check `theme.liquid` and confirm the
snippet is **before `</head>`** and not conditionally rendered.

---

## Useful Notes for All Integrations

### CORS

If you are utilizing a [CORS policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), you will need to include entries for `*.bestprice.gr` on `script-src` and `connect-src` directives. Otherwise, the network requests that are needed will be blocked.

### Notes
- The product URLs submitted via the XML feed should match the canonical URLs of the product pages. BestPrice 360 won’t work properly on some browsers if they don't.
