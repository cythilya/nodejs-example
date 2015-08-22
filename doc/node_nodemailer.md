#Node - 使用Nodemailer傳送Email
使用Nodemailer來傳送Email。文件可參考[Nodemailer](https://github.com/andris9/Nodemailer)。
###安裝與設定nodemailer

	npm install --save nodemailer

在app.js載入模組並設定使用的服務、帳號、密碼。這裡使用方便的Gmail。

	var nodemailer = require('nodemailer');

	var mailTransport = nodemailer.createTransport('SMTP', {
	    service: 'Gmail',
	    auth: {
	        user: credentials.gmail.user,
	        pass: credentials.gmail.password
	    }
	});

或需要直接連到SMTP伺服器，也是可以的。  

	var mailTransport = nodemailer.createTransport('SMTP', {
	    host: 'smtp.mailservice.com',
	    secureConnecton: true,
	    port: 587,
	    auth: {
	        user: credentials.mailservice.user,
	        pass: credentials.mailservice.pass
	    }
	});

更新credentials.js檔案。我們將帳號與密碼這種敏感資訊放在credentials裡面。

	module.exports = {
		cookieSecret: 'your cookie secret goes here',
		gmail: {
	        user: 'user name',
	        pass: 'password here...'		
		}
	};

然後下指令 `npm start` 就可以傳送Email了。  

###Demo
收到信了～  

![Node - 使用Nodemailer傳送Email](https://lh3.googleusercontent.com/RQK8pHWccd6KhleNGvlb1M71GthK2KL1bja7G7kEqs4=w534-h153-no)  

---
##傳送HTML Email
我們使用html欄位裝載HTML Email。  

	mailTransport.sendMail({
	    from: 'Meadowlark Travel <cythilya@gmail.com>',
	    to: 'Cythilya <cythilya@gmail.com>',
	    subject: 'Hi :)',
	    html: '<h1>Hello</h1><p>Nice to meet you.</p>'
	}, function(err){
	    if(err){
	        console.log('Unable to send email: ' + err);
	    }
	});

###Demo
![Node - 使用Nodemailer傳送Email](https://lh3.googleusercontent.com/5UZ9nHhoARGX4h1WQmcYTKuRbWZxP0ahTIBoaAvf18s=w348-h221-no)

---
##使用View來傳送HTML Email
###準備一個HTML樣版 - cart-thank-you.handlebars
我們將電子報的內容製成樣版，之後再從Server端取資料套版。

	<body>
		<h1>Thank You!</h1>
		<p>Hi, {{ cart.name }}</p>
		<p>Thank you for booking your trip with Meadowlark Travel.</p>
	</body>

###送出信件
送出表單後，會寄一封信到信箱。(備註：這邊都是做範例練習用的，所以不要計較為何使用訂閱電子報的表單，卻收到購物車相關的信件 XD)

	var mailTransport = nodemailer.createTransport({
	    service: 'Gmail',
	    auth: {
	        user: credentials.gmail.user,
	        pass: credentials.gmail.pass
	    }
	});
	
	app.post('/newsletter', function(req, res){
	    var name = req.body.name || '', 
	        email = req.body.email;
	
	    var cart = {
	        name: name,
	        email: email,
	    };
	
	    res.render('email/cart-thank-you', {
	        layout: null,
	        cart: cart
	    }, function(err, html) {
	        if(err){
	            console.log('error in email template');    
	        } 
	        mailTransport.sendMail({
	            from: '"Meadowlark Travel": cythilya@gmail.com',
	            to: email,
	            subject: 'Thank You for Book your Trip with Meadowlark',
	            html: html
	        }, function(err) {
	            if(err){
	                console.error('Unable to send confirmation: ' + err.stack)
	            };
	        });
	    });
	    res.render('cart-thank-you', {
	        cart: cart
	    });
	
	    return res.redirect(303, '/thankyou');
	});

###Demo
![Node - 使用Nodemailer傳送Email](https://lh3.googleusercontent.com/109DyECt2D_ch9KgD6h5agm9qtkZlg0DyEiRfr8FczQ=w800-h443-no)

---
##封裝Email功能
###封裝成一支javascript檔案 - email.js

	var nodemailer = require('nodemailer');
	module.exports = function(credentials) {
	    var mailTransport = nodemailer.createTransport('SMTP', {
	        service: 'Gmail',
		    auth: {
		        user: credentials.gmail.user,
		        pass: credentials.gmail.pass
		    }
	    });
	    var from = '"Meadowlark Travel" <cythilya@gmail.com>';
	    var errorRecipient = 'cythilya@gmail.com';
	    return {
	        send: function(to, subj, body) {
				mailTransport.sendMail({
				    from: 'Meadowlark Travel <cythilya@gmail.com>',
				    to: 'Cythilya <cythilya@gmail.com>',
				    subject: 'Hi :)',
				    html: '<h1>Hello</h1><p>Nice to meet you.</p>'
				}, function(err){
				    if(err){
				        console.log('Unable to send email: ' + err);
				    }
				});
	        },
	    	emailError: function(message, filename, exception) {
	        	var body = '<h1>Meadowlark Travel Site Error</h1>' + 'message:<br><pre>' + message + '</pre><br>';
	        	if (exception) body += 'exception:<br><pre>' + exception + '</pre><br>';
	        	if (filename) body += 'filename:<br><pre>' + filename + '</pre><br>';
	        	mailTransport.sendMail({
	            	from: from,
	            	to: errorRecipient,
	            	subject: 'Meadowlark Travel Site Error',
	            	html: body,
	            	generateTextFromHtml: true
				}, function(err){
				    if(err){
				        console.log('Unable to send email: ' + err);
				    }
				});
			}
	    }
	};

###使用這個封裝檔案

	var emailService = require('./lib/email.js')(credentials);
	
	emailService.send('cythilya@gmail.com', 'Hood River tours on sale today!', 'Get \'em while they\'re hot!');

---
##把Email當成網站監控工具
假設網站有任何問題，我們都可以寄一封信來通知...

	if (err) {
	    email.sendError('the widget broke down!', __filename);
	    // ... display error message to user
	}

或...
	
	try {
	    // do something iffy here....
	} 
	catch (ex) {
	    email.sendError('the widget broke down!', __filename, ex);
	    // ... display error message to user
	}

但這不是代替log的方法，之後會再討論相關議題。

---
####相關閱讀
- [Node - 使用Nodemailer傳送Email](http://cythilya.blogspot.tw/2015/08/node-nodemailer.html)：網誌版。
- [Nodemailer](https://github.com/andris9/Nodemailer)
- [郵件伺服器架構概觀](http://www-01.ibm.com/support/knowledgecenter/linuxonibm/liaaz/mailflow.htm?lang=zh-tw) 這篇文章對於一封電子郵件從送出到接收流程有完整的說明。也可以看這篇 [How email works (MTA, MDA, MUA)](http://ccm.net/contents/116-how-email-works-mta-mda-mua) 。
- [Email標頭、格式相關](http://ithelp.ithome.com.tw/question/10054360?tag=rt.rq)。
- [Node.js 系列學習日誌 #21 - 使用 nodemailer 套件透過 gmail 發送電子信箱](http://ithelp.ithome.com.tw/question/10160766)