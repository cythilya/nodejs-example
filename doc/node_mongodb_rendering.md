#Node - 從MongoDB取得資料到套版
經歷了前面關於Node基本安裝設定、模板引擎的熟悉，與一些基本的語法和操作，接著來到MongoDB的建置和使用，終於要把假資料換掉，使用真實世界的內容展現在畫面上啦。 

##製作Model
###定義欄位與資料型別
使用Model來定義個別Collection的欄位與資料型別。例如，一筆商品資料必定含有ID(並非DB中每筆資料的真實ID)、商品名稱(name)、商品描述(description)、價格(price)、庫存狀態(available)、圖片(images)等。

	var mongoose = require('mongoose');
	var productsSchema = new mongoose.Schema({
	    id: String,
	    name: String,
	    description: String,
	    images: [String],
	    price: Number,
	    available: Boolean
	}, {collection: 'products'});
	
	var Products = mongoose.model('Products', productsSchema);
	
	//放假資料的位置
	
	module.exports = Products;

###新增一筆假資料並存回DB
此時DB是沒有任何資料的，因此準備一筆或多筆假資料存進去。完成後放在剛剛預留放假資料的地方。

	var product = new Products({
	    id: '1',
	    name: '商品名稱',
	    description: '商品描述',
	    images: [
	        'http://dummyimage.com/600x400/000/fff',
	        'http://dummyimage.com/400x400/000/fff',
	        'http://dummyimage.com/600x600/000/fff'
	    ],
	    price: 490,
	    available: true
	});
	
	product.save(function(err, products) {
	    if (err) return console.error(err);
	});

##從Controller接DB的資料
使用 `npm start` 執行專案，看到DB有資料了。  

