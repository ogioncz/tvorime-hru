Začneme jednoduchým serverem, který na každé stránce zobrazí Hello world. Protože jsme líní a psaní webového serveru je spousta práce, použijeme [Express](http://expressjs.com/), který nám ji usnadní.

Nejprve ve složce s projektem vytvoříme soubor package.json pomocí příkazu `npm init`, stačí jen mačkat <kbd>Enter</kbd>. Pak nainstalujeme Express příkazem `npm install express --save`. Příkaz vytvoří složku node_modules a stáhne do ní Express a další potřebné soubory.

## Začínáme

Napíšeme následující kód do souboru index.js a ve složce s projektem spustíme `node index.js`.

```javascript
// Načteme knihovnu express
var express = require('express');
// vytvoříme aplikaci založenou na expressu
var app = express();

// zvolíme port
const PORT = 8080;

// když server obdrží požadavek `GET /`
app.get('/', function(req, res) {
	res.send('Hello world');
});

// Spustíme server na daném PORTu
app.listen(PORT, function() {
	console.log('Server běží na adrese http://localhost:' + PORT);
});
```

> Když prohlížeč něco po serveru chce, první, co mu řekne, bude dvojice `METODA adresa`. Metod je několik, nejčastěji se používají `GET` a `POST`, přičemž `GET` je výchozí, `POST` obvykle používají formuláře. Server se podle toho, jakou metodu a adresu obdrží rozhodne, co dál udělá. Podrobnější  vysvětlení je třeba na [wikipedii](https://cs.wikipedia.org/wiki/Hypertext_Transfer_Protocol#.C4.8Cinnost_protokolu).

## Formuláře

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

> V JavaScriptu existuje několik způsobů zápisu textového řetězce. Lze použít jednoduché uvozovky (apostrofy) `'text'`, dvojité uvozovky `"text"` nebo *backtick*y `` `text` ``. Zpětné apostrofy jsou v JavaScriptu [celkem nové](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings#Browser_compatibility), a mají tu výhodu, že můžou obsahovat i víceřádkový text.

Samozřejmě budeme potřebovat odeslaná data taky zpracovávat, na což budeme potřebovat *[body-parser](https://www.npmjs.com/package/body-parser)*. Nainstalujeme ho příkazem `npm install body-parser --save`.

```javascript
// Načteme knihovnu express
var express = require('express');
// vytvoříme aplikaci založenou na expressu
var app = express();

// potřebné pro zpracování odeslaných formulářů
var bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({extended: false}));

// sem budeme ukládat zprávy
var zpravy = [];

// zvolíme port
const PORT = 8080;

// když server obdrží požadavek `GET /`
app.get('/', function(req, res) {
	// vypíšeme začátek stránky s formulářem
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

	// postupně vypíšeme všechny zprávy
	for (var zprava of zpravy) {
		res.write('<li title="' + zprava.datum + '"><strong>' + zprava.jmeno + '</strong> ' + zprava.zprava + '</li>');
	}
	res.write(`
</ul>
</body>
</html>
`);
	// už je to všechno :-)
	res.end();
});

// když server obdrží požadavek `POST /`
app.post('/', function(req, res) {
	if (!req.body.jmeno) { // není zadané jméno
		res.send('Zadej jméno');
	} else if (!req.body.zprava) { // není zadaná zpráva
		res.send('Zadej zprávu');
	} else { // oboje je vyplněné
		// přidáme do seznamu zpráv novou zprávu
		zpravy.push({jmeno: req.body.jmeno, zprava: req.body.zprava, datum: new Date()});
		// přesměrujeme zpět na úvodní stránku
		res.redirect('/');
	}
});

// Spustíme server na daném PORTu
app.listen(PORT, function() {
	console.log('Server běží na adrese http://localhost:' + PORT);
});
```

> Krátký přehled metod:
>
> * res.write(text) – pošle prohlížeči text
> * res.end() – ukončí komunikaci s prohlížečem
> * res.send(text) – nejdřív pošle text a pak ukončí komunikaci
> * res.redirect(adresa) – řekne prohlížeči, že má přejít na jinou adresu, **předtím nesmí být poslán žádný text**
>
> Více informací v [dokumentaci](http://expressjs.com/4x/api.html#res)

## Šablony
Mít HTML kód v JavaScriptových souborech není zrovna nejpřehlednější, proto obvykle používáme samostatné soubory. Vytvořme tedy soubor `index.html` s následujícím kódem.

```html
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
</ul>
</body>
</html>
```

Při spuštění serveru soubor načteme funkcí readFileSync. Ve většině případů je lepší metoda readFile, tady použijeme pro jednoduchost synchronní verzi. Když takto načíme data ze souboru musíme taky říct prohlížeči, že se jedná o HTML.

```javascript
// načteme knihovnu pro práci se soubory
var fs = require('fs');
// přečteme soubor
var sablona = fs.readFileSync('index.html');

app.get('/', function(req, res) {
	// pošleme prohlížeči hlavičku s typem dokumentu
	res.set('Content-Type', 'text/html');
	// pošleme obsah souboru
	res.send(sablona);
});
```
