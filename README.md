# Cloud Keeper

Juego hecho en **lenguaje Jack** para la arquitectura **Hack** (nand2tetris).
Organización de Computadores · Evaluación 4 · 2026-2

> **Video en YouTube:** _(pegar aquí el enlace)_

## Equipo

- Jerónimo Zapata
- Nicolás Patiño
- Francis León

## El juego

Un globo cae con gravedad calculada y el jugador lo mantiene rebotando moviendo una
plataforma con las flechas del teclado. Cada vez que el globo rebota en la plataforma
el puntaje sube en pantalla, mientras las nubes se desplazan continuamente en el fondo
para dar sensación de movimiento.

| Tecla | Acción |
|---|---|
| `←` `→` | Mover la plataforma |
| `ESPACIO` | Empezar (pantalla de título) |
| `R` | Jugar otra vez (fin del juego) |
| `Q` | Salir |

- 3 vidas. Si el globo cae por debajo de la plataforma se pierde una.
- Cada rebote suma 1 punto. Cada 5 puntos sube el nivel: la gravedad aumenta y el juego
  se vuelve más rápido.

## Cómo cumple los 4 requisitos técnicos

| Requisito | Cómo lo hacemos |
|---|---|
| Interactividad por teclado | Se lee `Keyboard.keyPressed()` una vez por cuadro. No bloquea: si no hay tecla, el juego sigue. |
| Fondo móvil | 5 nubes que se desplazan sin parar hacia la izquierda, a 3 velocidades distintas (paralaje). Al salir por la izquierda reaparecen por la derecha. |
| Física modelada | Gravedad con aritmética de punto fijo (x16), rebote en paredes y techo, y rebote en la plataforma con ángulo según el punto de impacto (como Pong). |
| Puntaje visible | Marcador fijo arriba: puntaje, nivel y vidas, actualizado solo cuando cambia. |

## Estructura del código

```
CloudKeeper/
├── Main.jack      Punto de entrada, pantalla de título y reinicio
├── Game.jack      Bucle principal, marcador, niveles, fin del juego
├── Balloon.jack   Globo: física de punto fijo y dibujo
├── Platform.jack  Plataforma del jugador
└── Cloud.jack     Nube del fondo con dibujo incremental
```

```
Main ──crea──> Game ──tiene──> Balloon
                    ├─tiene──> Platform
                    └─tiene──> Cloud[5]
```

### Un cuadro del bucle principal (`Game.run`)

1. Leer el teclado sin bloquear y mover la plataforma.
2. Borrar el globo en su posición anterior.
3. Reparar las nubes que el borrado del globo tocó.
4. Mover las nubes.
5. Calcular la física del globo.
6. Dibujar el globo y la plataforma.
7. Actualizar el marcador si cambió.
8. Esperar un instante fijo (`Sys.wait`).

### Decisiones de ingeniería

- **Punto fijo en vez de decimales.** Hack solo tiene enteros de 16 bits. Posición y
  velocidad se guardan multiplicadas por 16, así la gravedad puede valer 0.125 px/cuadro².
- **Rebote de altura constante.** Al subir de nivel aumenta la gravedad `g`, y la velocidad
  de rebote se calcula con `v = √(2·g·h)` para que el globo siempre suba ~170 px.
  Así el juego se acelera sin volverse imposible.
- **Dibujo incremental (sin parpadeo).** Nunca se borra la pantalla completa. Para mover
  una nube 1 px solo se pinta una columna nueva a la izquierda y se borra una columna a la
  derecha de cada rectángulo. La plataforma solo borra la franja que deja atrás.
- **Sin fugas de memoria.** Los textos del marcador se imprimen una sola vez; en el bucle
  solo se reimprimen los números, porque cada `"texto"` en Jack crea un `String` nuevo
  en el heap.
- **Carriles para las nubes.** Cada nube va a una altura distinta, así nunca se borran
  entre ellas.

## Cómo ejecutarlo

1. Compilar la carpeta con el compilador de nand2tetris:
   ```
   JackCompiler CloudKeeper
   ```
2. Abrir la carpeta `CloudKeeper` en el **VM Emulator**.
3. Velocidad en "Fast", animación "No animation" (o "Screen"), y correr.

Si el juego va muy rápido o muy lento en su computador, se ajusta el valor de
`Sys.wait(5)` al final del bucle en `Game.jack`.
