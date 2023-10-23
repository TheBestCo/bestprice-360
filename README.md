The easiest way to install [BestPrice](https://www.bestprice.gr) 360º in your store is the plugin developed for the platform your store is based on.

Here are the platforms we support so far. Follow the link to install the plugin:

- OpenCart
  - [Marketplace](https://www.opencart.com/index.php?route=marketplace/extension/info&extension_id=38118&filter_member=bestpricegr)
  - [Download v2.0](https://www.bestprice.gr/public/360-plugins/opencart/2.0/opencart_bestprice.analytics.2.0-1.0.8.ocmod.zip)
  - [Download v2.1](https://www.bestprice.gr/public/360-plugins/opencart/2.1/opencart_bestprice.analytics.2.1-1.0.8.ocmod.zip)
  - [Download v2.2](https://www.bestprice.gr/public/360-plugins/opencart/2.2/opencart_bestprice.analytics.2.2-1.0.8.ocmod.zip)
  - [Download v2.3](https://www.bestprice.gr/public/360-plugins/opencart/2.3/opencart_bestprice.analytics.2.3-1.0.8.ocmod.zip)
  - [Download v3.x](https://www.bestprice.gr/public/360-plugins/opencart/3.x/opencart_bestprice.analytics.3.x-1.0.8.ocmod.zip)
  - [Download v4.0.1](https://www.bestprice.gr/public/360-plugins/opencart/4.0.1/bestpriceanalytics.ocmod.zip)
  - [Download v4.0.2](https://www.bestprice.gr/public/360-plugins/opencart/4.0.2/bestpriceanalytics.ocmod.zip)
- Magento
  - [Marketplace](https://marketplace.magento.com/bestprice-bestpriceanalytics.html)
  - [Download](https://www.bestprice.gr/public/assets/360/magento_bestprice_bestpriceanalytics_2.x-1.0.6.zip)
- [Woocommerce](https://wordpress.org/plugins/bestprice-analytics-integration/)
- CsCart
  - [Marketplace](https://marketplace.cs-cart.com/bestprice-analytics-360.html)
  - [Download](https://www.bestprice.gr/public/360-plugins/cscart/4.x/cs-cart-bestpriceanalytics_4.x-1.0.5.zip)
- [PrestaShop](https://www.bestprice.gr/public/360-plugins/prestashop/prestashop_bestpriceanalytics-1.6x-1.7x-8.x-1.0.6.zip) (Zip file)

----

If your store is not based on any of the platforms listed above, you can install the script using the "manual" method that follows.

## Snippet

Place the following snippet right before closing the `</body>` tag (not inside `<head>`).

NOTE: The snippet need to be added to the product page and to the cart page. Ideally, though, it should be added to all pages.

Make sure you replace the `BESTPRICE_360_KEY_HERE` with the BestPrice 360 key provided by the BestPrice Team.


```html
<script>
(function (a, b, c, d, s) {a.__bp360 = c;a[c] = a[c] || function (){(a[c].q = a[c].q || []).push(arguments);};
s = b.createElement('script'); s.async = true; s.src = d; s.charset = 'utf-8'; (b.body || b.head).appendChild(s);})
(window, document, 'bp', '//360.bestprice.gr/360.js');

bp('connect', 'BESTPRICE_360_KEY_HERE');
</script>
```

## Actions

In order to track orders and products, the following two methods are to be used.

### addOrder
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

### addProduct(s?)
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

### debug

```js
bp('debug');
```

Will enable logging.

## CORS

If you are utilizing a [CORS policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS), you will need to include entries for `*.bestprice.gr` on `script-src` and `connect-src` directives. Otherwise, the network requests needed will be blocked.

## Notes
- The product URLs submitted via the XML feed should match the canonical URLs of the product pages. BestPrice 360 won’t work properly on some browsers if they don't.
