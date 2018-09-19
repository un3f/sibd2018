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

casos que toman en cuenta los signos de puntuación

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
    texto=texto.replace(/[^\w]/g,'');
    return texto.split('').every(function(letra, posicion){
        var otra_letra=texto[texto.length-posicion-1];
        return letra==otra_letra;
    })
}
```

## ejemplo para el triángulo isósceles:
```js
function isosceles(a,b,c){
    return (a==b || b==c || a==c) && (a != c || a !=b || b!=c)
      && (a+b>c) && (a+c>b) && (b+c>a);
}
```
