#Node - 表單處理與檔案上傳 (Form Handling and File Uploads)
表單處理與檔案上傳。  

##將用戶端資料傳送給伺服器
將使用者的資訊傳遞給伺服器有兩種方法：「查詢字串」(querystring)與「請求內文」(request body)。

- 查詢字串(querystring)
	- 使用GET方式傳送表單資料。
	- 表單資料會直接顯示在瀏覽器網址列上，因此可以被暫存、書籤起來或留存在歷史紀錄中。公開透明，敏感資料不適用。
	- 由於在瀏覽器網址列有長度的限制(最大URL長度是2048 characters)，只適合資料量小的情況。但也因資料量少，速度快。
	- 資料只允許ASCII characters。
- 請求內文(request body)
	- 使用POST方式傳送表單資料。
	- 表單資料不會直接顯示在使用者可以看到的地方，而是放在HTTP message body裡面。
	- 無資料量限制。
	- 資料可為binary data，因此可傳輸多媒體檔等檔案格式。

看起來似乎使用「查詢字串」的GET方式傳送表單資料是不安全的，而使用「請求內文」POST方式傳送表單資料是安全的。但實際上，兩者只要使用 **HTTPS** 都是安全的，兩者不使用 **HTTPS*** 都不是安全的。因為如果不使用HTTPS，我們依然可以看到POST的內文資料，容易度與得到GET查詢字串無異。

總而言之，基於資料傳輸量的限制和保持瀏覽器網址列的簡潔乾淨，建議使用「請求內文」POST方式傳送表單資料。

