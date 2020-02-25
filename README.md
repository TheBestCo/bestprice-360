The easiest way to install BestPrice 360º in your store is the plugin developed for the platform your store is based on.

Here are the platform we support so far (follow the link to install the plugin)

- [OpenCart](https://www.opencart.com/index.php?route=marketplace/extension/info&extension_id=38118&filter_member=bestpricegr)
- Magento
  - [M1](https://marketplace.magento.com/bestprice-bestprice-analytics.html)
  - [M2](https://marketplace.magento.com/bestprice-bestpriceanalytics.html)
- [Woocommerce](https://wordpress.org/plugins/bestprice-analytics-integration/)
- [CsCart](https://marketplace.cs-cart.com/bestprice-analytics-360.html)
- [PrestaShop](https://www.bestprice.gr/public/assets/360/prestashop_bestpriceanalytics-1.6.x-1.7.x-1.0.1.zip) (Zip file)

----

If your store is not based on any of the platform listed above, you can install the script using the "manual" method that follows.

## Snippet

Place the following snipet right before closing the `</body>` tag (not inside `<head>`).

Make sure you replace the `BESTPRICE_360_KEY_HERE` with the BestPrice 360º key provided by the BestPrice team.


```html
<script>
(function (a, b, c, d, s) {a.__bp360 = c;a[c] = a[c] || function (){(a[c].q = a[c].q || []).push(arguments);};
s = b.createElement('script'); s.async = true; s.src = d; (b.body || b.head).appendChild(s);})
(window, document, 'bp', '//360.bestprice.gr/360.js');

bp('connect', 'BESTPRICE_360_KEY_HERE');
</script>
```

## Actions

In order to track orders and products, the following two methods are to be used.

### addOrder
```js
bp('addOrder', {
  orderId: '123456',    // Order ID. Required. (alias: order_id)
  revenue:  '1315.25',  // Grand Total (Cost + Tax +Shipping).
  shipping: '5.45',     // Shipping Cost
  tax:      '301.25'    // Tax.
  
  method: 'card|paypal|ondelivery|deposit' (optional)
  currency: 'euro', (optional)
  
//  products: [] // We can pass them here
});
```

### addProduct(s?)
```js
bp('addProduct', {
  orderId:   '123456',                                    // Order ID. Required. (alias: order_id)
  productId: '111222',                                    // Product ID. Required. (alias: product_id)
  title:       'Apple IPhone 6 Plus (16GB) Space Gray EU',  // Product title. Required. (alias: name)
  price:      '654.90',                                    // Price. Required. (Should include tax)
  quantity:   '2'                                          // Quantity. Required.
});
```
Or you can pass an array

```js
bp('addProduct', [products])`;
```

### debug

```js
bp('debug');
```

Will enable loggging.

### Notes
- If Skroutz Analytics is deployed, then - for convenience - all needed is inserting the snippet code.
