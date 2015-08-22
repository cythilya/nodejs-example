#Node - Cookie and Session
Cookie and Session。
##Cookie的特性
- Cookie無法對使用者保密：使用者可以查看所有Server端傳送到Client端的Cookie。
- 使用者可以刪除、停用Cookie：使用者擁有對Cookie的控制權。
- 正規的Cookie可能會被竄改：當Cookie對特定伺服器發出請求，而伺服器信任這請求時，等於門戶洞開，歡迎被攻擊。例如：去執行被竄改後的Cookie。因此，為了確保Cookie不被竄改，建議改用 **Signed Cookie**。
- Cookie可用來進行攻擊：跨站指令攻擊(XSS)其中ㄧ項技術是使用JavaScript惡意修改Cookie的內容，因此不要信任由Client端回傳到Server端的Cookie內容。改善的方法是：
	- 使用Signed Cookie：因為Server端不接受竄改過的Signed Cookie，且會重設回原本的值。
	- 指定Cookie只能由伺服器修改。 
- 使用者知道Cookie被濫用。
- 愛用Session，而非Cookie。Session較Cookie簡單、不浪費使用者的儲存空間、安全。

###補充說明
- [XSS攻擊](http://www.puritys.me/docs-blog/article-78-XSS-%E6%94%BB%E6%93%8A.html)：XSS簡單說明和範例。如果擔心Cookie被Javascript讀取和修改，則使用這段指令 `setCookie("name","value",time()+86400,"/","domain",false,true);` 。 其中最後一個參數是指定HttpOnly，表示Cookie不能夠被Javascript存取，目前主流瀏覽器皆有支援。
- [What are “signed” cookies in connect/expressjs?](http://stackoverflow.com/questions/11897965/what-are-signed-cookies-in-connect-expressjs)：Signed Cookie的簡單說明。

---
##外部認證
將Cookie保密是一個使得Cookie安全的必要方法，我們可以使用[xkcd Password Generator](http://preshing.com/20110811/xkcd-password-generator)或憑證來保障其安全性。這裡選用憑證。  

建立一個javascript檔案 - credential.js，將憑證存放在這支檔案裡面。記得，要加到.gitignore，使其不受版本控管。

###credential.js

	module.exports = {
		cookieSecret: 'your cookie secret goes here'
	};

###app.js

	var credentials = require('./credential.js');

###補充說明
- [xkcd Password Generator](http://preshing.com/20110811/xkcd-password-generator)

---
##Express裡的Cookie
###安裝cookie-parser

	npm install --save cookie-parser

###在app.js中設定載入模組cookie-parser

	var cookieParser = require('cookie-parser');
	
	app.use(cookieParser(credentials.cookieSecret));

###使用cookie-parser
####設定Cookie
假設在取得首頁時，設定兩組Cookie：monster和signed_monster。

	//get index page
	router.get('/', function(req, res, next) {
		//取得各城市的氣象相關資訊
		if(!res.locals.partials){
			res.locals.partials = {};
		}
		res.locals.partials.weather = getWeatherData();
	
		//取得首頁時，設定兩組Cookie：monster和signed_monster
		res.cookie('monster', 'nom nom');
		res.cookie('signed_monster', 'nom nom', { signed: true});
	
		res.render('index', { title: 'Index' });
	});

來看看結果。  

![Node - Cookie and Session](https://lh3.googleusercontent.com/pQBAh_pNlZ44amCZ26NDYYI5Aj4-mj2xBAnTapa3Pys=w776-h414-no)  

####讀取Cookie
將剛剛設定的Cookie從Server端讀取。

	var monsterCookie = req.cookies.monster;
	var signedMonsterCookie = req.signedCookies.signed_monster;

####刪除Cookie
刪除monster這個Cookie。

	res.clearCookie('monster');

####Cookie的屬性
- domain：可將Cookie指定給特定的子網域。
- path：控制Cookie套用的路徑。
- maxAge：指定Client端的Cookie的保留時間，意即在多久後刪除，以毫秒為單位。若忽略則會在瀏覽器關閉後刪除。也可以用expires指定到期時間。
- secure：指定這個Cookie只會透過HTTPS來傳送。
- httpOnly：指定這個Cookie只能被Server端修改。
- signed：若設定為true，則只能在 `res.signedCookies` 中使用，而不能在res.cookies中使用。Server端會拒絕被竄改的Signed Cookie，且重設為原本的值。

---
##Session
使用一個內含獨一無二識別碼的Cookie，Server端會用這個識別碼來取回適當的Session資訊。

###安裝並設定express-session
	
	npm install --save express-session

在app.js中設定載入模組express-session。

    var credentials = require('./credential.js'),
		session = require('express-session');

	app.use(cookieParser(credentials.cookieSecret));
	app.use(session());

###使用Session

	req.session.userName = "Pusheen";
	var colorScheme = req.session.colorScheme || 'dark';

	console.log(req.session.userName); //Pusheen
	console.log(colorScheme); //Dark

	delete req.session.userName; //remove session
	console.log(req.session.userName); //undefined

###補充說明
- [Cookie Session](https://www.npmjs.com/package/cookie-session)：將所有資訊存在Cookie中，而只將非識別碼存在Cookie中，再由識別碼跟Server拿取資料。

---
##使用Session來實作閃爍訊息
於是我們就可以利用Session來實作在[Node - 表單處理與檔案上傳 (Form Handling and File Uploads)](http://cythilya.blogspot.tw/2015/08/node-form-handling-and-file-uploads.html)提到的，當使用者提交表單後，直接在頁面上顯示Flash訊息來提示使用者成功送出資料。

###HTML Snippet

	{{#if flash}}
		<div>{{{flash.message}}}</div>
	{{/if}}

###app.js
提交表單後，設定flash訊息。如果存在flash訊息，則列印在畫面上，然後刪除。

	app.post('/newsletter', function(req, res){
	    var name = req.body.name || '', 
	        email = req.body.email;
	
	    req.session.flash = {
	        type: 'success',
	        message: 'You have been signed up for the newsletter'
	    };    
	
	    return res.redirect(303, '/thankyou');
	});

###index.js
取得thankyou page時列印flash訊息在畫面上。

	//get thankyou page
	router.get('/thankyou', function(req, res) {
		res.locals.flash = req.session.flash;
	    res.render('thankyou', { title: 'thankyou' });
	});

###Demo
![Node - Cookie and Session](https://lh3.googleusercontent.com/Tjg9jgKE-x5ZBZNbJQrp1L6I4nsqVELjhjGGriGszbc=w614-h158-no)

---
####相關閱讀
- [Node - Cookie and Session](http://cythilya.blogspot.tw/2015/08/node-cookie-and-session.html)：網誌版。
- [使用Node.js + Express建構一個簡單的微博網站](http://cythilya.blogspot.tw/2014/11/nodejs-express-microblog.html)
- [Node - 表單處理與檔案上傳 (Form Handling and File Uploads)](http://cythilya.blogspot.tw/2015/08/node-form-handling-and-file-uploads.html)
- [Hello Node - 基本設定和簡單範例](http://cythilya.blogspot.tw/2015/08/hello-node.html)
- [Node - 使用express-partials製作Partial View](http://cythilya.blogspot.tw/2015/08/node-express-partials.html)
- [Node - 使用模版引擎 Handlebars](http://cythilya.blogspot.tw/2015/08/node-handlebars.html)
- [Node - 隱藏Response Headers資訊](http://cythilya.blogspot.tw/2015/08/node-response-headers.html)