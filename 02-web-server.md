Začneme jednoduchým serverem, který na každé stránce zobrazí Hello world. Protože jsme líní a psaní webového serveru je spousta práce, použijeme [Express](http://expressjs.com/), který nám ji usnadní.

Nejprve ve složce s projektem vytvoříme soubor package.json pomocí příkazu `npm init`, stačí jen mačkat <kbd>Enter</kbd>. Pak nainstalujeme Express příkazem `npm install express --save`. Příkaz vytvoří složku node_modules a stáhne do ní Express a další potřebné soubory.

## Začínáme

Napíšeme následující kód do souboru index.js a ve složce s projektem spustíme `node index.js`.

> Když budeme chtít server restartovat, stačí stisknout <kbd><kbd>Ctrl</kbd>-<kbd>C</kbd></kbd> a znovu napsat `node index.js`.

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

Při spuštění serveru soubor načteme funkcí readFileSync. Ve většině případů je lepší metoda readFile, tady použijeme pro jednoduchost synchronní verzi. Když takto načítáme data ze souboru musíme taky říct prohlížeči, že se jedná o HTML.

```javascript
// načteme knihovnu pro práci se soubory
var fs = require('fs');
// přečteme soubor
var sablona = fs.readFileSync(__dirname + '/index.html');

app.get('/', function(req, res) {
	// pošleme prohlížeči hlavičku s typem dokumentu
	res.set('Content-Type', 'text/html');
	// pošleme obsah souboru
	res.send(sablona);
});
```

Teď ale musíme vyřešit, jak doprostřed souboru vložit již napsané zprávy. Jednou z možností by bylo rozdělit soubor na více částí (vršek a spodek), ale pokud bychom tam chtěli vkládat nějakou další informaci (třeba počet příspěvků do tagu `<title>`), počet souborů by se rychle rozrostl, což by bylo asi ještě méně přehledné, než mít HTML v JavaScriptových souborech.

Další možností je vložit si do souboru nějakou značku (placeholder), kterou později nahradíme:

```html
<ul>
{{{zpravy}}}
</ul>
```

```javascript
app.get('/', function(req, res) {
	// pošleme prohlížeči hlavičku s typem dokumentu
	res.set('Content-Type', 'text/html');

	// vytvoříme prázdný řetězec
	var zpravyHtml ='';
	// a postupně k němu přidáme HTML jednotlivých zpráv
	for (var zprava of zpravy) {
		zpravyHtml += '<li title="' + zprava.datum + '"><strong>' + zprava.jmeno + '</strong> ' + zprava.zprava + '</li>';
	}

	// převedeme data ze souboru na řetězec a nahradíme placeholder
	res.send(sablona.toString().replace('{{{zpravy}}}', zpravyHtml));
});
```

Právě jsme z HTML souboru udělali primitivní „šablonu“.

Stále tu ale zůstává trocha HTML kódu, ruční nahrazování taky není zrovna nejlepší. Navíc to není ani moc bezpečné, kdokoliv může odeslat třeba takovouto zprávu `<script>alert('kekeke');</script>`. Proto použijeme šablonovací systém

Příkazem `npm install express-handlebars --save` nainstalujeme [handlebars.js](http://handlebarsjs.com/). Proměnnou `sablona` už nebudeme potřebovat a tak ji nahradíme následujícím kódem.

```javascript
// načteme knihovnu pro práci se šablonami
var hbs  = require('express-handlebars');
// přípona šablon bude hbs a výchozí layout bude main.hbs
app.engine('hbs', hbs({defaultLayout: 'main', extname: '.hbs'}));
// výchozí šablony budou používat handlebars
app.set('view engine', 'hbs');
```

Dále vytvoříme soubor `views/layouts/main.hbs`, který bude obsahovat layout stránky. Pokud neřekneme jinak, naše stránky budou vloženy na místo `{{{body}}}` v této šabloně.

```handlebars
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>{{title}}</title>
</head>
<body>
{{{body}}}
</body>
</html>
```

Samotnou šablonu, která bude reprezentovat hlavní (a zatím jedinou) stránku uložíme do `views/zpravy.hbs`. `{{#each}}` je vlastně cyklus a pro každou položku seznamu se vytvoří proměnné odpovídající jednotlivým atributům objektu. `{{promenna}}` pak vykreslí proměnnou s ošetřenými HTML tagy.

```handlebars
<form action="" method="post">
<input type="text" name="jmeno">
<textarea name="zprava"></textarea>
<button type="submit">Odeslat</button>
</form>

<ul>
{{#each zpravy}}
<li title="{{datum}}"><strong>{{jmeno}}</strong> {{zprava}}</li>
{{/each}}
</ul>
```

Samotné vykreslení šablony už je snadné:

```javascript
app.get('/', function(req, res) {
	// pošleme prohlížeči hlavičku s typem dokumentu
	res.set('Content-Type', 'text/html');

	// vykreslíme šablonu zpravy.hbs a předáme jí nějaké proměnné
	res.render('zpravy', {title: 'Návštěvní kniha', zpravy: zpravy});
});
```

No není to pěkné?

## Statické soubory
Poslední věc, která by se mohla hodit je odesílání souborů jako jsou obrázky nebo [externí CSS](https://www.khanacademy.org/computing/computer-programming/html-css/more-ways-to-embed-css/p/using-external-stylesheets).

I zde se nabízí několik možností, nejjednodušší z nich je pro každý soubor napsat vlastní GET handler, který přímo přečte odpovídající soubor, podobně, jako jsme to dělali prvně u HTML souborů.

```javascript
var fs = require('fs');
app.get('/files/style.css', function(req, res) {
	res.set('Content-Type', 'text/css');
	var data = fs.readFileSync(__dirname +'/files/style.css');
	res.send(data);
});
```

Pokud bychom pracovali s mnoha desítkami souborů, nebylo by to moc pohodlné. Navíc tu ani neřešíme některé pokročilejší věci jako cacheování, takže by se soubory musely načítat pokaždé znovu, i když by se třeba ani nezměnily. Protože tenhle problém se týká naprosté většiny aplikací, má express zabudovaný middleware [static](http://expressjs.com/starter/static-files.html), který to řeší.

```javascript
// pro adresy začínající na /files použij middleware static
app.use('/files', express.static(__dirname + '/files'));
```

Middleware static se vždy koukne, jestli požadovaný soubor ve složce existuje a pokud ano, odešle jeho obsah spolu se správným typem a dalšími rozumnými HTTP hlavičkami.
