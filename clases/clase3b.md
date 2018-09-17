# para casos de prueba

Función palíndromo con acentos y minúsculas:

```js
function palindromo(texto){
    var tablaConversion={'á':'a', 'é':'e', 'í':'i', 'ó':'o', 'ú':'u', 'ü':'u'};
    texto=texto+'';
    texto=texto.toLowerCase();
    texto=texto.split('').map(function(letra){
        if(tablaConversion[letra]){ 
            return tablaConversion[letra];
        }
        return letra;
    }).join('');
    return texto.split('').every(function(letra, posicion){
        var otra_letra=texto[texto.length-posicion-1];
        return letra==otra_letra;
    })
}
```