![Node - 從MongoDB取得資料到套版](https://lh3.googleusercontent.com/DMF0tX27PB6NJBlUpcTnCrZ1UjSywJTP8jYK3cUK7Xs7oQm7r1Q4Tg7RxQ2Sc_cbAol7L_U7UY7nuYTvjIVzbtYbgofFtohW_Jh6IdV06Y4ck9T7fQiUOvU5VYhM91YpEQ-QF982IE2KWY-2OpqxBEPxp_tfYXw0nI4R8K90ScjbatHWk2xqxod7JnR-Bl1w4Ph7DKecGnkcVukZTMhrEGE6lEwx96UhwaDjhvA3NBelgR7uVWi96YGwSUmM15xIkDQGb0MYwxA946FS3d2dBw-3qskqAGcTRw-q8ii4jTKonpAiaQhD6vKeAKjNDbf8fCiCPYpc9ypVBwa09Uc9jAdPi1BMBBlXO3aajRGbb5HTT4tR1CpldQK2ednoqnnTuQShybpGoboLGmTw5c3XZGtl4lCsUGRZYIrB01zxX747WuwqC4OKBxTgvditNfB0PrnEMsabHAWxIagTuq_IBJN1n5MWaKvkjMPcy_ynkueDXaGw9xsw1HWtDWcoD_TJXxz1D4J-erSIgZn_YZkqvRM=w340-h244-no)  

既然DB有資料了，就取出來用吧。 

###取出整個collection
取資料。

	var express = require('express'),
	    router = express.Router(),
	    mongoose = require('mongoose'),
	    Product = require('../models/product.js'); //記得要載入Model
	
	//get products from db
	var productList = {};
	Product.find(function(err, goods) {
	    if (err) {
	        console.error(err);
	    } 
	    else {
	        productList = goods;
	    }
	});
	
	//render page
	router.get('/', function(req, res, next) {
	    res.render('index', {
	        products: productList //放到回傳給View的Response裡面，套板用
	    });
	});
	
	module.exports = router;

###只取出特定資料
我們經由網址列的參數id來決定要取哪一個商品，例如：`localhost:3000/product?id=1`。

	var express = require('express'),
		router = express.Router(),
		mongoose = require('mongoose');
	
	//get single product from db
	var Product = require('../models/product.js');
	var productDetail = {};
	
	router.get('/', function(req, res, next) {
	    Product.findOne({ id: req.param('id') }, function(error, goods) {
	        if (error) {
	            return handleError(error);
	        }
	        productDetail = goods;
	
	        res.render('product', {
	            product: productDetail
	        });
	    });
	});
	
	module.exports = router;

##將資料代入View
View在這裡使用樣板引擎[Handlebars.js](http://handlebarsjs.com)。代入名稱和Controller中的Response欄位名稱相同。

###取出整個collection
	
	{{#each products}}
	<article class="product">
		<div class="imageBlock">
			<img class="thumbnail" src="{{ images.[0] }}" alt="{{ name }}">
		</div>
		<h3 class="title">{{ name }}</h3>
		<div class="description">{{ description }}</div>
		<div class="price">{{ price }} NTD</div>
	</article>
	{{/each}}

###只取出特定資料 - 單一商品

	<div class="productIntro">
		<div class="imagePreview">
			<img class="thumbnail" src="{{ product.images.[0] }}" alt="{{ product.name }}">
		</div>
		<div class="previewArea">
			<div class="imageList">
				{{#each product.images}}
				<div class="imageItem" data-mediasrc="{{ this }}">
					<img class="thumbnail" src="{{ this }}" alt="{{ ../product.name }}">
				</div>
				{{/each}}
			</div>
		</div>
		<div class="productInfo">
			<h2 class="productName">{{ product.name }}</h2>
			<div class="productID">產品編號：{{ product._id }}</div>
			<h3 class="productDescription">{{ product.description }}</h3>
		</div>
		<div class="title">會員價：NT$ {{ product.price }}</div>
	</div>

##Demo
頁面展示存在DB裡面的資料。

###取出整個collection
![Node - 從MongoDB取得資料到套版](https://lh3.googleusercontent.com/2LtX_xzyp2xaafpqFYHb0tZDywyffEyURMDEAqY1LI6EDWawP04PcjT9eB3e-mNGnZFVlp0byY2SNtHDXHDosyOfXtfpbcNCTwkggH8BrpfTrY6Uq8PvycXSnLmUT-lBe87nLdh2MenncG_0vmDTvP2E3SzXrZcNqT4fiatEFsm083LOPwtL4M7d0KrmUQmeM5y2tOau12rxLHyBt1Im10Fo64xl-OqV3UU3ecjRll-A8hbm96I0nrOQNOpdksUFs-HyS9YSGLGOJjhPR5zmFxeJO1uGFS2bC7vaM0D4aCE3vbTCSeiwEs9sdYvObo07Dy0XlUVRlo8xlTNnSWLTOPV1FhH3nyxISClX8NKemLeAYpbcr65NGFX5XAaBSarLDf9yEybqXDLU6ZVN7lm6lpUR1vonVP_zcVUtn8jBEcL0zYaXJuQ3hqGEVVLtHDeO2sFYDVBJ6GAu5093L7IUIL95-6WOEDmGCzKtuJceR_TVC6UKiP9-Y9LpOcSLbGvL9Sx7hVdB-dN0GiEXSGrzX4g=w800-h479-no)

###只取出特定資料 - 單一商品
![Node - 從MongoDB取得資料到套版](https://lh3.googleusercontent.com/pRA-nUDr7lodH48x4BjGvdQjeBOPGIEuq7KSr_93f2TWypNRYFvUxflnLubpR6BTIfb17yJWEgFE7cmYed8O8QORhN8k30EFyk1_ZFZvATGtHxNVX8Ve-h79ILYPGAG_CxWkNUm_hBI1n0nNRgVMqT9miRUbZKZ60Pi5nFApWer7VYjDW2Uyax4Vq13r1-Ov3qy8AqCb3H8nYMUBdfcjzjzKtMhIV7c31F4usSFii3XKwW7NgmnygvyRFhJAfcgEr_wv8DbUB7853J7qxVaaxmKcC8OsHSar7OyPvO_iG3vggM6LLUAOTU4XwZNamrOs3fRbjZmse0IhSG2CXe6cVkdcwf-MUJfknfsX_f11jjIgrqj9CVl_KSk2IdDGD6_SWHfJpiR_4RAizuCnbV2X0yk3fZP1yZyeX-_S6KDCBYbh4wbJrOuyEIbBhKtOdJ5j2UrpfJZ0a2RRHMabWu8MmME20enKXi5qkDlNipvkWsajjfaRLzKf-A0P5U3IOx-3bO2tJeLNV4l2rr1LLDSBegE=w800-h421-no)

---
[Node - 從MongoDB取得資料到套版](http://cythilya.blogspot.tw/2015/09/node-mongodb-rendering.html)：網誌版。