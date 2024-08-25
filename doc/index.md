# ¿Cómo usar RxJS y no morir en el intento? 😱 💯
![Cómo sobrevivir a RxJS](https://media.giphy.com/media/YFkpsHWCsNUUo/giphy.gif)
## 1. Introducción a RxJS :rocket:
💬 **¿Qué es RxJS?**
- RxJS (Reactive Extensions for JavaScript) es una librería para manejar flujos de datos asíncronos y eventos mediante programación reactiva.
  
💬 **Conceptos clave:**
- **Observables:** Objetos que emiten datos a lo largo del tiempo.
- **Observers:** Subscriptores que reaccionan a los datos emitidos.
- **Operators:** Funciones que permiten transformar, filtrar, combinar o manipular flujos de datos.
- **Asynchronous:** Las operaciones pueden ejecutarse de manera independiente y no necesitan esperar a que otras terminen.


## 2. Explicación del flujo 👋
1. **Creación:** Un Observable emite datos.
2. **Suscripción:** Un Observer se suscribe para recibir esos datos.
3. **Transformación:** Uso de operadores para transformar los datos.
4. **Cancelación:** Desuscripción para liberar recursos.

**Diagrama:**
[Creación] → [Suscripción] → [Transformación] → [Cancelación]

**Ejemplo | [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-xzz2xd):**
```ts 
ngOnInit() {
    // [Creación]
    const _observable$ = new Observable<string>((observer) => {
      observer.next('Primer valor');
    });
    
    // [Subscripción]
    const subscription = _observable$
      // [Transformación]
      .pipe(debounceTime(1000))
      .subscribe((value) => {
        this.value = value;
      });
    // [Cancelación]
    setTimeout(() => {
      subscription.unsubscribe();
    }, 2000);
}
```
## 3. Operadores | [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-d4bxsh?file=src%2Fmain.ts)
- Acontinuación crearemos un operador para ejemplificar el uso: 
```ts 

[multiply-values.ts]
export function multiplyValue(
  number: number
): OperatorFunction<number, number> {
  return (source: Observable<number>): Observable<number> =>
    new Observable((observer) => {
      return source.pipe(map((value) => value * number)).subscribe(observer);
    });
}
...

[main.ts]
ngOnInit() {
  const _observable$ = of(1, 2, 3);
  const subscription = _observable$
    .pipe(
      multiplyValue(5),
      tap((value) => {
        this.values.push(value);
      })
    )
    .subscribe();

  setTimeout(() => {
    subscription.unsubscribe();
  }, 2000);
}
```
**Resultado:**
```
5
10
15
```
**Operadores más usados:**
- **map**: Transforma los valores emitidos por un observable. Aplica una función a cada valor emitido por un observable y emite los valores resultantes.
```ts
import { of } from 'rxjs';
import { map } from 'rxjs/operators';

of(1, 2, 3).pipe(
  map(value => value * 2)
).subscribe(console.log); // Output: 2, 4, 6
```
- **filter**:  Filtra los valores emitidos. Emite solo los valores que pasan una condición determinada.

```ts
import { of } from 'rxjs';
import { filter } from 'rxjs/operators';

of(1, 2, 3, 4, 5).pipe(
  filter(value => value % 2 === 0)
).subscribe(console.log); // Output: 2, 4
```
- **mergeMap**: Combina observables de manera plana. Mapea cada valor a un observable y fusiona los resultados.

```ts
import { of } from 'rxjs';
import { filter } from 'rxjs/operators';

of(1, 2, 3, 4, 5).pipe(
  filter(value => value % 2 === 0)
).subscribe(console.log); // Output: 2, 4
```

- **switchMap**: Cambia a un nuevo observable, cancelando el anterior. Mapea cada valor a un observable, pero solo emite los valores del observable más reciente.
  
```ts
import { of } from 'rxjs';
import { switchMap } from 'rxjs/operators';

of(1, 2, 3).pipe(
  switchMap(id => of(`Item ${id}`))
).subscribe(console.log); // Output: Item 3
```

- **concatMap**: Procesa los valores uno tras otro en orden. Mapea cada valor a un observable y concatena los resultados en orden.
  
```ts
import { of } from 'rxjs';
import { concatMap, delay } from 'rxjs/operators';

of(1, 2, 3).pipe(
  concatMap(value => of(`Item ${value}`).pipe(delay(1000)))
).subscribe(console.log); // Output: Item 1, Item 2, Item 3 (one per second)
```

- **debounceTime**: Espera un tiempo antes de emitir el valor más reciente. Retrasa la emisión de valores por un período de tiempo, emitiendo solo el valor más reciente después de la pausa.

```ts
import { fromEvent } from 'rxjs';
import { debounceTime, map } from 'rxjs/operators';

const input = document.getElementById('input');
fromEvent(input, 'input').pipe(
  debounceTime(300),
  map(event => (event.target as HTMLInputElement).value)
).subscribe(console.log);
```

- **distinctUntilChanged**: Evita emitir valores repetidos. Solo emite valores cuando son diferentes al valor anterior.

```ts
import { of } from 'rxjs';
import { distinctUntilChanged } from 'rxjs/operators';

of(1, 1, 2, 3, 3, 4).pipe(
  distinctUntilChanged()
).subscribe(console.log); // Output: 1, 2, 3, 4
```

- **catchError**: Maneja errores en la cadena de operadores. Permite manejar errores que ocurren en el flujo del observable y emitir un nuevo observable o valor alternativo.

```ts
import { of, throwError } from 'rxjs';
import { catchError } from 'rxjs/operators';

throwError('Error!').pipe(
  catchError(err => {
    console.error(err);
    return of('Recovered from error');
  })
).subscribe(console.log); // Output: Error! Recovered from error
```

- **take**: Emite un número específico de valores. Completa el observable después de emitir un número determinado de valores.

```ts
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

interval(1000).pipe(
  take(3)
).subscribe(console.log); // Output: 0, 1, 2
```

- **combineLatest**: Combina los valores más recientes de múltiples observables. Emite valores cuando cada uno de los observables emitió al menos un valor, combinando los últimos valores de cada uno.

```ts
import { combineLatest, of } from 'rxjs';

const observable1$ = of(1, 2, 3);
const observable2$ = of('a', 'b', 'c');

combineLatest([observable1$, observable2$]).subscribe(([value1, value2]) => {
  console.log(value1, value2);
});
// Output: 3 c (solo el último valor de cada observable)
```

- **forkJoin**: Espera a que todos los observables se completen y emite los últimos valores. Combina múltiples observables y emite un solo valor que contiene los últimos valores emitidos por cada uno de los observables cuando todos han completado.

```ts
import { of, forkJoin } from 'rxjs';
import { delay } from 'rxjs/operators';

const observable1$ = of('Hello').pipe(delay(1000));
const observable2$ = of('World').pipe(delay(2000));

forkJoin([observable1$, observable2$]).subscribe(([value1, value2]) => {
  console.log(value1, value2); // Output: Hello World
});
```

- **lastValueFrom**: Convierte un observable en una promesa y obtiene el último valor emitido. Convierte un observable en una promesa que resuelve con el último valor emitido por el observable. Si el observable no emite ningún valor, la promesa se rechaza.

```ts
import { of, lastValueFrom } from 'rxjs';
import { delay } from 'rxjs/operators';

const observable$ = of('Hello', 'World').pipe(delay(1000));

lastValueFrom(observable$).then(value => {
  console.log(value); // Output: World
}).catch(err => {
  console.error('Error:', err);
});
```

## 4. Ahora hablemos de lo que no se puede hacer, antipatrones que son...
![Antipatron](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExdzE5eWE1YmIxOXFnYzJmOG9rdGJucWRwNDN2cXl0ZWs3eXJsNjB2aCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/X9QEx7PMwNELs7l4p0/giphy.gif)
Los antipatrones son prácticas comunes en el desarrollo de software que, aunque pueden parecer útiles o eficientes en el corto plazo, generalmente conducen a problemas de mantenimiento, rendimiento o escalabilidad en el largo plazo. En el contexto de RxJS, es importante conocer estos antipatrones para evitarlos y así escribir código más robusto y sostenible.

1. **Suscripciones Anidadas | [Stackblitz](https://stackblitz.com/edit/stackblitz-starters-yj8cok?file=src%2Fmain.ts) | [Stackblitz 1](https://stackblitz.com/edit/stackblitz-starters-6undij?file=src%2Fmain.ts):**
Ocurre cuando se suscribe a un observable dentro de otro observable, en lugar de utilizar operadores de combinación como mergeMap, switchMap, concatMap, etc.

**Problema:**
Las suscripciones anidadas pueden llevar a un código difícil de seguir, errores de lógica y fugas de memoria.

```ts
  const observable1$ = of('Hello').pipe(delay(1000));
  const observable2$ = of('World').pipe(delay(2000));

  // Antipatrón: Suscripciones anidadas
  observable1$.subscribe((value1) => {
    console.log('First observable:', value1);
    observable2$.subscribe((value2) => {
      console.log('Second observable:', value2);
    });
  });
```
**Solución:**
```ts
  const observable1$ = of('Hello').pipe(delay(1000));
  const observable2$ = of('World').pipe(delay(2000));

  // Corrección: Usar switchMap en lugar de suscripciones anidadas
  observable1$
  .pipe(
    switchMap((value1) => {
      console.log('First observable:', value1);
      return observable2$;
    })
  )
  .subscribe((value2) => {
    console.log('Second observable:', value2);
  });
```

2. **No Desuscribirse:**
Ocurre cuando te suscribes a un observable, pero nunca te desuscribes, lo que puede causar fugas de memoria.

**Problema:**
En aplicaciones de larga duración, no desuscribirse de observables que no se completan puede llevar a que se acumulen suscripciones y a un consumo innecesario de memoria.
```ts
const subscription = observable$.subscribe(value => {
  console.log(value);
});
// No se llama a subscription.unsubscribe() en ningún momento
```
**Solución:**
Utiliza el operador takeUntil, unsubscribe manualmente o usa el patrón async en Angular:

```ts
ngOnDestroy() {
  this.destroy$.complete()
}
...
const subscription = observable$.pipe(
  takeUntil(this.destroy$)
).subscribe(value => {
  console.log(value);
});

// O manualmente
subscription.unsubscribe();
```

3. **Demasiadas Suscripciones en Componentes:**
Ocurre cuando se crean muchas suscripciones individuales en un componente en lugar de usar operadores para combinarlas.


**Problema:**
Esto lleva a un código desorganizado y difícil de mantener.
```ts
observable1$.subscribe(value1 => console.log(value1));
observable2$.subscribe(value2 => console.log(value2));
observable3$.subscribe(value3 => console.log(value3));
```

**Solución:**
Usa combineLatest, forkJoin u otros operadores de combinación

```ts
combineLatest([observable1$, observable2$, observable3$]).subscribe(([value1, value2, value3]) => {
  console.log(value1, value2, value3);
});
```


4. **Usar RxJS para Todo:**
RxJS es una herramienta poderosa, pero no todo requiere el uso de observables y operadores de RxJS.

**Problema:**
Abusar de RxJS para manejar lógica simple o sincronizaciones sencillas puede complicar innecesariamente el código.

```ts
// Uso innecesario de RxJS para una operación sincrónica simple
const observable$ = of(1).pipe(
  map(value => value + 1)
);

observable$.subscribe(console.log);
```
**Solución:**
```ts
const value = 1 + 1;
console.log(value); // Simple operación sincrónica
```

## En conclusión
RxJS ofrece una rica caja de herramientas para manejar la programación asíncrona y reactiva. Al comprender y aplicar correctamente los operadores disponibles, y al evitar antipatrones comunes, puedes escribir código más limpio, eficiente y mantenible. El dominio de RxJS es esencial para cualquier desarrollador que trabaje con aplicaciones modernas que requieran un manejo avanzado de flujos de datos.

![Hack the planet](
https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExY3FicG9rd2N6aTNuY3pocW1zdGw4bjZleXB3Znd5ajFjMGFkZGtscCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/FnGJfc18tDDHy/giphy.gif)