###補充說明
- [HTTP Methods: GET vs. POST](http://www.w3schools.com/tags/ref_httpmethods.asp)：想了解HTTP GET/POST基本概念可以看這篇文章。
- [淺談 HTTP Method：表單中的 GET 與 POST 有什麼差別？](http://blog.toright.com/posts/1203/%E6%B7%BA%E8%AB%87-http-method%EF%BC%9A%E8%A1%A8%E5%96%AE%E4%B8%AD%E7%9A%84-get-%E8%88%87-post-%E6%9C%89%E4%BB%80%E9%BA%BC%E5%B7%AE%E5%88%A5%EF%BC%9F.html)：關於HTTP GET/POST基本概念，如果英文版的看不習慣，這篇是中文版的。
- [HTML 表單中 GET 與 POST 的用法差異](http://www.wibibi.com/info.php?tid=235)：想了解在表單處理上，GET/POST的差異可以看這篇文章。

---
##HTML表單
範例表單如下：

	<form action="/process" method="POST">
		<ul>
			<li>
				<input type="hidden" name="hush" val="hidden, but not secret!">
			</li>
		    <li>
		    	<label for="fieldColor">Your favorite color: 
					<input type="text" id="fieldColor" name="color">
				</label>
			<li>
		</ul>
		<button type="submit">Submit</button>
	</form>

- 如果沒有在表單 `<form>` 的method屬性指定使用的方法(GET或POST)，**預設會是GET**。
- 如果沒有在表單 `<form>` 的action屬性指定路徑(在本例中為 `/process`)，預設表單傳送的路徑會與載入表單的路徑相同。因此建議一定要設定(無論使用Form Post還是Ajax傳送表單)，避免資料遺失。
- 伺服器以 `<input>` 的name屬性來辨識欄位。
- 在本例中，當使用者按下Submit按鈕提交表單時，`/process` URL會被呼叫，欄位值會使用POST方式在請求內文中被傳送至伺服器端。

---
##編碼
當表單被提交的時候，它必定會以某種方式編碼。

- 如果沒有特別指定，預設會使用 `application/x-www-form-urlencoded` 。這種編碼方式專門用來編碼URL且Express支援。
- 上傳檔案則需使用 `multipart/form-data` 。由於Express即將不支援且不建議使用，因此之後會討論替代方案。

###補充說明
- [四種常見的POST 提交數據方式](https://www.imququ.com/post/four-ways-to-post-data-in-http.html)：這篇文章把四種常用的編碼方式(application/x-www-form-urlencoded、multipart/form-data、application/json、text/xml)講解得非常清楚。

---
##表單的處理方式
- 使用同一個路徑來處理傳送/回應表單：FORM POST(傳送表單資料)、FORM GET(取得傳送後的回應)。
- 使用不同路徑來處理傳送/回應表單
	-  傳送表單的方法：由於使用不同路徑來傳送和回應表單，因此傳送表單使用FORM POST或GET皆可，但依舊不建議使用GET，除了資料容量限制外，也避免將敏感資訊直接暴露在瀏覽器網址列上。
	-  回應表單的方法
		- HTML回應：直接將HTML傳回給瀏覽器。若使用者重新整理頁面會出現警告、干擾書籤、回上一頁的功能，不建議使用。
		- 302 Redirect：302表示「暫時轉頁」，但在處理表單上是濫用302的意義，因此建議改用303。
		- 303 Redirect：這是比302更好的解法，303的意義為「通知Client端到另一個網址去查看上傳表單的結果」。而我們可在提交表單後轉跳到：
			- 重新導向至專用的成功/失敗頁面。優點是易於分析與實作，缺點是需針對每個狀況設計、實作與維護頁面。
			- 重新導向至原本的頁面，並使用一個閃爍的訊息(flash message)。為了不中斷使用者的瀏覽歷程，可用ajax提交表單而不需轉跳到其他頁面，或使用FORM POST/GET提交表單並導向原本的頁面，然後提示。
			- 重新導向至新的位置，並使用一個閃爍的訊息。同上，可以使用ajax提交表單，或使用FORM POST/GET提交表單，然後將使用者帶往下一個流程頁面。

###補充說明
- [網頁開發人員應了解的 HTTP 狀態碼](http://blog.miniasp.com/post/2009/01/16/Web-developer-should-know-about-HTTP-Status-Code.aspx)：保哥整理了HTTP 狀態碼並做了詳細說明，但沒用到通常不會太仔細看啦(笑)。

---
##使用Express處理表單
Server端以「name」這個屬性來辨識HTML Form的欄位。例如，如下範例，`<input type="text" name="name">` 就是以 `req.query.name` 傳遞給Server端。

###安裝與設定body-parser
如果使用POST來傳送表單資料，必須要有middleware來解析以URL編碼的內文。因此要安裝body-parser：

	npm install --save body-parser  

然後在app.js把它include進來：  

app.use(require('body-parser')()); 

###HTML Snippet
我們準備一個表單供使用者填寫後提交。這裡有一個隱藏的欄位「_csrf」，它並不會讓使用者直接在畫面上看見，但傳送表單的時候會將此欄位傳送出去。  

	<form class="formNewsLetter" action="/process?form=newsletter" method="POST">
		<div class="formContainer">
			<h2>Sign up for our newsletter to receive news and specials!</h2>
			<input type="hidden" name="_csrf" value="{{ csrf }}">
			<ul>
				<li>
					<label>
						Name: <input type="text" name="name">
					</label>
				</li>
				<li>
					<label>
						Email: <input type="text" name="email">
					</label>
				</li>
			</ul>
			<input type="submit" value="submit">
		</div>
	</form>
	
###app.js
Router設定，當表單傳送到Server端後，印出欄位值，最後會導到「thankyou」這個成功頁面。

	app.post('/process', function(req, res){
	    console.log('Form (from querystring): ' + req.query.form);
	    console.log('CSRF token (from hidden form field): ' + req.body._csrf);
	    console.log('Name (from visible form field): ' + req.body.name);
	    console.log('Email (from visible form field): ' + req.body.email);
	    res.redirect(303, '/thankyou');
	});

一開始在畫面上會先代入_csrf這個欄位的值。

	//get index page
	router.get('/', function(req, res, next) {
		res.render('index', { title: 'Index', csrf: '0000-1111-2222-3333' });
	});

這當中我們沒有使用client端的javascript任何的協助，純粹只是用表單原生的POST功能。

###Demo
填寫表單，填寫完成後送出。  

![Node - 使用Express處理表單](https://lh3.googleusercontent.com/xhsPNH28HWMNb25nXinFP9qu6xo0f407NWmgmKivrDo=w580-h163-no)  

Server端收到Client端POST過來的資料，並轉到「thankyou」這個成功頁面。  

![Node - 使用Express處理表單](https://lh3.googleusercontent.com/zWiFLu51hpoTBlKpYSj4bJ02ATRtcbdR8lfYH8uxTlw=w455-h91-no)  

###補充說明
- [使用 NodeJS + Express 從 GET/POST Request 取值](http://fred-zone.blogspot.tw/2012/02/nodejs-express-getpost-request.html)

---
##處理Ajax表單
在上一個例子中，使用 表單原生的POST功能，而接下來要改用ajax來處理表單。

###HTML Snippet
我們一樣準備一個表單供使用者填寫後提交。

	<form class="formNewsLetter" action="/process?form=newsletter" method="POST">
		<div class="formContainer">
			<h2>Sign up for our newsletter to receive news and specials!</h2>
			<input type="hidden" name="_csrf" value="{{ csrf }}">
			<ul>
				<li>
					<label>
						Name: <input type="text" name="name">
					</label>
				</li>
				<li>
					<label>
						Email: <input type="text" name="email">
					</label>
				</li>
			</ul>
			<input type="submit" value="submit">
		</div>
	</form>

###script.js
使用ajax提交表單後，如果成功的話，就顯示成功訊息，如果失敗就印出錯誤訊息。  

	var dFormNewsLetter = $('.formNewsLetter');
	
	dFormNewsLetter.on('submit', function(e){
		e.preventDefault();
	
		var dThisForm = $(this),
			action = dThisForm.attr('action'),
			$container = dThisForm.find('.formContainer');
	
		$.ajax({
			url: action,
			type: 'post',
			data: {},
			dataType: 'json',
			error: function (xhr) {
				alert('error: ' + xhr);
			},
			success: function (response) {
				if(response.success){
					$container.html('<h2>Thank You!</h2>');
				}
				else{
					$container.html('There is a problem');
				}
			}
		});
	});

###app.js
這裡有一個判斷式 `req.xhr || req.accepts('json, html') === 'json'` ，意即，如果是使用ajax請求的話，`req.xhr` 與 `req.accepts('json, html') === 'json'` 皆為true，那麼就發送成功訊息 `{ success: true}` ，畫面會直接append一段程式碼 `<h2>Thank You!</h2>` 表示表單傳送成功；如果為false，表示為form post，則轉跳到「thankyou」頁面。

	app.post('/process', function(req, res){
	    if(req.xhr || req.accepts('json, html') === 'json'){
	        res.send({ success: true});
	    }
	    else{
	        res.redirect(303, 'thankyou');
	    }
	});

這段主要是說明 `req.accepts()` 會去判斷哪一種回應的類型是最好的，而我們這邊是設定最佳回傳格式是JSON或HTML。當Server端收到Accept HTTP Header後就會做出判斷。所以，如果是ajax請求，或client side要求JSON或HTML或更好的回應，則回應JSON物件 `{ success: true}` ，否則就轉跳到「thankyou」頁面。

###Demo
填寫表單，填寫完成後送出。 

![Node - 處理Ajax表單](https://lh3.googleusercontent.com/_oOCjVQmDldAeU-oCTEW5oKjwIFyEGZPn0_MbtcMxqE=w580-h146-no)  

送出成功，回應「Thank You!」訊息。

![Node - 處理Ajax表單](https://lh3.googleusercontent.com/pxRZB7gsLvvxWJcbUYAhuBq9zUBWYS14s3bf6Y5jT2Y=w602-h699-no)

![Node - 處理Ajax表單](https://lh3.googleusercontent.com/IziKwfrZAEwhHPpHGZ2ZeuDCBEXN-No5ak5Ixg5Qk1k=w502-h270-no)

---
##檔案上傳
###檔案上傳有以下兩種方法：
- Busboy 或 Formidable。因為Formidable比較簡單，所以在這裡我們選用Formidable。
- FormData，可參考[FormData](https://developer.mozilla.org/en-US/docs/web/api/formdata)。

###安裝Formidable

	npm install --save formidable

###設定與使用Formidable
####HTML Snippet
我們一樣準備一個表單供使用者填寫後提交。因為檔案上傳為binary data (在此為接受所有圖檔)，所以資料編碼使用 `multipart/form-data` 格式。

	<form 
		class="formUpload"
		enctype="multipart/form-data"
		method="POST"
		action="/contest/vacation-photo/{year}/{month}">
		<div class="formContainer">
			<h2>Upload a vacation photo for contest!</h2>
			<input type="hidden" name="_csrf" value="{{ csrf }}">
			<ul>
				<li>
					<label>
						Name: <input type="text" name="name">
					</label>
				</li>
				<li>
					<label>
						File: <input type="file" name="photo" accept="image/*">
					</label>
				</li>
			</ul>
			<input type="submit" value="submit">
		</div>
	</form>

####app.js
處理router的部份。

	var formidable = require('formidable');

	app.get('/contest/vacation-photo', function(req, res){
	    var now = new Date();
	    res.render('cpntest/vacation-photo', {
	        year: now.getFullYear(), month: now.getmonth()
	    });
	});

取得表單後要做解析，若成功則轉至成功畫面，否則轉跳到錯誤畫面。
	
	app.post('/contest/vacation-photo/:year/:month', function(req, res){
	    var form = new formidable.IncomingForm();
	    form.parse(req, function(err, fields, files){
	        if(err){
	            return res.redirect(303, '/error');
	        }
	        console.log('received fields: ');
	        console.log(fields);
	        console.log('received files: ');
	        console.log(files);
	        return res.redirect(303, '/thankyou');
	    });
	});

###Demo
![Node - 檔案上傳](https://lh3.googleusercontent.com/p_juDfpB1_YFu-ZSNR249cj-UK6_8cXKV77RoMWWdTk=w531-h330-no)  

![Node - 檔案上傳](https://lh3.googleusercontent.com/PFTYLjrS_ohCMbNHucBDaN7Hjum99crfgyBUSrANGbE=w637-h421-no)  

###補充說明
- [自學Node.js 五：學習node-formidable](http://blog.csdn.net/lfsfxy9/article/details/8870069)：Formidable的使用和說明。

---
##jQuery檔案上傳
我們可能想用一些比較花俏的介面來做檔案上傳的動作，例如可以拖曳、上傳進度或馬上看到上傳後的縮圖。那麼就可以使用一些jQuery Plugin來完成。

###UI部份
- 檔案上傳使用 [jQuery File Upload](https://blueimp.github.io/jQuery-File-Upload)
- 顯示縮圖使用 [ImageMagic](http://www.imagemagick.org/script/index.php)

###安裝jquery-file-upload-middleware

	npm install --save jquery-file-upload-middleware

###設定與使用jquery-file-upload-middleware
####HTML Snippet
準備一個表單供使用者填寫後提交。這段程式碼和之前的沒有什麼差異 :(

	<form 
		class="formUpload"
		enctype="multipart/form-data"
		method="POST">
		<div class="formContainer">
			<h2>Upload a vacation photo for contest!</h2>
			<label>
				File: <input type="file" id="fieldPhoto" name="photo" accept="image/*" data-url="/upload" multiple>
			</label>
			<div id="uploads"></div>
		</div>
	</form>

####app.js	
處理router的部份。相關文件可以參考[jquery-file-upload-middleware](https://github.com/aguidrevitch/jquery-file-upload-middleware)。

	var jqupload = require('jquery-file-upload-middleware');
	
	app.use('/upload', function(res, req){
	    var now = Date.now();
	    jqupload.fileHandler({
	        uploadDir: function(){
	            return __dirname + '/public/uploads/' + now;
	        },
	        uploadUrl: function(){
	            return '/uploads/' + now;
	        }
	    })(req, res, next);
	});

####script.js
上傳成功後，將上傳成功的檔名列印在畫面上。

	var dfieldPhoto = $('#fieldPhoto');
	dfieldPhoto.fileupload({
		dataType: 'json',
		done: function(e, data){
			$each(data.result.files, function(index, file){
				dfieldPhoto.append($('<div class="upload">File uploaded: </div>' + file.originalName));
			})
		}
	});

---
[Node - 表單處理與檔案上傳 (Form Handling and File Uploads)](http://cythilya.blogspot.tw/2015/08/node-form-handling-and-file-uploads.html)：網誌版。
