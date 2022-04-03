# Electron FAQ

## Por que teño problemas para instalar Electron?

Ao executar `npm install electron`, algúns usuarios atópanse ocasionalmente erros de instalación.

En case todos os casos, estes erros son o resultado de problemas de rede e non problemas reais co paquete npm `electron`. Erros como `ELIFECICLE`,  `EAI_AGAIN`, `ECONNRESET` e `ETIMEDOUT` son todos indicativos deste tipo  problemas de rede. A mellor resolución é probar a cambiar de rede ou  agarda un pouco e tenta instalar de novo.

Tamén podes tentar descargar Electron directamente dende [electron/electron/releases](https://github.com/electron/electron/releases)  se falla a instalación mediante `npm`.

## Cando se actualizará Electron á última versión de Chrome?

A versión Chrome de Electron adoita cambiarse nunha ou dúas semanas despois lánzase unha nova versión estable de Chrome. Esta estimación non está garantida e depende da cantidade de traballo que implica a actualización.

Só se emprega a versión estable de Chrome. Se é unha mellora importante e  está en versión beta ou dev será incorporada. 

Para obter máis información, consulte a [introdución á seguridade](tutorial/security.md).

## Cando se actualizará Electron ao último Node.js?

Cando se publica unha nova versión de Node.js, adoitamos esperar aproximadamente un mes
antes de actualizar o de Electron. Así podemos evitar ser afectados por erros
introducido nas novas versións de Node.js, o que ocorre con moita frecuencia.

New features of Node.js are usually brought by V8 upgrades, since Electron is
using the **V8** shipped by Chrome browser, the shiny new JavaScript feature of a
new Node.js version is usually already in Electron.

## Como compartir datos entre páxinas web?

Para compartir datos entre páxinas web (os procesos de renderizado) o xeito máis sinxelo é empregar a API HTML5 que xa están dispoñibles nos navegadores. As boas alternativas son
[Storage API][storage], [`localStorage`][local-storage],
[`sessionStorage`][session-storage], e [IndexedDB][indexed-db].

Alternativamente, pode usar as primitivas IPC que proporciona Electron. Para
compartir datos entre os procesos principal e de renderizado, pode usar o
[`ipcMain`](api/ipc-main.md) e [`ipcRenderer`](api/ipc-renderer.md) módulos.
Para comunicarse directamente entre páxinas web, pode enviar un
[`MessagePort`][message-port] dun a outro, posiblemente a través do proceso principal
usando [`ipcRenderer.postMessage()`](api/ipc-renderer.md#ipcrendererpostmessagechannel-message-transfer).

A comunicación posterior a través dos portos de mensaxes é directa e non se desvía
o proceso principal.

## A app desaparece aos poucos minutos. 

Isto ocorre cando aparece a variable que se utiliza para almacenar e o recolector de lixo o apaña.   

Se atopas este problema, os seguintes artigos poden resultar útiles:

* [Memory Management][memory-management]
* [Variable Scope][variable-scope]

Se queres unha solución rápida, podes facer que as variables sexan globais cambiandoo teu código a partir deste:

```javascript
const { app, Tray } = require('electron')
app.whenReady().then(() => {
  const tray = new Tray('/path/to/icon.png')
  tray.setTitle('hello world')
})
```

to this:

```javascript
const { app, Tray } = require('electron')
let tray = null
app.whenReady().then(() => {
  tray = new Tray('/path/to/icon.png')
  tray.setTitle('hello world')
})
```

## Non podo usar jQuery/RequireJS/Meteor/AngularJS en Electron.

Debido á integración de Electron con Node.js, hai algúns símbolos adicionais
inserido no DOM como `módulo`, `exportacións`, `require`. Isto causa problemas
para algunhas bibliotecas xa que queren inserir os símbolos co mesmo nome.

Para solucionar isto, pode desactivar a integración de nodos en Electron:

```javascript
// In the main process.
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({
  webPreferences: {
    nodeIntegration: false
  }
})
win.show()
```

Pero se queres manter as capacidades de usar Node.js e as API de Electron, tes que renomear os símbolos da páxina antes de incluír outras bibliotecas:

```html
<head>
<script>
window.nodeRequire = require;
delete window.require;
delete window.exports;
delete window.module;
</script>
<script type="text/javascript" src="jquery.js"></script>
</head>
```

## `require('electron').xxx` non está definido.

When using Electron's built-in module you might encounter an error like this:

```sh
> require('electron').webFrame.setZoomFactor(1.0)
Uncaught TypeError: Cannot read property 'setZoomLevel' of undefined
```

It is very likely you are using the module in the wrong process. For example
`electron.app` can only be used in the main process, while `electron.webFrame`
is only available in renderer processes.

## O tipo de letra parece borroso, que é isto e que podo facer?

If [sub-pixel anti-aliasing](https://alienryderflex.com/sub_pixel/) is deactivated, then fonts on LCD screens can look blurry. Example:

![subpixel rendering example]

Sub-pixel anti-aliasing needs a non-transparent background of the layer containing the font glyphs. (See [this issue](https://github.com/electron/electron/issues/6344#issuecomment-420371918) for more info).

To achieve this goal, set the background in the constructor for [BrowserWindow][browser-window]:

```javascript
const { BrowserWindow } = require('electron')
const win = new BrowserWindow({
  backgroundColor: '#fff'
})
```

The effect is visible only on (some?) LCD screens. Even if you don't see a difference, some of your users may. It is best to always set the background this way, unless you have reasons not to do so.

Notice that just setting the background in the CSS does not have the desired effect.

[memory-management]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Memory_Management
[variable-scope]: https://msdn.microsoft.com/library/bzt2dkta(v=vs.94).aspx
[electron-module]: https://www.npmjs.com/package/electron
[storage]: https://developer.mozilla.org/en-US/docs/Web/API/Storage
[local-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
[session-storage]: https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage
[indexed-db]: https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API
[message-port]: https://developer.mozilla.org/en-US/docs/Web/API/MessagePort
[browser-window]: api/browser-window.md
[subpixel rendering example]: images/subpixel-rendering-screenshot.gif
