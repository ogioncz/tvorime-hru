Začneme jednoduchým serverem, který na každé stránce zobrazí Hello world. Protože jsme líní a psaní webového serveru je spousta práce, použijeme Express, který nám ji usnadní.

Nejprve ve složce s projektem vytvoříme soubor package.json pomocí příkazu `npm init`, stačí jen mačkat <kbd>Enter</kbd>. Pak nainstalujeme Express příkazem `npm install express --save`. Příkaz vytvoří složku node_modules a stáhne do ní Express a dalšé potřebné soubory.

Napíšeme následující kód do souboru index.js a ve složce s projektem spustíme `node index.js`.

```javascript
// Načteme knihovnu express
var express = require('express');
// vytvoříme aplikaci založenou na expressu
var app = express();

// nastavíme port
const PORT = 8080;

// GET /
app.get('/', function(req, res) {
	res.send('Hello world');
});

// Spustíme server na daném PORTu
app.listen(PORT, function() {
	console.log('Server běží na adrese http://localhost:' + PORT);
});
```

Mít webový server, který pořád jen vypisuje ten stejný text je trochu nudné, zkusme tedy vytvořit jednoduchou návštěvní knihu.

Jako první vytvoříme odesílací formulář.

```javascript
app.get('/', function(req, res) {
	res.send(`
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Návštěvní kniha</title>
</head>
<body>
<form action="" method="post">
<input type="text" name="jmeno">
<textarea name="zprava"></textarea>
<button type="submit">Odeslat</button>
</form>
</body>
</html>
`);
});
```

> V JavaScriptu existuje několik způsobů zápisu textového řetězce. Lze použít jednoduché uvozovky (apostrofy) `'text'`, dvojité uvozovky `"text"` nebo *backtick*y („zpětné apostrofy“) `` `text` ``. Zpětné apostrofy jsou v JavaScriptu celkem nové (ale [moderní prohlížeče](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings#Browser_compatibility) už je podporují) a mají tu výhodu, že můžou obsahovat i víceřádkový text.

Samozřejmě budeme potřebovat odeslaná data taky zpracovávat, na což budeme potřebovat *bodyParser*, nainstalujeme ho příkazem `npm install body-parser --save`.

```javascript
// Načteme knihovnu express
var express = require('express');
// vytvoříme aplikaci založenou na expressu
var app = express();

var bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({extended: false}));

var zpravy = [];

// nastavíme port
const PORT = 8080;

// GET /
app.get('/', function(req, res) {
	res.write(`
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Návštěvní kniha</title>
</head>
<body>
<form action="" method="post">
<input type="text" name="jmeno">
<textarea name="zprava"></textarea>
<button type="submit">Odeslat</button>
</form>

<ul>
`);
	for (var zprava of zpravy) {
		res.write(`<li title="${zprava.datum}"><strong>${zprava.jmeno}</strong> ${zprava.zprava}</li>`);
	}
	res.write(`
</ul>
</body>
</html>
`);
	res.end();
});

app.post('/', function(req, res) {
	if (!req.body.jmeno) {
		res.send('Zadej jméno');
	} else if (!req.body.zprava) {
		res.send('Zadej zprávu');
	} else {
		zpravy.push({jmeno: req.body.jmeno, zprava: req.body.zprava, datum: new Date()});
		res.redirect('/');
	}
});

// Spustíme server na daném PORTu
app.listen(PORT, function() {
	console.log('Server běží na adrese http://localhost:' + PORT);
});
```
