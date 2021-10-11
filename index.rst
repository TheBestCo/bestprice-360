The easiest way to install `BestPrice <https://www.bestprice.gr>`__ 360
in your store is the plugin developed for the platform your store is
based on.

Here are the platforms we support so far. Follow the link to install the
plugin:

-  `OpenCart <https://www.opencart.com/index.php?route=marketplace/extension/info&extension_id=38118&filter_member=bestpricegr>`__
-  `Magento <https://marketplace.magento.com/bestprice-bestpriceanalytics.html>`__
-  `Woocommerce <https://wordpress.org/plugins/bestprice-analytics-integration/>`__
-  `CsCart <https://marketplace.cs-cart.com/bestprice-analytics-360.html>`__
-  `PrestaShop <https://www.bestprice.gr/public/assets/360/prestashop_bestpriceanalytics-1.6.x-1.7.x-1.0.2.zip>`__
   (Zip file)

--------------

If your store is not based on any of the platforms listed above, you can
install the script using the "manual" method that follows.

Snippet
-------

Place the following snippet right before closing the ``</body>`` tag
(not inside ``<head>``).

NOTE: The snippet need to be added to the product page and to the cart
page. Ideally, though, it should be added to all pages.

Make sure you replace the ``BESTPRICE_360_KEY_HERE`` with the BestPrice
360 key provided by the BestPrice Team.

.. code:: html

    <script>
    (function (a, b, c, d, s) {a.__bp360 = c;a[c] = a[c] || function (){(a[c].q = a[c].q || []).push(arguments);};
    s = b.createElement('script'); s.async = true; s.src = d; s.charset = 'utf-8'; (b.body || b.head).appendChild(s);})
    (window, document, 'bp', '//360.bestprice.gr/360.js');

    bp('connect', 'BESTPRICE_360_KEY_HERE');
    </script>

Actions
-------

In order to track orders and products, the following two methods are to
be used.

addOrder
~~~~~~~~

.. code:: js

    bp('addOrder', {
      orderId:  '123456',                        // Order ID (alias: order_id)           [Required] 
      revenue:  '1315.25',                       // Grand Total (Cost + Tax + Shipping)  [Required]
      shipping: '5.45',                          // Shipping Cost                        [Required]
      tax:      '301.25',                        // Tax                                  [Required]

      method: 'card|paypal|ondelivery|deposit',  // [Optional]
      currency: 'euro',                          // [Optional]

      // products: []                            // You can pass them here
    });

addProduct(s?)
~~~~~~~~~~~~~~

.. code:: js

    bp('addProduct', {
      orderId:   '123456',                         // Order ID (alias: order_id)      [Required]
      productId: '111222',                         // Product ID (alias: product_id)  [Required]
      title:     'Apple IPhone 11 (64GB) Red EU',  // Product Title (alias: name)     [Required]
      price:     '654.90',                         // Price, should include tax       [Required]
      quantity:  '2'                               // Quantity                        [Required]
    });

Or, you can pass an array like so:

.. code:: js

    bp('addProduct', [products]);

debug
~~~~~

.. code:: js

    bp('debug');

Will enable logging.

Notes
~~~~~

-  If Skroutz Analytics is deployed, then - for convenience - all that's
   needed is inserting the default snippet code.

