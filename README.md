## Snippet

```html
<script>
(function (a, b, c, d, s) {a.__bp360 = c;a[c] = a[c] || function (){(a[c].q = a[c].q || []).push(arguments);};
s = b.createElement('script'); s.async = true; s.src = c; b.body.appendChild(s);})
(window, document, 'bp', 'https://360.bestprice.gr/360.js');

bp('connect', 'BESTPRICE_360_KEY_HERE');
</script
```

## Actions

### addOrder
```js
bp('addOrder', {
  orderId: '123456',    // Order ID. Required. (alias: order_id)
  revenue:  '1315.25',  // Grand Total (Cost + Tax +Shipping).
  shipping: '5.45',     // Shipping Cost
  tax:      '301.25'    // Tax.
});
```

### addProduct(s?)
```js
bp('addProduct', {
  orderId:   '123456',                                    // Order ID. Required. (alias: order_id)
  productId: '111222',                                    // Product ID. Required. (alias: product_id)
  title:       'Apple IPhone 6 Plus (16GB) Space Gray EU',  // Product title. Required. (alias: name)
  price:      '654.90',                                    // Price. Required.
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
