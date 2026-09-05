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

NOTE: The snippet must be added to product pages, cart pages, **and the checkout success / thank-you (order confirmation) page**. On the success page, emit `addOrder` and at least one `addProduct` only after the purchase is confirmed; installing the loader only on product and cart pages cannot register completed orders. Ideally, the snippet should be added to all pages. It should always be served (not only after accepting cookies, etc.).

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
        const checkout = event?.data?.checkout;
        const rawOrderId = checkout?.order?.id;

        // Shopify documents several MoneyV2 fields as nullable. Preserve a
        // real zero, but reject missing / non-numeric amounts instead of
        // serializing "undefined" or silently turning missing prices into 0.
        const readAmount = (money) => {
            const raw = money?.amount;
            if (raw === null || raw === undefined || raw === '') return null;
            const amount = Number(raw);
            return Number.isFinite(amount) ? amount : null;
        };

        const revenue = readAmount(checkout?.totalPrice);
        if (!checkout || rawOrderId === null || rawOrderId === undefined ||
            !String(rawOrderId).trim() || revenue === null) {
            throw new Error('checkout_completed event is missing order id or total price');
        }

        const orderId = String(rawOrderId);

        // The pixel runs in a Shopify sandbox and CANNOT read the storefront's
        // cookie/localStorage. The BestPrice 360.js stamps the visitor identity
        // (gid + signed click token) into the CART ATTRIBUTES, which DO reach the
        // pixel here on event.data.checkout.attributes — read those first.
        const attributes = Array.isArray(checkout.attributes) ? checkout.attributes : [];
        const bpAttrs = attributes.reduce((m, a) => {
            if (a && a.key) m[a.key] = a.value;
            return m;
        }, {});

        let bpSessionValue = bpAttrs.bp_cookie_session;
        if (!bpSessionValue) bpSessionValue = await browser.cookie.get(cookieName);
        if (!bpSessionValue) bpSessionValue = await browser.localStorage.getItem('__bpgid');

        const bpClickToken = bpAttrs.bp_click_token || "";

        const bpQueue = [];
        bpQueue.push(["connect", bestpriceKey]);
        const order = {
            orderId:           orderId,
            revenue:           String(revenue),
            shipping:          String(readAmount(checkout.shippingLine?.price) ?? 0),
            tax:               String(readAmount(checkout.totalTax) ?? 0),
            bp_cookie_session: String(bpSessionValue || ""),
            bp_click_token:    String(bpClickToken || "")
        };
        const currency = checkout.currencyCode || checkout.totalPrice?.currencyCode;
        if (currency) order.currency = String(currency);
        bpQueue.push(["addOrder", order]);
        bpQueue.push(["native", true]);

        const lineItems = Array.isArray(checkout.lineItems) ? checkout.lineItems : [];
        let productCount = 0;
        lineItems.forEach((item) => {
            const rawProductId = item?.variant?.id ?? item?.id;
            const quantity = Number(item?.quantity);
            let price = readAmount(item?.variant?.price);
            if (price === null) price = readAmount(item?.unitPrice);
            if (price === null) price = readAmount(item?.price);

            // finalLinePrice is the whole line. Use it only as a last-resort
            // unit-price fallback and only when quantity is usable.
            if (price === null && Number.isFinite(quantity) && quantity > 0) {
                const finalLinePrice = readAmount(item?.finalLinePrice);
                if (finalLinePrice !== null) price = finalLinePrice / quantity;
            }

            if (rawProductId === null || rawProductId === undefined ||
                !String(rawProductId).trim() || !Number.isFinite(quantity) ||
                quantity <= 0 || price === null) {
                console.warn('BestPrice Tracking: skipped line item with missing id, quantity, or price');
                return;
            }

            const productId = String(rawProductId);
            bpQueue.push(["addProduct", {
                orderId:   orderId,
                productId: productId,
                title:     String(item?.title || item?.variant?.product?.title || `Product ${productId}`),
                price:     String(price),
                quantity:  String(quantity)
            }]);
            productCount += 1;
        });

        if (productCount === 0) {
            throw new Error('checkout_completed event has no line item with a usable id, quantity, and price');
        }

        const response = await fetch('https://www.bestprice.gr/api/order', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(bpQueue),
            keepalive: true
        });

        if (!response || !response.ok) {
            throw new Error(`BestPrice order request failed${response ? ` (${response.status})` : ''}`);
        }

    } catch (e) {
        console.error('BestPrice Tracking Error:', e);
    }
});
```

*Replace `BESTPRICE_360_KEY_HERE` with the merchant's key. If it hasn't been provided by the BestPrice Team, you can find it [here](https://merchants.bestprice.gr/account/360).*

4. Click **Save** and then **Connect**.

### Save Order prompt on Shopify

Shopify Custom Pixels run in a sandboxed iframe and cannot access or write to
the top-frame DOM. For that reason, the pixel itself cannot display the
BestPrice iframe on the Thank you page. Do not defer or replay that popup on a
later storefront page: it would be detached from the checkout action and could
surprise or annoy the buyer.

The supported way to show Save Order immediately on the Thank you page is a
Shopify Checkout UI extension, using a target such as
[`purchase.thank-you.block.render`](https://shopify.dev/docs/api/checkout-ui-extensions/latest/targets/thank-you/block).
Use a compact inline block with an explicit **Save to BestPrice** action; open
login only after that click. This requires a BestPrice Shopify app/extension to
be installed and placed by the merchant; a Custom Pixel cannot provide that UI
surface.

---

## Useful Notes for All Integrations

### CORS

If you are utilizing a [CORS policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), you will need to include entries for `*.bestprice.gr` on `script-src` and `connect-src` directives. Otherwise, the network requests that are needed will be blocked.

### Notes
- The product URLs submitted via the XML feed should match the canonical URLs of the product pages. BestPrice 360 won’t work properly on some browsers if they don't.
