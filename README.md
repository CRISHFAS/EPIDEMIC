# Epidemia

## Introducción

Este es un descripción muy breve, con una simulación escrita en JavaScript. Es un modelo muy simple de cómo una enfermedad puede propagarse en una población. Usted puede obtener el código, ajustar los parámetros y ver diferentes resultados. La idea de este proyecto surgió de esto: [Por qué los brotes como el coronavirus se propagan exponencialmente, y cómo "flatten la curva"](https://www.washingtonpost.com/graphics/2020/world/corona-simulator/). Te recomiendo mucho leerlo.

## Teoría

Recomiendo ver este video, para alguna teoría: [Video Teórico](https://youtu.be/Kas0tIxDvrg)

Es un buen comienzo, hay mucha más información sobre esto en Internet, así que no me molestaré con las fórmulas aquí.

## Simulación

El modelo se parece mucho al del enlace señalado anteriormente, con una adición: este modelo también simula la muerte, no todos los que se recuperan.

Brevemente, sólo la dinámica molecular en 2D, en una caja, con colisiones elásticas. Todas las partículas tienen la misma masa y pueden no estar infectadas, curadas o muertas. Los muertos desaparecen de la simulación. Una vez que una partícula infectada golpea a una que no lo estaba la infectará automáticamente. Una vez que tiene la enfermedad, tiene una posibilidad de morir (pero sólo en los dos últimos tercios del período de tiempo infectado). Después de estar enfermo por un tiempo, si no muere mientras tanto, se recuperará y tendrá inmunidad, por lo que no puede ser infectado de nuevo.

También hay un gráfico que muestra el número de infecciones con el tiempo. Notarás que se nivelará por debajo del 100%, la diferencia es de los que no contrajeron una infección, tener suerte y los muertos.

El código tiene una característica que suele ser un insecto: debido al paso demasiado grande, algunas partículas pueden mantenerse juntas. Básicamente cuando la colisión se superponen un poco (más, si el paso del tiempo es grande) y si no puede separarse en un paso de una sola vez, permanecer solapados, se considerarán como chocando de nuevo y así en adelante. Hay varias maneras de resolver esto pero para este modelo lo considero una característica: la gente a veces hace algo análogo.

Las partículas azules no están infectadas, las rojas están infectadas y las verdes se curan. Los que mueren desaparecen de la lona.

El cálculo será un poco lento en un teléfono, para esto debe utilizar mejor un escritorio.

Relacionado con esto, para una mejor implementación en tales dinámicas moleculares donde sólo hay interacción de corto alcance, visite [Event Driven Molecular Dynamics](https://compphys.go.ro/event-driven-molecular-dynamics/). También en 3D y con gráficos más bonitos, usando OpenGL.

## El código

Hay algunos comentarios en el código, esperemos que esté lo suficientemente claro. Lo escribí muy rápido, así que está lejos de ser perfecto, pero parece funcionar como estaba previsto.

### Original

```javascript
(function () {
    var canvas = document.getElementById("Canvas");
    var chart = document.getElementById("Chart");

    canvas.width = Math.min(window.innerWidth - 50, window.innerHeight - 50);
    if (canvas.width < 300) canvas.width = 300;
    else if (canvas.width > 800) canvas.width = 800;
    canvas.height = canvas.width;

    chart.width = canvas.width;
    chart.height = chart.width / 3;

    var ctx = canvas.getContext("2d");
    var canvasText = document.getElementById("canvasText");

    var ctxChart = chart.getContext("2d");
    ctxChart.strokeStyle = "#FF0000";
    ctxChart.lineWidth = 4;

    var people = [];

    // algunos parámetros
    var nrPeople = 500;
    var speed = canvas.width / 50.;
    var radius = canvas.width / 100.;
    var deltat = 0.01;
    var cureTime = 5.;
    var deathProb = 0.05;
    var infectProb = 1.;

    // gráfico
    var chartPosX = 0;
    var chartValY = 0;
    var chartXScale = 0.1;
    var chartYScale = chart.height / nrPeople;

    // estadísticas
    var deaths = 0;
    var infections = 1;
    var cures = 0;

    function Init() {
        deaths = 0;
        infections = 1;
        cures = 0;
        people = [];
        chartPosX = 0;
        chartValY = 0;

        for (i = 0; i < nrPeople; ++i) {
            var person =
            {
                posX: 0,
                posY: 0,
                velX: 0,
                velY: 0,
                dead: false,
                infected: false,
                cured: false,
                infectedTime: 0.0,

                NormalizeVelocity: function () {
                    var len = Math.sqrt(this.velX * this.velX + this.velY * this.velY);
                    this.velX /= len;
                    this.velY /= len;
                },

                Distance: function (other) {
                    var distX = this.posX - other.posX;
                    var distY = this.posY - other.posY;

                    return Math.sqrt(distX * distX + distY * distY);
                },

                Collision: function (other) {
                    var dist = this.Distance(other);

                    return dist <= 2. * radius;
                },

                Collide: function (other) {
                    var velXdif = this.velX - other.velX;
                    var velYdif = this.velY - other.velY;

                    var posXdif = this.posX - other.posX;
                    var posYdif = this.posY - other.posY;

                    var dist2 = posXdif * posXdif + posYdif * posYdif;
                    var dotProd = velXdif * posXdif + velYdif * posYdif;
                    dotProd /= dist2;

                    this.velX -= dotProd * posXdif;
                    this.velY -= dotProd * posYdif;

                    other.velX += dotProd * posXdif;
                    other.velY += dotProd * posYdif;
                }
            };

            for (; ;) {
                var X = Math.floor(Math.ran
```

[Enlace de demostración](https://epidemic-page.netlify.app)

Terminaré con una cita de [Will the pandemic Kill you?](https://www.bruceonpolitics.com/2020/03/12/will-the-pandemic-kill-you/) por el premio Nobel Peter Doherty:

> Lo que también nos dice la gripe es que, si bien los virus que se propagan a través de la vía respiratoria son la causa más probable de alguna pandemia futura, sólo las restricciones más draconianas e inmediatas en los viajes humanos probablemente limiten la propagación de la infección, y luego sólo brevemente.  
> Es probable que estas limitaciones se apliquen rápidamente si nos enfrentamos a una situación en la que, por ejemplo, más del 30% de los afectados desarrollan consecuencias graves o incluso fatales. La situación más peligrosa puede ser cuando las tasas de mortalidad se encuentran en el rango del 1-3%, causando (en última instancia) entre 70 y 210 millones de muertes en todo el mundo. Tal infección podría desaparecer antes de darnos cuenta de lo que estaba sucediendo.
