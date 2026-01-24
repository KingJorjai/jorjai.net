---
title: Creando una API para los buses de mi ciudad
authors: jorjai
tags: [programación, javascript, nodejs, puppeteer, api, web scraping]
---

import dbusConsultarTiemposVideo from './assets/dbus-consultar-tiempos.webm';

¡Hola de nuevo a todxs! Durante esta semana me he hecho una lista en un Post-it con algunas ideas sobre los temas que me gustaría tratar en próximas entradas del blog. Ahora que ha llegado el sábado, por fin tengo algo de tiempo para sentarme a escribir (técnicamente llevo todo el día sentado, solo que he estado leyendo más que escribiendo 😅).

Hoy me gustaría hablar sobre un proyecto personal que comencé en febrero del año pasado (2025) a raíz de una frustración: los **buses** que tengo que coger para volver a casa desde la universidad **nunca llegan a la hora**. Uno nunca sabe si va a llegar 5 minutos antes o 20 minutos después de lo previsto.

<!-- truncate -->

Pero, ¡no temáis! Porque el ayuntamiento ofrece una **aplicación para el móvil** que da los tiempos de llegada **en tiempo real**. *"Entonces, ¿cuál es el problema?"*, os preguntaréis. Pues en realidad, ninguno. Solo que todos los días al final de la última clase, tengo que sacar el móvil, abrir la app, buscar la parada, y mirar cuánto le queda a mi bus. Dependiendo de lo que ponga, sé si tengo que correr o no. Y yo no sé vosotrxs, pero si estudio ingeniería es para que mi vida sea eficiente. Y no os voy a mentir, me da mucha pereza tener que hacer todo eso.

Así que pensé: *"No sería increíble que me llegara un mensaje al móvil **justo** cuando mi bus esté a 7 minutos de llegar?"*.

## El plan

Como buen ingeniero, lo primero que hice fue **dividir el problema en partes más pequeñas**:

1. **Obtener los datos** de llegada de los buses en tiempo real
2. Crear una **API** que me permita consultar esos datos fácilmente
3. Hacer que me llegue una **notificación al móvil** cuando el bus esté a 7 minutos

Os voy avisando de que esta entrada va a ser un poco larga, porque me gustaría explicar con detalle cómo conseguí hacer cada una de estas partes. Así que sin más dilación, ¡empecemos!

## Primera fase: Obtener los datos

Honestamente, esta parte fue la más divertida de todas. Allá vamos.

