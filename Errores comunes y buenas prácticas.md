# Errores comunes y buenas prácticas

Este artículo pretende explicar los escollos más comunes a los que se enfrentan los propietarios de servidores.

## Siempre **haz Copia de seguridad**

Hay dos tipos de personas: las que hacen copias de seguridad y las que empezarán a hacerlas. Es sólo cuestión de tiempo que experimentes una pérdida de datos. Haz siempre copias para evitar perder tus mundos o los datos de tus plugins. Puedes aplicar esto a cualquier flujo de trabajo relacionado con la informática, no sólo a minecraft.

## No utilices software obsoleto

Si utilizas versiones de software obsoletas, corres el riesgo de que tus jugadores abusen de exploits no parcheados, incluyendo la duplicación de objetos (objetos infinitos). También añade un factor de inconveniencia ya que tus jugadores tienen que bajar específicamente su versión de cliente para que coincida con tu servidor. Esto puede evitarse utilizando un hack de protocolo, pero no es lo ideal.

## No ejecutes más Bukkit/Spigot

Bukkit y Spigot están básicamente en modo mantenimiento. Se actualizan cada vez que hay una nueva versión y si se encuentra un exploit crítico, pero no añaden ninguna actualización de rendimiento. Esto significa que cualquier problema de rendimiento que puedas experimentar en esos softwares nunca mejorará con el tiempo. Para evitarlo, actualice a [Paper](https://papermc.io/downloads) o [Purpur](https://purpurmc.org/downloads). Los plugins Bukkit/Spigot funcionarán igual de bien (quizá incluso mejor) con el software de servidor mencionado. Si no lo hacen, entonces es seguro asumir que el desarrollador del plugin está haciendo cosas que no debería o hizo un trabajo negligente al crear su plugin. También añaden parches de optimización como un sistema de carga de trozos que puede aprovechar múltiples hilos de cpu o un ajuste que permite al servidor marcar menos trozos de los que realmente envía al jugador. Consulta la [guía principal de optimización](https://github.com/spectrasonic117/minecraft-optimization) para más detalles.

## Evite el alojamiento compartido si es posible

Los alojamientos compartidos suelen ser la opción más barata, y eso es por una razón válida. Te ofrecen 2 tipos de recursos - garantizados y compartidos. Los recursos garantizados suelen ser irrisoriamente bajos y pueden no ser suficientes para hacer funcionar un servidor para unos pocos jugadores. Por otro lado, los recursos compartidos suelen ser suficientes para hacer funcionar un servidor con un rendimiento decente. Los recursos compartidos, como su nombre indica, se comparten entre tu servidor y otros servidores de la misma máquina física. Tu servidor sólo puede beneficiarse de tenerlos cuando ningún otro servidor los utiliza. La situación en la que tu servidor utiliza totalmente los recursos compartidos es prácticamente imposible, ya que la mayoría de los alojamientos compartidos sobrevenden sus recursos. Al igual que los billetes de avión, el sitio de alojamiento vende más recursos de los que tiene disponibles con la esperanza de que no se utilicen todos. Esto a menudo lleva a situaciones en las que todos los servidores están atascados porque no hay suficientes recursos de sobra.

## Evita los datapacks que usan funciones de comando

Los paquetes de datos que ejecutan comandos son extremadamente lentos. Puede que no sea mucho con pocos jugadores, pero eso no se escala bien con el número de jugadores y hará que tu servidor se ralentice rápidamente a medida que ganes jugadores. Los paquetes de datos que modifican biomas, tablas de botín, etc. están bien. Es mejor que busques un plugin alternativo.

## Avoid datapacks that use command functions.

Datapacks that run commands are extremely slow. It may not be much with few players, but that doesn't scale well with the number of players and will cause your server to slow down quickly as you gain players. Data packs that modify biomes, loot tables, etc. are fine. You're better off looking for an alternative plugin.

You should absolutely avoid Hard Drives (HDDs). Their speeds are simply way too slow to justify running a server on them since minecraft is heavy on I/O operations (especially with high view distances and higher player counts). A Solid State drive (SSD) is a far better choice because of it's much faster I/O.
