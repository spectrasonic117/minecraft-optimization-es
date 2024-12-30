# Guia de Optimizacion de Servidores de Minecraft

> _Nota para los usuarios que están en Vanilla, Fabric o Spigot (o cualquier fork de **Paper**) - vaya a su server.properties y cambie `sync-chunk-writes` a `false`. Esta opción está forzosamente establecida en false en Paper y sus forks, pero en otras implementaciones de servidor necesitas cambiarla a false manualmente. Esto permite al servidor guardar chunks fuera del hilo principal, disminuyendo la carga en el bucle principal._

# Guía para la versión 1.20 - 1.21, algunas cosas todavía pueden aplicarse a 1.15 - 1.19.

Basada en [esta guía](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) y otras fuentes (todas ellas están enlazadas a lo largo de la guía cuando son relevantes).

Utilice la tabla de contenidos situada más arriba (junto a `README.md`) para navegar fácilmente por esta guía.

# Introducción

Nunca habrá una guía que te dé resultados perfectos. Cada servidor tiene sus propias necesidades y límites sobre cuánto puedes o estás dispuesto a sacrificar. Juguetear con las opciones para ajustarlas a las necesidades de tu servidor es de lo que se trata. Esta guía sólo pretende ayudarte a entender qué opciones tienen impacto en el rendimiento y qué cambian exactamente. Si crees que has encontrado información incorrecta en esta guía, eres libre de abrir una incidencia o hacer un pull request para corregirla.

# Preparativos

## Servidor JAR

Tu elección de software de servidor puede hacer una gran diferencia en el rendimiento y las posibilidades de la API. Actualmente existen múltiples JARs de servidor populares y viables, pero también hay algunos de los que deberías mantenerte alejado por varias razones.

Las mejores opciones recomendadas:

