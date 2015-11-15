Začneme jednoduchým serverem, který na každé stránce zobrazí Hello world. Napíšeme následující kód do souboru index.js a ve složce s projektem spustíme `node index.js`.

```javascript
// Načteme knihovnu http
var http = require('http');

// nastavíme port
const PORT = 8080;


// funkce, která se zavolá pokaždé, když se prohlížeč serveru na něco zeptá
var odpoved = function(request, response){
	response.end('Hello world');
}

// Vytvoříme server
var server = http.createServer(odpoved);

// Spustíme server na daném PORTu
server.listen(PORT, function(){
	console.log('Server běží na adrese http://localhost:%s', PORT);
});
```