El servicio de buses también publica los tiempos de llegada en [su página web](https://dbus.eus/es/). Desde la página principal, seguimos los siguientes pasos:
1. **Elegir la línea desde el desplegable**. Esto nos lleva a la página especifica de la línea.
2. En la página de la línea, **elegir la parada desde otro desplegable**. Esto nos lleva a la página de la parada.
3. En la página de la parada, podemos ver los tiempos de llegada de **cualquier línea** que pase por esa parada. (Este es un detalle importante, ya que veremos más líneas además de la que nos interesa).

<video src={dbusConsultarTiemposVideo} autoPlay muted loop controls style={{maxWidth: '100%', height: 'auto'}} />

### Obteniendo las líneas

El desplegable de selección de línea muestra una lista con todas las líneas de bus disponibles. Así que podemos parsear el HTML de la página principal para obtener esa lista.

![Inspeccionando el código HTML de la página](./assets/select-desplegable-lineas.png)

Sabemos que el desplegable tiene el atributo `id="desplegable-lineas"`, así que podemos buscar ese elemento en el DOM para obtener todas las opciones disponibles.

```html
<select id="desplegable-lineas" name="lineas">
```

*¿Que qué es el DOM?* Pues el Document Object Model, una representación estructurada del documento HTML que nos permite interactuar con él mediante JavaScript. No te preocupes, yo tampoco lo sabía en ese momento. De hecho, fue la primera vez que usé JavaScript en serio. Cuando veamos la implementación, nos iremos familiarizando con los términos.

Si nos metemos al código fuente de la página, encontramos la siguiente función:

```javascript
function get_lineas_front(){
    lineas_front=JSON.parse('[{"texto":"05 | Benta Berri","valor":"1","enlace":"https:\/\/dbus.eus\/05-benta-berri\/"},{"texto":"08 | Gros-Intxaurrondo","valor":"7","enlace":"https:\/\/dbus.eus\/08-gros-intxaurrondo\/"},{"texto":"09 | Egia-Intxaurrondo","valor":"18","enlace":"https:\/\/dbus.eus\/09-egia-intxaurrondo\/"},{"texto":"13 | Altza ","valor":"19","enlace":"https:\/\/dbus.eus\/13-altza\/"},{"texto":"14 | Bidebieta","valor":"20","enlace":"https:\/\/dbus.eus\/14-bidebieta\/"},{"texto":"16 | Igeldo","valor":"10","enlace":"https:\/\/dbus.eus\/16-igeldo\/"},{"texto":"17 | Gros-Amara-Miramon","valor":"22","enlace":"https:\/\/dbus.eus\/17-gros-amara-miramon\/"},{"texto":"18 | Seminarioa","valor":"11","enlace":"https:\/\/dbus.eus\/18-seminarioa\/"},{"texto":"19 | Aiete-Bera Bera","valor":"12","enlace":"https:\/\/dbus.eus\/19-aiete-bera-bera\/"},{"texto":"21 | Amara-Mutualitateak","valor":"13","enlace":"https:\/\/dbus.eus\/21-amara-mutualitateak\/"},{"texto":"23 | Errondo-Puio","valor":"14","enlace":"https:\/\/dbus.eus\/23-errondo-puio\/"},{"texto":"24 | Altza-Gros-Antiguo-Intxaurrondo","valor":"23","enlace":"https:\/\/dbus.eus\/24-altza-gros-antiguo-intxaurrondo\/"},{"texto":"25 | BentaBerri-A\u00f1orga","valor":"2","enlace":"https:\/\/dbus.eus\/25-bentaberri-anorga\/"},{"texto":"26 | Amara-Martutene","valor":"15","enlace":"https:\/\/dbus.eus\/26-amara-martutene\/"},{"texto":"27 | Altza-Intxaurrondo-Antiguo-Gros","valor":"24","enlace":"https:\/\/dbus.eus\/27-altza-intxaurrondo-antiguo-gros\/"},{"texto":"28 | Amara-Ospitaleak","valor":"16","enlace":"https:\/\/dbus.eus\/28-amara-ospitaleak\/"},{"texto":"29 | Intxaurrondo Sur","valor":"21","enlace":"https:\/\/dbus.eus\/29-intxaurrondo-sur\/"},{"texto":"31 | Intxaurrondo-Ospitaleak-Altza","valor":"25","enlace":"https:\/\/dbus.eus\/31-intxaurrondo-ospitaleak-altza\/"},{"texto":"32 | Puio-Errondo","valor":"17","enlace":"https:\/\/dbus.eus\/32-puio-errondo\/"},{"texto":"33 | Larratxo-Intxaur-Berio-Igara","valor":"26","enlace":"https:\/\/dbus.eus\/33-larratxo-intxaurrondo-berio-igara\/"},{"texto":"35 | Antiguo-Aiete-Ospitaleak","valor":"27","enlace":"https:\/\/dbus.eus\/35-antiguo-aiete-ospitaleak\/"},{"texto":"36 | Aldakonea-San Roke","valor":"30","enlace":"https:\/\/dbus.eus\/36-aldakonea-san-roke\/"},{"texto":"37 | Rodil-Zorroaga","valor":"39","enlace":"https:\/\/dbus.eus\/37-rodil-zorroaga\/"},{"texto":"38 | Trintxerpe-Altza-Molinao","valor":"40","enlace":"https:\/\/dbus.eus\/38-trintxerpe-altza-molinao\/"},{"texto":"39 | Urgull","valor":"41","enlace":"https:\/\/dbus.eus\/39-urgull\/"},{"texto":"40 | Gros-Antiguo-Igara","valor":"28","enlace":"https:\/\/dbus.eus\/40-gros-antiguo-igara\/"},{"texto":"41 | Gros-Egia-Martutene","valor":"29","enlace":"https:\/\/dbus.eus\/41-gros-egia-martutene\/"},{"texto":"42 | Aldapa-Egia","valor":"54","enlace":"https:\/\/dbus.eus\/42-aldapa-egia\/"},{"texto":"43 |\u00a0Anoeta-Igara","valor":"43","enlace":"https:\/\/dbus.eus\/43-anoeta-igara\/"},{"texto":"45 | Estaciones Renfe-Bus Geltokiak-Antiguo-Aiete","valor":"56","enlace":"https:\/\/dbus.eus\/45-estaciones-renfe-bus-geltokiak-antiguo-aiete\/"},{"texto":"46 | San Antonio - Morlans","valor":"59","enlace":"https:\/\/dbus.eus\/46-san-antonio-morlans\/"},{"texto":"B1 | Benta Berri-Berio-A\u00f1orga","valor":"31","enlace":"https:\/\/dbus.eus\/b1-benta-berri-berio-anorga\/"},{"texto":"B2 | Aiete-Bera Bera","valor":"32","enlace":"https:\/\/dbus.eus\/b2-aiete-bera-bera\/"},{"texto":"B3 | Egia-Intxaurrondo","valor":"33","enlace":"https:\/\/dbus.eus\/b3-egia-intxaurrondo\/"},{"texto":"B4 | Amara-Riberas-Martutene","valor":"34","enlace":"https:\/\/dbus.eus\/b4-amara-riberas-martutene\/"},{"texto":"B6 | Altza","valor":"35","enlace":"https:\/\/dbus.eus\/b6-altza\/"},{"texto":"B7 | Igeldo","valor":"36","enlace":"https:\/\/dbus.eus\/b7-igeldo\/"},{"texto":"B8 | Miraconcha-BentaBerri-Seminario","valor":"37","enlace":"https:\/\/dbus.eus\/b8-miraconcha-bentaberri-seminario\/"},{"texto":"B9 | Amara-Errondo-Puio","valor":"38","enlace":"https:\/\/dbus.eus\/b9-amara-errondo-puio\/"},{"texto":"B10 | Zubiaurre-Bidebieta-Buenavista","valor":"42","enlace":"https:\/\/dbus.eus\/b10-zubiaurre-bidebieta-buenavista\/"},{"texto":"Gautxorien aurreko zerbitzuak","valor":"52","enlace":"https:\/\/dbus.eus\/pre-buhos-2\/"},{"texto":"TB6 |\u00a0Ulia Taxibusa","valor":"47","enlace":"https:\/\/dbus.eus\/tb6-taxibus-ulia\/"},{"texto":"TB7 | Atotxaerreka - Itsasargi Pasealeku Taxibusa","valor":"62","enlace":"https:\/\/dbus.eus\/tb7-taxibus-atotxaerreka-paseo-del-faro-2\/"},{"texto":"Zerbitzu berezia Anoetara (Futbola)","valor":"48","enlace":"https:\/\/www.dbus.eus\/descargas\/futbol-basket\/servicios-especiales-anoeta-futbol_eu.pdf"},{"texto":"Illunbe \u2013 erdialdea anezka zerbitzua","valor":"65","enlace":"https:\/\/dbus.eus\/illunbe-erdialdea-anezka-zerbitzua\/"},{"texto":"Linea aldaketen taula","valor":"51","enlace":"https:\/\/www.dbus.eus\/descargas\/transbordos.pdf "}]');

    var $desplegable_lineas=jQuery("#desplegable-lineas");
    jQuery.each(lineas_front, function(k,v) {
        jQuery("<option />", {value: v.valor, text: v.texto,enlace:v.enlace}).appendTo($desplegable_lineas);
    });
}
```

Aquí vemos que la función `get_lineas_front` crea un array de objetos JSON llamado `lineas_front`, donde cada objeto representa una línea de bus con su texto, valor y enlace. Luego, utiliza jQuery para seleccionar el elemento del DOM con el id `desplegable-lineas` y añade cada línea como una opción en el desplegable.

Jamás he visto semejante forma de definir datos en mi vida. Pero bueno, funciona. Como nosotros somos un poco más elegantes, vamos a encontrar esa línea usando regex y a extraer el array JSON directamente.

```javascript
async function getBusLines() {
    const response = await fetch('https://dbus.eus/es/');

    if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const html = await response.text();

    // Extraer el array JSON de la función get_lineas_front
    const match = html.match(/lineas_front=JSON\.parse\('(.+?)'\);/);
    if (!match) {
        throw new Error('Could not find lineas_front data in HTML');
    }

    // Reemplazar caracteres escapados y parsear el JSON
    const jsonString = match[1].replace(/\\\//g, '/').replace(/\\"/g, '"');
    const linesData = JSON.parse(jsonString);

    const options = linesData
        .filter(item => item.texto && item.texto.includes('|'))
        .map(item => {
            // Manejar tanto espacios regulares como espacios no separables
            // después de la barra vertical
            const [code, name] = item.texto.split(/\s*\|\s*/);
            return {
                code: code ? code.trim() : '',
                name: name ? name.trim() : '',
                url: item.enlace,
                internal_id: item.valor
            };
        });

    return options;
}
```

¿Qué hace exactamente este código? Vamos paso a paso:

1. **Obtener el HTML**: Primero hacemos una petición HTTP a la página principal de dbus.eus usando `fetch()`. Si la respuesta no es exitosa (código HTTP distinto de 2xx), lanzamos un error.

2. **Buscar el patrón con Regex**: Aquí viene la parte interesante. Usamos una expresión regular para buscar en todo el HTML la línea donde se define `lineas_front=JSON.parse('...')`. El patrón `/lineas_front=JSON\.parse\('(.+?)'\);/` captura todo lo que está entre las comillas simples. El `(.+?)` es un grupo de captura que obtiene el contenido del string JSON.

3. **Limpiar los caracteres escapados**: Como el JSON viene escapado dentro de un string de JavaScript (con `\/` en lugar de `/` y `\"` en lugar de `"`), necesitamos reemplazar estos caracteres para obtener JSON válido:
   - `replace(/\\\//g, '/')` convierte `\/` en `/`
   - `replace(/\\"/g, '"')` convierte `\"` en `"`

4. **Procesar los datos**: Una vez tenemos el JSON parseado, lo transformamos para que sea más fácil de usar:
   - `.filter()` filtra solo las líneas que tienen el formato "código | nombre" (descartando servicios especiales)
   - `.split(/\s*\|\s*/)` separa el código del nombre usando la barra vertical como delimitador, ignorando espacios
   - `.map()` transforma cada elemento en un objeto más limpio con propiedades descriptivas

El resultado es un array limpio y estructurado con todas las líneas de bus:

```json
[
  {
    "code": "05",
    "name": "Benta Berri",
    "url": "https://dbus.eus/05-benta-berri/",
    "internal_id": "1"
  },
  {
    "code": "08",
    "name": "Gros-Intxaurrondo",
    "url": "https://dbus.eus/08-gros-intxaurrondo/",
    "internal_id": "7"
  },
  ...
]
```

¡Y listo! Ya tenemos la lista de líneas de bus. En la siguiente sección, veremos cómo obtener las paradas para una línea específica.

### Obteniendo las paradas

Ahora que tenemos la lista de líneas, el siguiente paso es obtener las paradas de una línea específica. Por ejemplo, si quiero saber qué paradas tiene la línea 05, necesito ir a su página específica.

Recordemos que cada línea tiene una URL propia (como https://dbus.eus/05-benta-berri/). Si accedemos a esa página, encontramos otro desplegable similar al anterior, pero esta vez con las paradas de esa línea.

Esta vez lo tenemos que hacer un poco diferente, porque la página de la línea carga las paradas dinámicamente con JavaScript. No están directamente en el HTML inicial como antes. Analizando el código fuente, vemos que las paradas se cargan en un `<select>` con el id `select_paradas_1`.

![Inspeccionando el código HTML de la página](./assets/select_paradas_1.png)

Echemos un vistazo al código fuente para entender cómo funciona. Resulta que las paradas no se generan de la misma manera que las líneas. En lugar de estar hardcodeadas en el HTML, se cargan desde un archivo XML mediante una petición AJAX:

```javascript
jQuery.ajax({
  url: '//dbus.eus/wp-content/uploads/wp-google-maps/3markers.xml?u='+UniqueCode,
  async: false,
  type: "GET",
  success: function(data) {
    jQuery(data).find("marker").each(function(){
      // Crear opción en el select por cada parada
      jQuery("<option />", {
        value: jQuery(this).find('parada_id').text(), 
        text: jQuery(this).find('title_eu').text()
      }).appendTo(s);
    });
  }
});
```

¡Interesante! Aquí vemos que se hace una petición AJAX a un archivo XML que contiene todas las paradas. Luego, por cada `<marker>` en el XML, se crea una opción en el desplegable con el id y el nombre de la parada. Dejo a continuación el primer marker del XML como ejemplo:

```xml
<marker>
    <marker_id>2</marker_id>
    <map_id>3</map_id>
    <title_es>34 | Boulevard 17</title_es>
    <title_eu>34 | Boulevard 17</title_eu>
    <title_en>34 | Boulevard 17</title_en>
    <title_fr>34 | Boulevard 17</title_fr>
    <parada_id>1587</parada_id>
    <pto_turistico>0</pto_turistico>
    <address>43.32214974,-1.983930094</address>
    <desc_es>&lt;p&gt;Asociada a las l&#xED;neas &lt;a title="05 | Benta Berri" href="https://www.dbus.eus/es/05-benta-berri/"&gt;05&lt;/a&gt;, &lt;a title="25 | BentaBerri-A&#xF1;orga" href="https://www.dbus.eus/es/25-bentaberri-anorga/"&gt;25&lt;/a&gt;, &lt;a title="B1 | Benta Berri-Berio-A&#xF1;orga" href="https://www.dbus.eus/es/b1-benta-berri-berio-anorga/"&gt;B1&lt;/a&gt;, &lt;a title="B8 | Miraconcha-Benta Berri-Seminario" href="https://www.dbus.eus/es/b8-miraconcha-benta-berri-seminario/"&gt;B8&lt;/a&gt;&lt;/p&gt;</desc_es>
    <desc_eu>&lt;p&gt;&lt;b&gt;&lt;/b&gt;Linea hauekin erlazionatuta &lt;a title="05 | Benta Berri" href="https://www.dbus.eus/05-benta-berri/"&gt;05&lt;/a&gt;, &lt;a title="25 | BentaBerri-A&#xF1;orga" href="https://www.dbus.eus/25-bentaberri-anorga/"&gt;25&lt;/a&gt;, &lt;a title="B1 | Benta Berri-Berio-A&#xF1;orga" href="https://www.dbus.eus/b1-benta-berri-berio-anorga/"&gt;B1&lt;/a&gt;, &lt;a title="B8 | Miraconcha-Benta Berri-Seminario" href="https://www.dbus.eus/b8-miraconcha-benta-berri-seminario/"&gt;B8&lt;/a&gt;&lt;/p&gt;</desc_eu>
    <desc_en>&lt;p&gt;&lt;b&gt;Asociada&lt;/b&gt; a las l&#xED;neas &lt;a title="L&#xED;nea 5" href="http://www.dbus.eus/linea-5/"&gt;05&lt;/a&gt;, &lt;a title="Linea 25" href="http://www.dbus.eus/linea-25/"&gt;25&lt;/a&gt;&lt;br /&gt; Horarios de paso por parada(en)&lt;/p&gt;</desc_en>
    <desc_fr>&lt;p&gt;&lt;b&gt;Asociada&lt;/b&gt; a las l&#xED;neas &lt;a title="L&#xED;nea 5" href="http://www.dbus.eus/linea-5/"&gt;05&lt;/a&gt;, &lt;a title="Linea 25" href="http://www.dbus.eus/linea-25/"&gt;25&lt;/a&gt;&lt;br /&gt; Horarios de paso por parada(fr3)&lt;/p&gt;</desc_fr>
    <pic></pic>
    <icon>https://www.dbus.eus/wp-content/uploads/2014/03/05.png</icon>
    <linkd></linkd>
    <lat>43.32214974</lat>
    <lng>-1.983930094</lng>
    <anim>0</anim>
    <category>1</category>
    <infoopen>0</infoopen>
  </marker>
```

Lo ideal sería hacer una petición AJAX similar para obtener las paradas. Sin embargo, tras mirar la página de cada línea, me di cuenta de que cada una hacía referencia a un archivo XML diferente, con un parámetro único en la URL. Por ejemplo, la línea 05 usa `3markers.xml`, mientras que la línea 08 usa `11markers.xml`. Esto complicaba un poco las cosas, ya que no había una forma directa de saber qué archivo XML correspondía a cada línea sin inspeccionar el código fuente de cada página. Tal vez en un futuro me moleste en entender cómo se asignan esos números, pero como necesitaba un prototipo rápido, hice un apaño un poco sucio.

Al ejecutarse el código JavaScript en un cliente real, el archivo XML se carga correctamente y las paradas se muestran en el desplegable. Así que lo que hice fue **simular la ejecución del JavaScript** para obtener el HTML final con las paradas ya cargadas. Esto lo logré usando una librería llamada **Puppeteer**, que nos permite controlar un navegador desde nuestro código.

Lo sé, lo sé. Es un poco exagerado usar un navegador completo solo para obtener unas paradas. Pero funcionó, y era rápido de implementar. (Aunque esto trajo sus propios problemas, como veremos más adelante).

Vamos a ver cómo funciona el proceso paso a paso:

**1. Buscar la información de la línea**

Primero necesitamos saber la URL de la línea. Para esto, usamos la función que creamos anteriormente (`getBusLines()`) que obtiene todas las líneas disponibles. Buscamos la línea que coincida con el código que nos interesa:

```javascript
const allLines = await getBusLines();
const lineInfo = allLines.find(line => line.code === lineCode);

if (!lineInfo) {
    throw new Error(`Line ${lineCode} not found`);
}

// Ahora tenemos lineInfo.url, por ejemplo: "https://dbus.eus/05-benta-berri/"
```

**2. Abrir el navegador y navegar a la página**

Con Puppeteer, lanzamos un navegador headless (sin interfaz gráfica) y abrimos la URL de la línea:

```javascript
const browser = await puppeteer.launch({ headless: true });
const page = await browser.newPage();
await page.goto(lineInfo.url, { waitUntil: 'networkidle0' });
```

La opción `waitUntil: 'networkidle0'` le dice al navegador que espere hasta que no haya más peticiones de red activas, lo que asegura que el JavaScript se haya ejecutado completamente.

**3. Manejar las cookies**

Muchas páginas web muestran un banner de cookies. Si no lo aceptamos, puede que algunos elementos no carguen correctamente:

```javascript
// Intentar aceptar las cookies si aparece el banner
try {
    await page.click('#cookie-accept-button', { timeout: 2000 });
    await page.reload({ waitUntil: 'networkidle0' });
} catch (e) {
    // Si no hay banner de cookies, continuamos
}
```

**4. Esperar a que aparezca el selector de paradas**

El desplegable de paradas tiene el id `select_paradas_1`. Esperamos a que este elemento aparezca en el DOM:

```javascript
await page.waitForSelector('#select_paradas_1', { timeout: 10000 });
```

**5. Extraer las opciones del desplegable**

Una vez que el selector está presente, podemos extraer todas sus opciones. Usamos `page.evaluate()` para ejecutar código JavaScript dentro del contexto de la página:

```javascript
const stops = await page.evaluate(() => {
    const selectElement = document.querySelector('#select_paradas_1');
    const options = Array.from(selectElement.options);
    
    return options
        .filter(option => option.value && option.text.includes('|'))
        .map(option => {
            const [code, name] = option.text.split(/\s*\|\s*/);
            return {
                code: code ? code.trim() : '',
                name: name ? name.trim() : '',
                internal_id: option.value
            };
        });
});
```

Este código se ejecuta en el navegador (no en Node.js), por eso podemos usar `document.querySelector`. Extrae todas las opciones del `<select>`, filtra las que tienen el formato "código | nombre", y devuelve un array estructurado.

**6. Cerrar el navegador**

Importante no olvidarse de esto para liberar recursos:

```javascript
await browser.close();
```

Juntándolo todo, la función completa quedaría así:

```javascript
async function getBusStops(lineCode) {
    // 1. Buscar la información de la línea
    const allLines = await getBusLines();
    const lineInfo = allLines.find(line => line.code === lineCode);

    if (!lineInfo) {
        throw new Error(`Line ${lineCode} not found`);
    }

    // 2. Abrir el navegador y navegar a la página
    const browser = await puppeteer.launch({ headless: true });
    const page = await browser.newPage();
    await page.goto(lineInfo.url, { waitUntil: 'networkidle0' });

    // 3. Manejar las cookies
    try {
        await page.click('#cookie-accept-button', { timeout: 2000 });
        await page.reload({ waitUntil: 'networkidle0' });
    } catch (e) {
        // Si no hay banner de cookies, continuamos
    }

    // 4. Esperar a que aparezca el selector de paradas
    await page.waitForSelector('#select_paradas_1', { timeout: 10000 });

    // 5. Extraer las opciones del desplegable
    const stops = await page.evaluate(() => {
        const selectElement = document.querySelector('#select_paradas_1');
        const options = Array.from(selectElement.options);
        
        return options
            .filter(option => option.value && option.text.includes('|'))
            .map(option => {
                const [code, name] = option.text.split(/\s*\|\s*/);
                return {
                    code: code ? code.trim() : '',
                    name: name ? name.trim() : '',
                    internal_id: option.value
                };
            });
    });

    // 6. Cerrar el navegador
    await browser.close();

    return stops;
}
```

Y voilà! Tenemos nuestras paradas:

```json
[
  {
    "code": "34",
    "name": "Boulevard 17",
    "internal_id": "1587"
  },
  {
    "code": "35",
    "name": "Urbieta 20",
    "internal_id": "1590"
  },
  ...
]
```

El proceso completo tarda unos 3-5 segundos por línea, lo cual no está mal para un prototipo. Aunque como mencioné antes, usar un navegador completo solo para extraer datos de un `<select>` es un poco overkill. Pero cuando estás aprendiendo y necesitas que algo funcione rápido, a veces "lo que funciona" es mejor que "lo perfecto".

A todo esto, nunca había usado Puppeteer antes, así que aproveché que tenía **Copilot** en VSCode para ir pidiéndole lo que necesitaba y dejando que me lo implementara. Me ahorré mucho tiempo de estar buscando *"Cómo hacer X con Puppeteer"* en Google. También lo utilicé para **generar las expresiones regulares** (yo sigo sin saber cómo hay humanos capaces de crear esos patrones por sí mismos).

### Obteniendo los tiempos de llegada

Ya casi tenemos todo lo necesario. Ahora mismo contamos con la lista de líneas y con una lista de paradas para cada línea. El último paso es obtener los tiempos de llegada en tiempo real para una parada específica.

![Cuadro con los tiempos de llegada de la parada 34 - Boulevard 17](./assets/proximas-llegadas.png)

Tenemos un botón muy útil que dice *"Volver a consultar tiempos"*. Si inspeccionamos qué hace este botón cuando lo pulsamos, veremos que llama a una función JavaScript llamada `calcula_parada`. Aquí está el código relevante:

```javascript
jQuery("body").on("click", "#brecalcular_fecha", function() {
    var dia = new Date().getDate();
    var mes = new Date().getMonth() + 1;
    var year = new Date().getFullYear();
    var hora = new Date().getHours() + 100;
        hora = hora.toString().substring(1);
    var minutos = new Date().getMinutes() + 100;
        minutos = minutos.toString().substring(1);
    var parada_id = jQuery("#select_paradas").find(":selected").val();
    var linea = '05';
    
    calcula_parada(linea, parada_id, dia, mes, year, hora, minutos);
});
```

Hay algo curioso aquí: ese truco para formatear las horas y minutos. En lugar de usar `padStart(2, '0')` (que probablemente no existía o no conocían cuando escribieron esto), suman 100 al número. Por ejemplo, si son las 14:05:
- `14 + 100 = 114` → `.substring(1)` → `"14"`
- `5 + 100 = 105` → `.substring(1)` → `"05"`

Ingenioso, aunque un poco oscuro. Pero funciona.

La función `calcula_parada` hace una petición AJAX para obtener los tiempos de llegada:

```javascript
function calcula_parada(linea, parada_id, dia, mes, year, hora, minuto) {
    var data = {
        action: 'calcula_parada',
        security: 'bae19310a6',
        linea: linea,
        parada: parada_id,
        dia: dia,
        mes: mes,
        year: year,
        hora: hora,
        minuto: minuto
    };
    
    var ajaxurl = "https://dbus.eus/wp-admin/admin-ajax.php";
    
    jQuery.ajax({
        type: 'POST',
        url: ajaxurl,
        data: data,
        success: function(result) {
            jQuery("#consulta_horarios").html(result);
            jQuery("#proximas_llegadas").html(jQuery("#prox_lle").html());
            jQuery("#consulta_horarios").find("#prox_lle").remove();
            // ... más código para configurar el datepicker y timepicker
        },
        async: true
    });
}
```

¡Perfecto! Esto significa que podemos hacer la misma petición desde nuestro código para obtener los tiempos de llegada. Pero hay un detalle importante: el parámetro `security`.

#### El código de seguridad

Este código de seguridad es un mecanismo para prevenir peticiones no autorizadas. Si miramos el código fuente de la página de cada línea, podemos encontrarlo en uno de los scripts:

```javascript
security: 'a1b2c3d4e5'
```

Este código cambia periódicamente, así que necesitamos extraerlo cada vez que visitamos la página. Por suerte, ya estamos usando Puppeteer para obtener las paradas, así que podemos aprovechar para extraer este código al mismo tiempo.

Dentro del navegador, buscamos en todos los `<script>` de la página el que contiene la palabra `security`:

```javascript
const securityCode = await page.evaluate(() => {
    const scriptTags = Array.from(document.querySelectorAll('script'));
    
    for (const script of scriptTags) {
        if (script.textContent.includes('security')) {
            const match = script.textContent.match(/security:\s*'(\w+)'/);
            if (match) {
                return match[1];
            }
        }
    }
    return null;
});
```

Una vez que tenemos el código de seguridad, podemos hacer la petición para obtener los tiempos.

#### Haciendo la petición

Con todos los ingredientes listos, construimos la petición POST:

```javascript
async function getBusTimeAtStop(lineNumber, stopCode) {
    // 1. Obtener las paradas de la línea y encontrar la específica
    const stops = await getBusStops(lineNumber);
    const stop = stops.find(s => s.code === stopCode);

    if (!stop) {
        throw new Error(`Stop ${stopCode} not found`);
    }

    // 2. Construir los parámetros de la petición con la fecha/hora actual
    const now = new Date();
    const params = {
        action: 'calcula_parada',
        security: securityCode,  // El que extrajimos antes
        linea: lineNumber.toString(),
        parada: stop.internal_id,
        dia: now.getDate().toString().padStart(2, '0'),
        mes: (now.getMonth() + 1).toString().padStart(2, '0'),
        year: now.getFullYear().toString(),
        hora: now.getHours().toString().padStart(2, '0'),
        minuto: now.getMinutes().toString().padStart(2, '0')
    };

    // 3. Hacer la petición POST
    const response = await fetch('https://dbus.eus/wp-admin/admin-ajax.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: new URLSearchParams(params)
    });

    const html = await response.text();
    
    // 4. Parsear la respuesta...
}
```

#### Parseando la respuesta

La respuesta que recibimos es HTML que contiene una lista con los tiempos de llegada de todas las líneas que pasan por esa parada:

```html
<div id="prox_lle">
    <ul>
        <li>Linea 5: 14:35 (5 min)</li>
        <li>Linea 25: 14:42 (12 min)</li>
        <li>Linea 5: 14:50 (20 min)</li>
    </ul>
</div>
```

Necesitamos parsear este HTML y extraer solo el tiempo de la línea que nos interesa. Para eso usamos `jsdom`, una librería que nos permite manipular HTML en Node.js como si estuviéramos en un navegador:

```javascript
import { JSDOM } from 'jsdom';

function parseTimeResponse(html, lineNumber) {
    const dom = new JSDOM(html);
    const doc = dom.window.document;

    // Obtener todos los elementos de la lista
    const listItems = Array.from(doc.querySelectorAll('#prox_lle ul li'))
        .map(item => item.textContent.trim());
    
    // Buscar el que corresponde a nuestra línea
    const requestedLine = listItems.find(item => 
        item.includes(`Linea ${parseInt(lineNumber, 10)}:`)
    );

    if (!requestedLine) {
        throw new Error('Bus time not found');
    }

    // Extraer el tiempo del texto
    return extractTimeFromText(requestedLine);
}
```

#### Extrayendo el tiempo

El texto puede venir en dos formatos:
- `"14:35"` - hora exacta de llegada
- `"5 min"` - minutos restantes para la llegada

Necesitamos extraer los minutos que faltan para que llegue el bus:

```javascript
function extractTimeFromText(timeText) {
    // Intentar capturar formato "HH:MM"
    const timeMatch = timeText.match(/(\d{2}:\d{2})/);
    
    if (timeMatch) {
        // Calcular cuánto tiempo falta
        const now = new Date();
        const [hours, minutes] = timeMatch[1].split(':').map(Number);
        const busTime = new Date(now.getFullYear(), now.getMonth(), 
                                  now.getDate(), hours, minutes);
        
        const timeDiff = busTime - now;
        const minutesDiff = Math.floor(timeDiff / 60000);
        return Math.max(0, minutesDiff);
    }
    
    // Intentar capturar formato "X min"
    const minutesMatch = timeText.match(/(\d+)\s*min/);
    if (minutesMatch) {
        return parseInt(minutesMatch[1], 10);
    }

    throw new Error('Bus time format not recognized');
}
```

¡Y con esto ya tenemos todo! La función completa nos devuelve el número de minutos que faltan para que llegue nuestro bus. Escribir esta parte del blog me ha llevado un par de horas, no recordaba haber trabajado tanto cuando lo programé hace un año. Pero qué se le va a hacer. Ahora que ya podemos obtener toda la información que necesitamos, en la siguiente sección veremos cómo montar todo esto en una API restful.

## Segunda fase: Crear la API

Ya lo siento, pero me he quedado sin tiempo para hoy. En la próxima entrega veremos cómo montar todo esto en una API RESTful usando Node.js y Express, para que podamos consultar los tiempos de llegada de cualquier línea y parada de forma sencilla. ¡Hasta la próxima!

(Continuará...)