-   [Paper](https://github.com/PaperMC/Paper)- El software de servidor más popular que pretende mejorar el rendimiento a la vez que corrige incoherencias de jugabilidad y mecánicas.
-   [Pufferfish](https://github.com/pufferfish-gg/Pufferfish)- Fork de Paper que pretende mejorar aún más el rendimiento del servidor.
-   [Purpur](https://github.com/PurpurMC/Purpur)- Fork de Pufferfish centrado en las características y la libertad de personalización.

Evita estos JARs de servidor:

-   Cualquier JAR de servidor de pago que reclame async cualquier cosa - 99,99% de posibilidades de ser una estafa.
-   Bukkit/CraftBukkit/Spigot - Extremadamente anticuado en términos de rendimiento comparado con otro software de servidor al que tenga acceso.
-   Cualquier plugin/software que habilite/deshabilite/recargue plugins en tiempo de ejecución. Vea [esta sección](#plugins-enablingdisabling-other-plugins) para entender por qué [**PLUGMAN**].
-   Versiones inestables de Paper, Pufferfish o Purpur, estos pueden causar inestabilidad y otros problemas. Si buscas rendimiento, optimice el servidor o invierta en un fork privado personal.

## Pre-generacion de mapa

La pregeneración de mapas, gracias a varias optimizaciones a la generación de chunk añadidas a lo largo de los años, ahora sólo es útil en servidores con CPUs terribles, de un solo hilo o limitadas. Sin embargo, la pregeneración se utiliza comúnmente para generar chunks para plugins de mapas del mundo como Pl3xMap o Dynmap.

Si aún así quieres pregenerar el mundo, puedes usar un plugin como [**Chunky**](https://github.com/pop4959/Chunky) para hacerlo. Asegúrate de establecer un borde del mundo para que tus jugadores no generen nuevos chunks. Ten en cuenta que la pregeneración puede tardar horas dependiendo del radio que establezcas en el plugin. Ten en cuenta que con Paper y superiores tus tps no se verán afectados por la carga de chunks, pero la velocidad de carga de chunks puede ralentizarse significativamente cuando la cpu de tu servidor está sobrecargada

Es clave recordar que el Overworld, el Nether y el End tienen bordes de mundo separados que deben configurarse para cada mundo. La dimensión del nether es 1/o más pequeña que el Overworld (si no se modifica con un datapack), así que si configuras el tamaño incorrectamente, tus jugadores podrían terminar fuera del borde del mundo

**Asegúrate de configurar un borde de mundo vainilla (`/worldborder set [diameter]`), ya que limita ciertas funcionalidades como el rango de búsqueda de mapas del tesoro que puede causar picos de lag.**

# Configuraciones

## Networking

### [server.properties]

#### network-compression-threshold

`Good starting value: 256`

Permite establecer el límite del tamaño de un paquete antes de que el servidor intente comprimirlo. Establecerlo más alto puede ahorrar algunos recursos de CPU a costa de ancho de banda, y establecerlo a -1 lo desactiva. Establecerlo más alto también puede perjudicar a los clientes con conexiones de red más lentas. Si su servidor está en una red con un proxy o en la misma máquina (con menos de 2 ms de ping), desactivar esto (-1) será beneficioso, ya que las velocidades de la red interna normalmente pueden manejar el tráfico adicional sin comprimir.

### [purpur.yml](https://purpurmc.org/docs/Configuration/)

#### use-alternate-keepalive

`Good starting value: true`

Puedes habilitar el sistema de keepalive alternativo de Purpur para que los jugadores con mala conexión no se desconecten tan a menudo. Tiene incompatibilidad conocida con TCPShield.

> Habilitar esto envía un paquete keepalive una vez por segundo a un jugador, y sólo se inicia por tiempo de espera si ninguno de ellos fue respondido en 30 segundos. Respondiendo a cualquiera de ellos en cualquier orden mantendrá al jugador conectado. AKA, no va a patear sus jugadores porque 1 paquete se cae en algún lugar a lo largo de las líneas
>
> ~ https://purpurmc.org/docs/Configuration/#use-alternate-keepalive

---

## Chunks

### [server.properties]

#### simulation-distance

`Good starting value: 4`

La distancia de simulación es la distancia en chunks alrededor del jugador que el servidor marcará. Esencialmente es la distancia desde el jugador a la que sucederán las cosas. Esto incluye hornos de fundición, cultivos y árboles jóvenes creciendo, etc. Esta es una opción que querrás poner baja a propósito, en algún lugar alrededor de `3` o `4`, debido a la existencia de `view-distance`. Esto permite cargar más chunks. Esto permite a los jugadores ver más lejos sin el mismo impacto en el rendimiento.

#### view-distance

`Good starting value: 7`

Esta es la distancia en chunks que se enviará a los jugadores, similar a la distancia de vista sin pulsar del papel.

La distancia total de la vista será igual al mayor valor entre `distancia-simulación` y `distancia-vista`. Por ejemplo, si la distancia de simulación es 4, y la distancia de vista es 12, la distancia total enviada al cliente será de 12 chunks.

### [spigot.yml](https://www.spigotmc.org/wiki/spigot-configuration/)

#### view-distance

`Good starting value: default`

Este valor sobrescribe server.properties uno si no se establece en `default`. Usted debe mantener por defecto para tener tanto la simulación y la distancia de vista en un solo lugar para facilitar la gestión.

### [paper-world configuration](https://docs.papermc.io/paper/reference/world-configuration)

#### delay-chunk-unloads-by

`Good starting value: 10s`

Esta opción le permite configurar cuánto tiempo permanecerán cargados los chunks después de que un jugador se vaya. Esto ayuda a no cargar y descargar constantemente los mismos chunks cuando un jugador va y viene. Valores demasiado altos pueden resultar en que se carguen demasiados chunks a la vez. En áreas que son frecuentemente teletransportadas y cargadas, considera mantener el área permanentemente cargada. Esto será más ligero para tu servidor que cargar y descargar chunks constantemente.

#### max-auto-save-chunks-per-tick

`Good starting value: 8`

Le permite ralentizar el ahorro incremental de mundo repartiendo la tarea en el tiempo aún más para un mejor rendimiento medio. Es posible que desee establecer este más alto que `8` con más de 20-30 jugadores. Si el guardado incremental no puede terminar a tiempo, bukkit guardará automáticamente los chunks sobrantes de una vez y comenzará el proceso de nuevo.

#### prevent-moving-into-unloaded-chunks

`Good starting value: true`

Cuando está activada, evita que los jugadores se desplacen a chunks no cargados y causen cargas de sincronización que atascan el hilo principal causando lag. La probabilidad de que un jugador tropiece con un trozo descargado es mayor cuanto menor sea la distancia de visión.

#### entity-per-chunk-save-limit

```
Good starting values:

    area_effect_cloud: 8
    arrow: 16
    dragon_fireball: 3
    egg: 8
    ender_pearl: 8
    experience_bottle: 3
    experience_orb: 16
    eye_of_ender: 8
    fireball: 8
    firework_rocket: 8
    llama_spit: 3
    potion: 8
    shulker_bullet: 8
    small_fireball: 8
    snowball: 8
    spectral_arrow: 16
    trident: 16
    wither_skull: 4
```

Con la ayuda de esta entrada puedes establecer límites a cuantas entidades de un tipo específico pueden ser guardadas. Usted debe proporcionar un límite para cada proyectil por lo menos para evitar problemas con cantidades masivas de proyectiles que se guardan y el servidor se bloquea en la carga que. Puedes poner cualquier id de entidad aquí, mira la wiki de minecraft para encontrar IDs de entidades. Por favor, ajuste el límite a su gusto. Valor sugerido para todos los proyectiles es de alrededor de `10`. También puede añadir otras entidades por sus nombres de tipo a esa lista. Esta opción de configuración no está diseñada para evitar que los jugadores hagan grandes granjas de mobs.

### [pufferfish.yml](https://docs.pufferfish.host/setup/pufferfish-fork-configuration/)

#### max-loads-per-projectile

`Good starting value: 8`

Especifica la cantidad máxima de chunks que un proyectil puede cargar durante su vida. Si se reduce, se reducirán las cargas de chunks causadas por proyectiles de entidad, pero podrían producirse problemas con tridentes, enderpearls, etc.

---

## Mobs

### [bukkit.yml](https://bukkit.fandom.com/wiki/Bukkit.yml)

#### spawn-limits

```
Good starting values:

    monsters: 20
    animals: 5
    water-animals: 2
    water-ambient: 2
    water-underground-creature: 3
    axolotls: 3
    ambient: 1
```

La matemática de limitar mobs es `[playercount] * [limit]`, donde "playercount" es la cantidad actual de jugadores en el servidor. Lógicamente, cuanto menor sea el número, menos mobs verás. `per-player-mob-spawn` aplica un límite adicional a esto, asegurando que los mobs se distribuyen equitativamente entre los jugadores. Reducir esto es un arma de doble filo; sí, tu servidor tiene menos trabajo que hacer, pero en algunos modos de juego los mobs de desove natural son una gran parte de la jugabilidad. Puedes llegar a 20 o menos si ajustas `mob-spawn-range` adecuadamente. Si ajustas `mob-spawn-range` más bajo parecerá que hay más mobs alrededor de cada jugador. Si estás usando Paper, puedes establecer límites de mobs por mundo en [paper-world configuration].

#### ticks-per

```
Good starting values:

    monster-spawns: 10
    animal-spawns: 400
    water-spawns: 400
    water-ambient-spawns: 400
    water-underground-creature-spawns: 400
    axolotl-spawns: 400
    ambient-spawns: 400
```

Esta opción establece la frecuencia (en ticks) con la que el servidor intenta desovar ciertas entidades vivas. Los mobs de agua/ambiente no necesitan aparecer cada tick ya que no suelen morir tan rápido. En cuanto a los monstruos: Aumentar ligeramente el tiempo entre desovaciones no debería afectar a la tasa de desovaciones incluso en granjas de monstruos. En la mayoría de los casos todos los valores de esta opción deberían ser superiores a "1". Establecer este valor más alto también permite a su servidor para hacer frente mejor a las zonas donde el desove mob está desactivado.

### [spigot.yml](https://www.spigotmc.org/wiki/spigot-configuration/)

#### mob-spawn-range

`Good starting value: 3`

Te permite reducir el rango (en chunks) de donde aparecerán los mobs alrededor del jugador. Dependiendo del modo de juego de tu servidor y de su número de jugadores, puede que quieras reducir este valor junto con `spawn-limits` de [bukkit.yml]. Establecer este valor más bajo hará que parezca que hay más mobs a tu alrededor. Este valor debería ser menor o igual que la distancia de simulación, y nunca mayor que el rango de despawn / 16.

#### entity-activation-range

```
Good starting values:

      animals: 16
      monsters: 24
      raiders: 48
      misc: 8
      water: 8
      villagers: 16
      flying-monsters: 48
```

Puedes establecer a qué distancia del jugador debe estar una entidad para que haga tick (hacer cosas). La reducción de estos valores ayuda al rendimiento, pero puede dar lugar a inconsistencias hasta que el jugador se acerca mucho a ellos. Reducir demasiado este valor puede romper ciertas granjas de mobs; las granjas de hierro son la víctima más común.

#### entity-tracking-range

```
Good starting values:

      players: 48
      animals: 48
      monsters: 48
      misc: 32
      other: 64
```

Esta es la distancia en bloques a partir de la cual las entidades serán visibles. Simplemente no serán enviadas a los jugadores. Si se establece demasiado bajo, puede hacer que los monstruos parezcan aparecer de la nada cerca de un jugador. En la mayoría de los casos debería ser mayor que tu
`entity-activation-range`.

#### tick-inactive-villagers

`Good starting value: false`

Esto te permite controlar si los aldeanos deben ser marcados fuera del rango de activación. Esto hará que los aldeanos procedan de forma normal e ignoren el rango de activación. Desactivar esta opción mejorará el rendimiento, pero puede resultar confuso para los jugadores en determinadas situaciones. Esto puede causar problemas con las granjas de hierro y el reabastecimiento del comercio.

#### nerf-spawner-mobs

`Good starting value: true`

Puedes hacer que los monstruos generados por un generador de monstruos no tengan IA. Los monstruos nerfeados no harán nada. Puedes hacer que salten mientras están en el agua cambiando `spawner-nerfed-mobs-should-jump` a `true` en [paper-world configuration].

### [paper-world configuration](https://docs.papermc.io/paper/reference/world-configuration)

#### despawn-ranges

```
Good starting values:

      ambient:
        hard: 72
        soft: 30
      axolotls:
        hard: 72
        soft: 30
      creature:
        hard: 72
        soft: 30
      misc:
        hard: 72
        soft: 30
      monster:
        hard: 72
        soft: 30
      underground_water_creature:
        hard: 72
        soft: 30
      water_ambient:
        hard: 72
        soft: 30
      water_creature:
        hard: 72
        soft: 30
```

Te permite ajustar los rangos de despawn de las entidades (en bloques). Baja esos valores para eliminar más rápido a los monstruos que están lejos del jugador. Deberías mantener el rango suave alrededor de `30` y ajustar el rango duro a un poco más de la distancia real de simulación, para que las criaturas no desaparezcan inmediatamente cuando el jugador va más allá del punto en el que un trozo está siendo cargado (esto funciona bien debido a `delay-chunk-unloads-by` en [paper-world configuration]). Cuando un mob está fuera del rango duro, será despawneado instantáneamente. Cuando esté entre el rango suave y el duro, tendrá una probabilidad aleatoria de despawn. Tu rango duro debe ser mayor que tu rango blando. Deberías ajustar esto de acuerdo a tu distancia de visión usando `(simulation-distance * 16) + 8`. Esto tiene en cuenta parcialmente los chunks que aún no se han descargado después de que el jugador los haya visitado.

#### per-player-mob-spawns

`Good starting value: true`

Esta opción decide si la aparición de mobs debe tener en cuenta cuántos mobs hay ya alrededor del jugador objetivo. Puedes evitar muchos problemas de inconsistencia en la aparición de mobs debido a que los jugadores crean granjas que ocupan todo el mobcap. Esto permitirá una experiencia de desove más parecida a la de un jugador, permitiéndote establecer `límites de desove` más bajos. La activación de esta opción tiene un ligero impacto en el rendimiento, pero queda eclipsado por las mejoras en los "límites de aparición" que permite.

#### max-entity-collisions

`Good starting value: 2`

Sobrescribe la opción con el mismo nombre en [spigot.yml]. Permite decidir cuántas colisiones puede procesar una entidad a la vez. Un valor de `0` causará la incapacidad de empujar a otras entidades, incluyendo jugadores. Un valor de `2` debería ser suficiente en la mayoría de los casos. Vale la pena señalar que esto hará maxEntityCramming gamerule inútil si su valor es superior al valor de esta opción de configuración.

#### update-pathfinding-on-block-update

`Good starting value: false`

Al desactivar esta opción, se hará menos pathfinding, lo que aumentará el rendimiento. En algunos casos, esto hará que los mobs parezcan más lentos; simplemente actualizarán pasivamente su ruta cada 5 ticks (0.25 seg).

#### fix-climbing-bypassing-cramming-rule

`Good starting value: true`

Al activar esta opción, las entidades no se verán afectadas por el apilamiento al escalar. Esto evitará que se apilen cantidades absurdas de mobs en espacios pequeños aunque estén trepando (arañas).

#### armor-stands.tick

`Good starting value: false`

En la mayoría de los casos se puede establecer con seguridad a `false`. Si estás usando soportes de armadura o cualquier plugin que modifique su comportamiento y experimentas problemas, vuelve a activarlo. Esto evitará que los soportes de armadura sean empujados por el agua o se vean afectados por la gravedad.

#### armor-stands.do-collision-entity-lookups

`Good starting value: false`

Aquí puedes desactivar las colisiones de los soportes de armadura. Esto te ayudará si tienes muchos puestos de armadura y no necesitas que colisionen con nada.

#### tick-rates

```
Good starting values:

  behavior:
    villager:
      validatenearbypoi: 60
      acquirepoi: 120
  sensor:
    villager:
      secondarypoisensor: 80
      nearestbedsensor: 80
      villagerbabiessensor: 40
      playersensor: 40
      nearestlivingentitysensor: 40
```

> It is not recommended to change these values from their defaults while [Pufferfish's DAB](#dabenabled) is enabled!

Esto decide la frecuencia con la que se disparan los comportamientos y sensores especificados en ticks. El comportamiento `acquirepoi` para los aldeanos parece ser el más pesado, por lo que se ha aumentado considerablemente. Disminúyelo en caso de problemas con los aldeanos para encontrar su camino.

### [pufferfish.yml](https://docs.pufferfish.host/setup/pufferfish-fork-configuration/)

#### dab.enabled

`Good starting value: true`

DAB (activación dinámica del cerebro) reduce la cantidad que se marca una entidad cuanto más lejos está de los jugadores. DAB funciona en un gradiente en lugar de un corte duro como EAR. En lugar de marcar completamente las entidades cercanas y apenas marcar las entidades lejanas, DAB reducirá la cantidad de marcación de una entidad basándose en el resultado de un cálculo influenciado por [dab.activation-dist-mod](#dabactivation-dist-mod).

#### dab.max-tick-freq

`Good starting value: 20`

Define la cantidad más lenta a la que se moverán las entidades más alejadas de los jugadores. El aumento de este valor puede mejorar el rendimiento de las entidades lejos de la vista, pero puede romper las granjas o nerf en gran medida el comportamiento mafia. Si al activar DAB se rompen las granjas, prueba a reducir este valor.

#### dab.activation-dist-mod

`Good starting value: 7`

Controla el gradiente en el que se marcan los mobs. Disminuir este valor activará DAB más cerca de los jugadores, mejorando las ganancias de rendimiento de DAB, pero afectará a cómo las entidades interactúan con su entorno y puede romper las granjas de turbas. Si al activar DAB se rompen las granjas de mobs, prueba a aumentar este valor.

#### enable-async-mob-spawning

`Good starting value: true`

Si se debe habilitar el desove asíncrono de mobs. Para que esto funcione, el ajuste de Paper per-player-mob-spawns debe estar habilitado. Esta opción no genera realmente mobs de forma asíncrona, pero descarga gran parte del esfuerzo computacional involucrado en la generación de nuevos mobs a un hilo diferente. Activar esta opción no debería afectar al juego.

#### enable-suffocation-optimization

`Good starting value: true`

Esta opción optimiza la comprobación de asfixia (la comprobación para ver si un monstruo está dentro de un bloque y si debería recibir daño por asfixia), limitando la tasa de comprobación al tiempo de espera del daño. Esta optimización debería ser imposible de notar a menos que seas un jugador extremadamente técnico que esté usando una sincronización precisa de ticks para matar a una entidad en el momento exacto por sofocación.

#### inactive-goal-selector-throttle

`Good starting value: true`

Acelera el selector de objetivos de la IA en los ticks de entidades inactivas, haciendo que las entidades inactivas actualicen su selector de objetivos cada 20 ticks en lugar de cada ticks. Puede mejorar el rendimiento en un pequeño porcentaje y tiene implicaciones menores en la jugabilidad.

### [purpur.yml](https://purpurmc.org/docs/Configuration/)

#### zombie.aggressive-towards-villager-when-lagging

`Good starting value: false`

Activar esta opción hará que los zombis dejen de apuntar a los aldeanos si el servidor está por debajo del umbral de tps establecido con `lagging-threshold` en [purpur.yml].

#### entities-can-use-portals

`Good starting value: false`

This option can disable portal usage of all entities besides the player. This prevents entities from loading chunks by changing worlds which is handled on the main thread. This has the side effect of entities not being able to go through portals.

#### villager.lobotomize.enabled

`Good starting value: true`

> This should only be enabled if villagers are causing lag! Otherwise, the pathfinding checks may decrease performance.

Los aldeanos lobotomizados son despojados de su IA y solo reponen sus ofertas cada cierto tiempo. Activar esta opción lobotomizará a los aldeanos que no puedan seguir la ruta hasta su destino. Si los liberas, los deslobotomizarás.

#### villager.search-radius

```
Good starting values:

          acquire-poi: 16
          nearest-bed-sensor: 16
```

Radio dentro del cual los aldeanos buscarán bloques de lugares de trabajo y camas. Esto aumenta significativamente el rendimiento con una gran cantidad de aldeanos, pero evitará que detecten bloques de lugares de trabajo o camas que estén más lejos del valor establecido.

---

## Misc

### [spigot.yml](https://www.spigotmc.org/wiki/spigot-configuration/)

#### merge-radius

```
Good starting values:

      item: 3.5
      exp: 4.0
```

Esto decide la distancia entre los ítems y orbes de exp a ser fusionados, reduciendo la cantidad de ítems tickeando en el suelo. Un valor demasiado alto hará que la ilusión de objetos u orbes de exp desaparezcan al fusionarse. Un valor demasiado alto romperá algunas granjas y permitirá que los objetos se teletransporten a través de los bloques. No se realizan comprobaciones para evitar que los objetos se fusionen a través de las paredes (a menos que esté activada la opción `fix-items-merging-through-walls` de Paper). La Exp sólo se fusiona en el momento de la creación.

#### hopper-transfer

`Good starting value: 8`

Tiempo en ticks que las tolvas esperarán para mover un artículo. Aumentar esto ayudará a mejorar el rendimiento si hay muchas tolvas en tu servidor, pero romperá los relojes basados en tolvas y posiblemente los sistemas de clasificación de artículos si se establece demasiado alto.

#### hopper-check

`Good starting value: 8`

Tiempo en ticks entre que las tolvas comprueban si hay un objeto sobre ellas o en el inventario sobre ellas. Aumentar este valor mejorará el rendimiento si hay muchas tolvas en el servidor, pero estropeará los relojes basados en tolvas y los sistemas de clasificación de artículos basados en flujos de agua.

### [paper-world configuration](https://docs.papermc.io/paper/reference/world-configuration)

#### alt-item-despawn-rate

```
Good starting values:

      enabled: true
      items:
        cobblestone: 300
        netherrack: 300
        sand: 300
        red_sand: 300
        gravel: 300
        dirt: 300
        short_grass: 300
        pumpkin: 300
        melon_slice: 300
        kelp: 300
        bamboo: 300
        sugar_cane: 300
        twisting_vines: 300
        weeping_vines: 300
        oak_leaves: 300
        spruce_leaves: 300
        birch_leaves: 300
        jungle_leaves: 300
        acacia_leaves: 300
        dark_oak_leaves: 300
        mangrove_leaves: 300
        cactus: 300
        diorite: 300
        granite: 300
        andesite: 300
        scaffolding: 600
```

Esta lista te permite establecer un tiempo alternativo (en ticks) para que ciertos tipos de objetos caídos desaparezcan más rápido o más lento que por defecto. Esta opción se puede utilizar en lugar de los plugins de limpieza de elementos junto con `merge-radius` para mejorar el rendimiento.

#### redstone-implementation

`Good starting value: ALTERNATE_CURRENT`

Sustituye el sistema de redstone por versiones más rápidas y alternativas que reducen las actualizaciones redundantes de bloques, reduciendo la cantidad de lógica que tiene que calcular tu servidor. El uso de una implementación no vainilla puede introducir inconsistencias menores con redstone muy técnico, pero las ganancias de rendimiento superan con creces los posibles problemas de nicho. Una opción de implementación no vainilla puede arreglar adicionalmente otras inconsistencias de redstone causadas por CraftBukkit.
La implementación `ALTERNATE_CURRENT` se basa en el mod [Alternate Current](https://modrinth.com/mod/alternate-current). Puedes encontrar más información sobre este algoritmo en su página de recursos.

#### hopper.disable-move-event

`Good starting value: false`

`InventoryMoveItemEvent` no se dispara a menos que haya un plugin escuchando activamente ese evento. Esto significa que sólo debe ponerlo a true si tiene tal(es) plugin(s) y no le importa que no puedan actuar sobre este evento. **No lo establezca a true si quiere usar plugins que escuchen este evento, por ejemplo, plugins de protección.**

#### hopper.ignore-occluding-blocks

`Good starting value: true`

Determina si las tolvas ignorarán los contenedores dentro de bloques llenos, por ejemplo la tolva minecart dentro de un bloque de arena o grava. Si se mantiene activada esta opción, se romperán algunos artilugios que dependen de ese comportamiento.

#### tick-rates.mob-spawner

`Good starting value: 2`

Esta opción te permite configurar la frecuencia con la que deben marcarse los generadores. Valores más altos significan menos lag si tienes muchos spawners, aunque si se establece demasiado alto (en relación con el retraso de tus spawners) las tasas de spawn de mobs disminuirán.

#### optimize-explosions

`Good starting value: true`

Establecer esto a `true` reemplaza el algoritmo de explosión vainilla por uno más rápido, a costa de una ligera inexactitud al calcular el daño de la explosión. Esto no suele ser perceptible.

#### treasure-maps.enabled

`Good starting value: false`

Generar mapas del tesoro es extremadamente caro y puede colgar un servidor si la estructura que está intentando localizar está en un trozo no generado. Sólo es seguro activar esto si has pregenerado tu mundo y establecido un borde de mundo vainilla.

#### treasure-maps.find-already-discovered

```
Good starting values:
      loot-tables: true
      villager-trade: true
```

El valor por defecto de esta opción obliga a los mapas recién generados a buscar estructuras inexploradas, que suelen estar en chunks aún no generados. Establecer esto a `true` hace que los mapas puedan llevar a las estructuras que fueron descubiertas anteriormente. Si no cambias esto a `true` puedes experimentar que el servidor se cuelgue o se cuelgue al generar nuevos mapas del tesoro. villager-trade" es para mapas comerciados por aldeanos y "loot-tables" se refiere a cualquier cosa que genere botín dinámicamente como cofres del tesoro, cofres de mazmorras, etc.

#### tick-rates.grass-spread

`Good starting value: 4`

Tiempo en ticks entre que el servidor intenta esparcir hierba o micelio. Esto hará que las grandes áreas de tierra tarden un poco más en convertirse en hierba o micelio. Establecerlo en torno a `4` debería funcionar bien si quieres reducirlo sin que se note la disminución de la velocidad de propagación.

#### tick-rates.container-update

`Good starting value: 1`

Tiempo en ticks entre actualizaciones de contenedores. Aumentarlo puede ayudar si las actualizaciones de contenedores te causan problemas (rara vez ocurre), pero facilita que los jugadores experimenten desincronización al interactuar con inventarios (objetos fantasma).

#### non-player-arrow-despawn-rate

`Good starting value: 20`

Tiempo en ticks después del cual las flechas disparadas por los mobs deberían desaparecer después de golpear algo. De todas formas, los jugadores no pueden recogerlas, así que puedes establecerlo en algo como `20` (1 segundo).

#### creative-arrow-despawn-rate

`Good starting value: 20`

Tiempo en ticks tras el cual las flechas disparadas por los jugadores en modo creativo deberían desaparecer después de golpear algo. De todas formas, los jugadores no pueden recogerlas, así que puedes establecerlo en algo como `20` (1 segundo).

### [pufferfish.yml](pufferfish.yml)

#### disable-method-profiler

`Good starting value: true`

Esta opción desactivará algunos perfiles adicionales realizados por el juego. Este perfilado no es necesario para correr en producción y puede causar lag adicional.

### [purpur.yml](https://purpurmc.org/docs/Configuration/)

#### dolphin.disable-treasure-searching

`Good starting value: true`

Evita que los delfines realicen búsquedas de estructuras similares a los mapas del tesoro

#### teleport-if-outside-border

`Good starting value: true`

Te permite teletransportar al jugador al spawn del mundo si se encuentra fuera de la frontera del mundo. Resulta útil, ya que la frontera del mundo de vainilla se puede evitar y el daño que causa al jugador se puede mitigar.

---

## Helpers

### [paper-world configuration](https://docs.papermc.io/paper/reference/world-configuration)

#### anti-xray.enabled

`Good starting value: true`

Activa esta opción para ocultar las menas de los rayos X. Para una configuración detallada de esta función, consulte [Configuring Anti-Xray](https://docs.papermc.io/paper/anti-xray). Activar esta opción disminuirá el rendimiento, sin embargo es mucho más eficiente que cualquier plugin anti-xray. En la mayoría de los casos el impacto en el rendimiento será insignificante.

#### nether-ceiling-void-damage-height

`Good starting value: 127`

Si esta opción es mayor que `0`, los jugadores por encima del nivel y establecido sufrirán daños como si estuvieran en el vacío. Esto evitará que los jugadores utilicen el techo del Nether. El nether vainilla tiene una altura de 128 bloques, por lo que probablemente deberías establecerlo en `127`. Si modificas la altura del nether de alguna manera, deberías establecerla en `[your_nether_height] - 1`.

---

# Java startup flags

[Vanilla Minecraft y Minecraft software de servidor en la versión 1.19 requiere Java 17 o superior](https://docs.papermc.io/java-install-update). Oracle ha cambiado su concesión de licencias, y ya no hay una razón de peso para obtener su Java de ellos. Los proveedores recomendados son [Adoptium](https://adoptium.net/) y [Amazon Corretto](https://aws.amazon.com/corretto/). Implementaciones alternativas de JVM como OpenJ9 o GraalVM pueden funcionar, sin embargo no están soportadas por Paper y se sabe que causan problemas, por lo que no se recomiendan actualmente.

Tu recolector de basura puede ser configurado para reducir los picos de lag causados por grandes tareas del recolector de basura. Puedes encontrar banderas de inicio optimizadas para servidores Minecraft [aquí](https://docs.papermc.io/paper/aikars-flags)[`SOG`]. Ten en cuenta que esta recomendación no funcionará en implementaciones JVM alternativas.
Se recomienda utilizar el generador de banderas de inicio [flags.sh](https://flags.sh) para obtener las banderas de inicio correctas para su servidor

Además, añadir la bandera beta `--add-modules=jdk.incubator.vector` antes de `-jar` en tus banderas de inicio puede mejorar el rendimiento. Esta bandera permite a Pufferfish usar instrucciones SIMD en tu CPU, haciendo algunas matemáticas más rápidas. Actualmente, sólo se utiliza para hacer el renderizado en mapas de plugins de juegos (como imageonmaps) posiblemente 8 veces más rápido.

# """Muy buenos para ser verdad""" plugins

## Plugins que eliminan items del suelo

Absolutamente innecesarios ya que pueden ser reemplazados por [merge-radius](#merge-radius) y [alt-item-despawn-rate](#alt-item-despawn-rate) y francamente, son menos configurables que las configuraciones básicas del servidor. Tienden a utilizar más recursos escaneando y eliminando elementos que no eliminándolos en absoluto.

## Mob stacker plugins

Es muy difícil justificar su uso. Apilar entidades creadas de forma natural causa más lag que no apilarlas debido a que el servidor está constantemente intentando crear más mobs. El único caso de uso "aceptable" es para los spawners en servidores con una gran cantidad de spawners.

## Plugins que activan o desactivan plugins

Cualquier cosa que active o desactive plugins en tiempo de ejecución es extremadamente peligroso. Cargar un plugin así puede causar errores fatales con los datos de rastreo y desactivar un plugin puede conducir a errores debido a la eliminación de la dependencia. El comando `/reload` sufre exactamente los mismos problemas y puedes leer más sobre ellos en [me4502's blog post](https://madelinemiller.dev/blog/problem-with-reload/)

# Midiendo Rendimiento

## mspt

Paper ofrece un comando `/mspt` que te dirá cuánto tiempo ha tardado el servidor en calcular los ticks recientes. Si el primer y segundo valor que ves son inferiores a **50**, ¡enhorabuena! Su servidor no tiene lag. Si el tercer valor es superior a 50, significa que ha habido al menos un tick que ha tardado más. Esto es completamente normal y ocurre de vez en cuando, así que no te asustes.

## Spark

[Spark](https://spark.lucko.me/) es un plugin que te permite perfilar el uso de CPU y memoria de tu servidor. Puedes leer cómo usarlo [en su wiki](https://spark.lucko.me/docs/). También hay una guía sobre cómo encontrar la causa de los picos de lag [**aquí**](https://spark.lucko.me/docs/guides/Finding-lag-spikes).

## Timings

Una forma de ver lo que puede estar pasando cuando tu servidor se está retrasando es Timings. Timings es una herramienta que le permite ver exactamente qué tareas están tomando más tiempo. Es la herramienta más básica de solución de problemas y si usted pide ayuda con respecto a lag lo más probable es que se le preguntó por su Timings. Timings es conocido por tener un serio impacto en el rendimiento de los servidores, se recomienda utilizar el plugin Spark sobre Timings y utilizar Purpur o Pufferfish para desactivar Timings.

Para obtener los tiempos de tu servidor, sólo tienes que ejecutar el comando `/timings paste` y hacer clic en el enlace que se te proporciona. Puedes compartir este enlace con otras personas para que te ayuden. También es fácil equivocarse si no sabes lo que estás haciendo. Hay un [videotutorial de Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) detallado sobre cómo leerlos.

---

# Minecraft exploits y como solucionarlos

Para ver cómo solucionar los exploits que pueden causar picos de lag o caídas en el servidor de Minecraft, consulte [**Aqui**](https://github.com/YouHaveTrouble/minecraft-exploits-and-how-to-fix-them).

<!-- --- -->

# Por que no haces una guia de optimizacion de Mods

Respuesta corta, no me gustan y desde hace años estoy efocado a plugins, es donde me he especializado

<!-- Basado en el proyecto original de [YouHaveTrouble](https://github.com/YouHaveTrouble/minecraft-optimization)
Traduccion hecha por [Spectrasonic](https://github.com/spectrasonic117) -->

[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[server.properties]: https://minecraft.wiki/w/Server.properties
[bukkit.yml]: https://bukkit.fandom.com/wiki/Bukkit.yml
[spigot.yml]: https://www.spigotmc.org/wiki/spigot-configuration/
[paper-global configuration]: https://docs.papermc.io/paper/reference/global-configuration
[paper-world configuration]: https://docs.papermc.io/paper/reference/world-configuration
[purpur.yml]: https://purpurmc.org/docs/Configuration/
[pufferfish.yml]: https://docs.pufferfish.host/setup/pufferfish-fork-configuration/
[Petal]: https://github.com/Bloom-host/Petal
