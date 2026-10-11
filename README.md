# Guía Completa de Referencia y Arquitectura: Minecraft Alpha 1.2.6 (MCP) - Antigravity

> **Informe técnico exhaustivo y enciclopedia de desarrollo para modificación (Modding) del código fuente de Minecraft Alpha 1.2.6 decompilado con Minecraft Coder Pack.**

![Minecraft Version](https://img.shields.io/badge/Minecraft-Alpha%201.2.6-blue.svg)
![Decompiler](https://img.shields.io/badge/MCP-Minecraft%20Coder%20Pack-brightgreen.svg)
![Total Classes](https://img.shields.io/badge/Clases%20Analizadas-457-orange.svg)
![Status](https://img.shields.io/badge/Documentaci%C3%B3n-100%25%20Completa-success.svg)

---
## Tabla de Contenidos
1. [Introducción y Contexto Histórico de Alpha 1.2.6](#1-introducci%C3%B3n-y-contexto-hist%C3%B3rico-de-alpha-126)
2. [Arquitectura del Motor y Ciclos Fundamentales](#2-arquitectura-del-motor-y-ciclos-fundamentales)
   - [El Game Loop y Sincronización de Ticks (20 TPS)](#el-game-loop-y-sincronizaci%C3%B3n-de-ticks-20-tps)
   - [Espacio de Coordenadas, Chunks y Almacenamiento](#espacio-de-coordenadas-chunks-y-almacenamiento)
   - [Tubería Gráfica y Renderizado Inmediato (Tessellator)](#tuber%C3%ADa-gr%C3%A1fica-y-renderizado-inmediato-tessellator)
   - [Físicas, Cajas AABB y Colisiones](#f%C3%ADsicas-cajas-aabb-y-colisiones)
   - [Protocolo Multijugador SMP y Concurrencia de Red](#protocolo-multijugador-smp-y-concurrencia-de-red)
   - [Formato de Serialización NBT](#formato-de-serializaci%C3%B3n-nbt)
3. [Guía Rápida de Modificación (Cheat Sheet & Modder's CookBook)](#3-gu%C3%ADa-r%C3%A1pida-de-modificaci%C3%B3n-cheat-sheet--modders-cookbook)
   - [Cómo crear y registrar un nuevo Bloque](#c%C3%B3mo-crear-y-registrar-un-nuevo-bloque)
   - [Cómo crear y registrar un nuevo Ítem / Herramienta](#c%C3%B3mo-crear-y-registrar-un-nuevo-%C3%ADtem--herramienta)
   - [Cómo añadir Recetas de Crafteo y Fundición](#c%C3%B3mo-a%C3%B1adir-recetas-de-crafteo-y-fundici%C3%B3n)
   - [Cómo crear una nueva Criatura (Mob) y su Renderizado](#c%C3%B3mo-crear-una-nueva-criatura-mob-y-su-renderizado)
   - [Cómo modificar la Generación de Mundo (Minerales y Estructuras)](#c%C3%B3mo-modificar-la-generaci%C3%B3n-de-mundo-minerales-y-estructuras)
   - [Cómo crear una nueva Pantalla GUI](#c%C3%B3mo-crear-una-nueva-pantalla-gui)
4. [Catálogo Exhaustivo de los 13 Subsistemas (457 Clases)](#4-cat%C3%A1logo-exhaustivo-de-los-13-subsistemas-457-clases)
   - [01. Núcleo y Ciclo del Juego (Core Game Loop & Engine) (34 clases)](#01-core)
   - [02. Bloques y Materiales (Blocks, Materials & Mechanics) (66 clases)](#02-blocks)
   - [03. Objetos, Crafteo e Inventarios (Items, Crafting & Inventory) (47 clases)](#03-items)
   - [04. Entidades, Mobs y Físicas (Entities, Living Mobs, Vehicles & AI) (60 clases)](#04-entities)
   - [05. Generación de Mundo y Biomas (World Generation & Biomes) (19 clases)](#05-worldgen)
   - [06. Mundo, Chunks, Iluminación y Guardado (World, Chunks, Lighting & Storage) (30 clases)](#06-world)
   - [07. TileEntities y Contenedores (TileEntities & Containers) (13 clases)](#07-tileentities)
   - [08. Motor de Renderizado y Gráficos (Rendering Pipeline, Models & FX) (83 clases)](#08-rendering)
   - [09. Interfaz de Usuario y Menús (GUI, HUD & Screens) (30 clases)](#09-gui)
   - [10. Redes y Protocolo Multijugador (Networking & Packets) (46 clases)](#10-networking)
   - [11. Sistema NBT y Serialización (Named Binary Tag) (13 clases)](#11-nbt)
   - [12. Motor de Audio y Sonido (Sound Engine & Music) (5 clases)](#12-audio)
   - [13. Matemáticas, Física y Estructuras de Datos (Math & Fast Collections) (11 clases)](#13-math)
5. [Índice Maestro Alfabético (Quick Lookup A-Z)](#5-%C3%ADndice-maestro-alfab%C3%A9tico-quick-lookup-a-z)

---
## 1. Introducción y Contexto Histórico de Alpha 1.2.6
La versión **Alpha 1.2.6** de Minecraft (lanzada el 3 de diciembre de 2010) representa la cúspide y versión final de la era Alpha antes de la llegada de la Beta 1.0. Incorpora todas las mecánicas clave introducidas durante la célebre **Halloween Update** (Alpha 1.2.0): la dimensión del Inframundo (**Nether**), portales de obsidiana, biomas con mapas de temperatura/humedad, bloques del Nether (Netherrack, Soul Sand, Glowstone), y las primeras criaturas interdimensionales (Ghasts y Zombie Pigmen).

El código fuente presente en este espacio de trabajo ha sido decompilado y renombrado mediante **Minecraft Coder Pack (MCP)**. Consta de **457 clases Java** repartidas entre:
- `net.minecraft.client` (2 clases): Puntos de entrada del ejecutable de escritorio y del applet web.
- `net.minecraft.src` (455 clases): Todo el núcleo lógico, gráfico, físico, procedural y de red del juego.

---
## 2. Arquitectura del Motor y Ciclos Fundamentales
### El Game Loop y Sincronización de Ticks (20 TPS)
El cliente se ejecuta en un bucle continuo `while(running)` dentro de [`Minecraft.run()`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/client/Minecraft.java). Para garantizar que la simulación física y la lógica del mundo sean independientes de la tasa de cuadros por segundo (FPS), el motor utiliza la clase [`Timer`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Timer.java):

1. **Tick Lógico Fijo (20 Hz / 50 ms por tick)**: Se invoca `runTick()`. En cada tick se procesan los movimientos del jugador, la actualización de entidades vivas (`EntityLiving.onUpdate()`), los bloques programados en la cola de ticks (`World.TickUpdates`), la propagación de agua/lava/redstone y la recepción de paquetes de red.
2. **Renderizado Desacoplado con Interpolación**: Entre dos ticks lógicos, la tarjeta gráfica puede dibujar fotogramas a 60, 120 o más FPS. Para evitar movimientos toscos, `Timer` calcula la fracción de tiempo transcurrida `renderPartialTicks` (un valor flotante entre `0.0` y `1.0`). Los renderizadores interpolan linealmente entre la posición anterior (`prevPosX`) y la posición actual (`posX`):
```java
double renderX = entity.prevPosX + (entity.posX - entity.prevPosX) * (double)renderPartialTicks;
```

### Espacio de Coordenadas, Chunks y Almacenamiento
El mundo de Minecraft Alpha está particionado en columnas verticales llamadas **Chunks** ([`Chunk.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Chunk.java)):

- **Dimensiones de Chunk**: 16 bloques de ancho en X, 16 bloques de largo en Z, y exactamente **128 bloques de altura en Y** (de 0 a 127).
- **Conversión de Coordenadas**: Para convertir coordenadas globales de bloque a coordenadas de chunk se aplica desplazamiento de bits:
  ```java
  int chunkX = blockX >> 4; // blockX / 16
  int chunkZ = blockZ >> 4; // blockZ / 16
  ```
- **Estructura de Memoria en RAM**: Cada chunk almacena sus datos en arrays unidimensionales planos de tamaño `16 * 16 * 128 = 32,768` elementos:
  - `byte[] blocks`: Almacena el ID del bloque (0 a 255).
  - `NibbleArray data`: Metadatos de 4 bits por bloque (orientación, nivel de agua, etapas de cultivo).
  - `NibbleArray skylight`: Nivel de luz solar natural (0 a 15).
  - `NibbleArray blocklight`: Nivel de luz emitida por antorchas, lava, fuego y glowstone (0 a 15).
  - `int[] heightMap`: Altura del bloque opaco más elevado en cada columna X-Z para acelerar el cálculo de sombras.

### Tubería Gráfica y Renderizado Inmediato (Tessellator)
El renderizado de Minecraft Alpha 1.2.6 se fundamenta en la tubería de función fija de OpenGL 1.x/2.x sobre **LWJGL** (Lightweight Java Game Library). Para no colapsar la GPU con llamadas individuales `glVertex3f`, Notch implementó la clase [`Tessellator`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Tessellator.java):

- **Búfer Nativo Directo**: Tessellator acumula coordenadas espaciales (X, Y, Z), coordenadas UV de textura y colores RGBA en un `ByteBuffer` directo de memoria nativa fuera del heap de Java.
- **Emisión Masiva de Quads**: Mediante un único llamado a `glDrawArrays`, miles de polígonos se despachan a la GPU en milisegundos.
- **Listas de Visualización de Chunks (Display Lists)**: [`WorldRenderer.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldRenderer.java) divide cada chunk en cubos de 16x16x16 y compila su geometría en listas OpenGL (`glGenLists` / `glCallList`). Solo se recompilan cuando un bloque del fragmento se modifica o cambia la iluminación.

### Físicas, Cajas AABB y Colisiones
- **AxisAlignedBB**: Las colisiones se evalúan mediante cajas delimitadoras tridimensionales alineadas con los ejes coordenados ([`AxisAlignedBB.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/AxisAlignedBB.java)).
- **Piscina de Reciclaje (Memory Pool)**: Para evitar instanciar millones de objetos `new AxisAlignedBB()` por segundo (lo que provocaría pausas constantes por recolección de basura), Notch diseñó un pool estático reciclable: `AxisAlignedBB.getBoundingBoxFromPool(minX, minY, minZ, maxX, maxY, maxZ)`.
- **Gravedad y Fricción**: Las entidades aplican aceleración de gravedad hacia abajo en `motionY` en cada tick (`motionY -= 0.08D`), y desaceleración por fricción del aire o del bloque inferior (`motionX *= 0.91D`, `motionZ *= 0.91D`).

### Protocolo Multijugador SMP y Concurrencia de Red
El protocolo cliente-servidor de Alpha 1.2.6 es asíncrono y orientado a paquetes sobre TCP:

- [`NetworkManager.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetworkManager.java): Dispone de dos hilos secundarios dedicados: uno para lectura continua (`NetworkReaderThread`) y otro para escritura (`NetworkWriterThread`).
- [`Packet.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet.java): Mantiene un mapa estático bidireccional entre IDs en formato de byte (0x00 a 0xFF) y las clases de paquetes.
- [`NetClientHandler.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetClientHandler.java): Despacha cada paquete entrante modificando el mundo local del cliente.

### Formato de Serialización NBT
El formato **Named Binary Tag (NBT)** es el estándar de serialización binaria creado por Notch para guardar el estado del juego de forma jerárquica y compacta:

- Árbol de etiquetas tipadas: [`NBTTagByte`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagByte.java) (1), `NBTTagShort` (2), `NBTTagInt` (3), `NBTTagLong` (4), `NBTTagFloat` (5), `NBTTagDouble` (6), `NBTTagByteArray` (7), `NBTTagString` (8), `NBTTagList` (9), [`NBTTagCompound`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagCompound.java) (10).
- [`CompressedStreamTools`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CompressedStreamTools.java): Comprime y descomprime los árboles NBT en disco mediante flujos GZIP (`GZIPInputStream` / `GZIPOutputStream`).

---
## 3. Guía Rápida de Modificación (Cheat Sheet & Modder's CookBook)
Tutoriales paso a paso con código real adaptado a la arquitectura de Minecraft Alpha 1.2.6:

### Cómo crear y registrar un nuevo Bloque
1. **Crear la clase del bloque** (ej. `BlockRuby.java` en `net/minecraft/src`):
```java
package net.minecraft.src;

import java.util.Random;

public class BlockRuby extends Block {
    public BlockRuby(int blockId, int textureIndex) {
        super(blockId, textureIndex, Material.rock);
        setHardness(3.0F);       // Dureza similar al diamante
        setResistance(5.0F);     // Resistencia a explosiones
        setStepSound(soundStoneFootstep);
        setLightValue(0.5F);     // Emite luz suave (nivel 8 de 15)
    }

    // Qué ítem suelta al romperse
    public int idDropped(int metadata, Random random) {
        return Item.diamond.shiftedIndex; // O tu ítem personalizado
    }

    // Cuántos ítems suelta
    public int quantityDropped(Random random) {
        return 1;
    }
}
```
2. **Registrar el bloque en [`Block.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Block.java)**:
```java
// Busca un ID libre menor a 256 (por ejemplo 95)
public static final Block rubyBlock = (new BlockRuby(95, 120)).setBlockName("rubyBlock");
```
3. **Registrar el ítem del bloque en [`Item.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Item.java)**:
```java
Item.itemsList[Block.rubyBlock.blockID] = new ItemBlock(Block.rubyBlock.blockID - 256);
```

### Cómo crear y registrar un nuevo Ítem / Herramienta
1. **Crear la clase del ítem** (ej. `ItemRubyWand.java`):
```java
package net.minecraft.src;

public class ItemRubyWand extends Item {
    public ItemRubyWand(int itemId) {
        super(itemId);
        maxStackSize = 1;        // No apilable
        setMaxDamage(64);        // 64 usos de durabilidad
        setIconIndex(50);        // Índice del icono en /gui/items.png
    }

    // Acción al hacer clic derecho con el ítem en la mano
    public ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer player) {
        itemstack.damageItem(1); // Desgasta 1 punto de durabilidad
        world.playSoundAtEntity(player, "random.bow", 0.5F, 0.4F);
        if(!world.multiplayerWorld) {
            world.entityJoinedWorld(new EntitySnowball(world, player)); // Dispara proyectil
        }
        return itemstack;
    }
}
```
2. **Registrar el ítem en [`Item.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Item.java)**:
```java
// Elige un ID libre mayor o igual a 256 (desplazado a shiftedIndex)
public static Item rubyWand = (new ItemRubyWand(400)).setItemName("rubyWand");
```

### Cómo añadir Recetas de Crafteo y Fundición
En [`CraftingManager.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingManager.java) añade:
```java
// Receta con patrón (3x3 o 2x2)
addRecipe(new ItemStack(Item.rubyWand, 1), new Object[] {
    "  R",
    " S ",
    "S  ",
    'R', Block.rubyBlock,
    'S', Item.stick
});
```
En [`FurnaceRecipes.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/FurnaceRecipes.java) añade:
```java
// Fundir un bloque de rubí devuelve un diamante
addSmelting(Block.rubyBlock.blockID, new ItemStack(Item.diamond, 1));
```

### Cómo crear una nueva Criatura (Mob) y su Renderizado
1. **Crear la entidad** (ej. `EntityGoblin.java`):
```java
package net.minecraft.src;

public class EntityGoblin extends EntityMobs {
    public EntityGoblin(World world) {
        super(world);
        texture = "/mob/goblin.png";
        moveSpeed = 0.8F;
        attackStrength = 3;
        health = 15;
    }

    protected int getDropItemId() {
        return Item.goldNugget.shiftedIndex;
    }
}
```
2. **Registrar la entidad en [`EntityList.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityList.java)**:
```java
addMapping(EntityGoblin.class, "Goblin", 65); // ID numérico de entidad libre
```
3. **Registrar el renderizador en [`RenderManager.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderManager.java)**:
```java
entityRenderMap.put(EntityGoblin.class, new RenderBiped(new ModelBiped(), 0.5F));
```

### Cómo modificar la Generación de Mundo (Minerales y Estructuras)
En [`ChunkProviderGenerate.populate()`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderGenerate.java):
```java
// Generar 10 vetas de RubyBlock por chunk entre Y=0 e Y=24
for(int v = 0; v < 10; v++) {
    int rx = chunkX + rand.nextInt(16);
    int ry = rand.nextInt(24);
    int rz = chunkZ + rand.nextInt(16);
    (new WorldGenMinable(Block.rubyBlock.blockID, 8)).generate(world, rand, rx, ry, rz);
}
```

### Cómo crear una nueva Pantalla GUI
Crear una subclase de [`GuiScreen.java`](file:///c:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiScreen.java):
```java
package net.minecraft.src;

public class GuiCustomMenu extends GuiScreen {
    public void initGui() {
        controlList.clear();
        controlList.add(new GuiButton(1, width / 2 - 100, height / 4 + 48, "¡Hacer Algo!"));
        controlList.add(new GuiButton(2, width / 2 - 100, height / 4 + 72, "Volver"));
    }

    protected void actionPerformed(GuiButton button) {
        if(button.id == 1) {
            mc.thePlayer.addChatMessage("¡Botón presionado!");
        } else if(button.id == 2) {
            mc.displayGuiScreen(null); // Cerrar pantalla
        }
    }

    public void drawScreen(int mouseX, int mouseY, float renderPartialTicks) {
        drawDefaultBackground();
        drawCenteredString(fontRenderer, "Mi Pantalla Personalizada", width / 2, 40, 0xFFFFFF);
        super.drawScreen(mouseX, mouseY, renderPartialTicks);
    }
}
```


---
## 4. Catálogo Exhaustivo de los 13 Subsistemas (457 Clases)
Documentación pormenorizada de cada archivo del código fuente organizado por responsabilidad arquitectónica:


### 01. Núcleo y Ciclo del Juego (Core Game Loop & Engine)
> **Resumen del subsistema**: Clases fundamentales que inicializan la máquina virtual, gestionan la ventana de visualización mediante LWJGL, coordinan el ciclo de ticks (20 TPS) frente a la tasa de cuadros (FPS), procesan la entrada del usuario y manejan las sesiones y excepciones críticas.

**Total de clases**: 34

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`CanvasCrashReport`](#class-canvascrashreport) | `class` | `ext Canvas` | 17 | Lienzo de dibujo para la pantalla de reporte de fallo (Crash Repo... |
| [`CanvasIsomPreview`](#class-canvasisompreview) | `class` | `ext Canvas, impl KeyListener, MouseListener, MouseMotionListener, Runnable
` | 466 | Lienzo de dibujo del visor de mapas isométrico integrado en Alpha... |
| [`CanvasMajongLogo`](#class-canvasmajonglogo) | `class` | `ext Canvas` | 33 | Lienzo gráfico para dibujar el logotipo inicial de Mojang. |
| [`CanvasMinecraftApplet`](#class-canvasminecraftapplet) | `class` | `ext Canvas` | 30 | Componente Canvas de AWT para el lienzo de renderizado del Applet... |
| [`EnumOS1`](#class-enumos1) | `enum` | `-` | 49 | Enumeración de sistemas operativos compatibles (Windows, macOS, S... |
| [`EnumOS2`](#class-enumos2) | `enum` | `-` | 50 | Enumeración auxiliar de variantes de sistemas operativos. |
| [`EnumOSMappingHelper`](#class-enumosmappinghelper) | `class` | `-` | 37 | Ayudante de mapeo entre cadenas de arquitectura y valores de enum... |
| [`GameSettings`](#class-gamesettings) | `class` | `-` | 391 | Gestor del archivo de configuración options.txt y preferencias de... |
| [`GameWindowListener`](#class-gamewindowlistener) | `class` | `ext WindowAdapter` | 35 | Oyente de eventos de la ventana gráfica de AWT/LWJGL. |
| [`KeyBinding`](#class-keybinding) | `class` | `-` | 18 | Asociación entre una acción de juego y una tecla física del tecla... |
| [`Minecraft`](#class-minecraft) | `class` | `impl Runnable
` | 1492 | El orquestador maestro y corazón del cliente de Minecraft. |
| [`MinecraftApplet`](#class-minecraftapplet) | `class` | `ext Applet` | 145 | Punto de entrada para ejecutar Minecraft dentro de un navegador w... |
| [`MinecraftAppletImpl`](#class-minecraftappletimpl) | `class` | `ext Minecraft` | 28 | Implementación concreta de Minecraft asociada a la ventana Canvas... |
| [`MinecraftError`](#class-minecrafterror) | `class` | `ext Error` | 13 | Error irrecuperable lanzado por el motor cuando ocurre un fallo c... |
| [`MinecraftException`](#class-minecraftexception) | `class` | `ext RuntimeException` | 14 | Excepción general del juego para fallos de carga o guardado. |
| [`MinecraftImpl`](#class-minecraftimpl) | `class` | `ext Minecraft` | 27 | Implementación estándar de Minecraft para ejecución nativa en esc... |
| [`MouseHelper`](#class-mousehelper) | `class` | `-` | 57 | Controlador de bajo nivel para el puntero del ratón. |
| [`MovementInput`](#class-movementinput) | `class` | `-` | 36 | Clase base de entrada de movimiento para el jugador. |
| [`MovementInputFromOptions`](#class-movementinputfromoptions) | `class` | `ext MovementInput` | 89 | Lee el estado físico del teclado según las opciones configuradas ... |
| [`OsMap`](#class-osmap) | `class` | `-` | 36 | Mapeo de opciones y rutas según el sistema operativo detectado. |
| [`PanelCrashReport`](#class-panelcrashreport) | `class` | `ext Panel` | 92 | Panel contenedor AWT para la presentación de informes de errores. |
| [`PlayerController`](#class-playercontroller) | `class` | `-` | 126 | Clase base abstracta para controladores de modo de juego del juga... |
| [`PlayerControllerMP`](#class-playercontrollermp) | `class` | `ext PlayerController` | 212 | Controlador del jugador para partidas multijugador. |
| [`PlayerControllerSP`](#class-playercontrollersp) | `class` | `ext PlayerController` | 147 | Controlador del modo Supervivencia para un solo jugador. |
| [`PlayerControllerTest`](#class-playercontrollertest) | `class` | `ext PlayerController` | 46 | Controlador de pruebas experimentales y modo creativo preliminar. |
| [`Session`](#class-session) | `class` | `-` | 57 | Almacena las credenciales de la sesión activa del jugador. |
| [`ThreadConnectToServer`](#class-threadconnecttoserver) | `class` | `ext Thread` | 63 | Hilo asíncrono para negociar la conexión con un servidor multijug... |
| [`ThreadDownloadImage`](#class-threaddownloadimage) | `class` | `ext Thread` | 55 | Hilo secundario para descargar imágenes de texturas en segundo pl... |
| [`ThreadDownloadImageData`](#class-threaddownloadimagedata) | `class` | `-` | 23 | Contenedor y gestor de estado para imágenes descargadas asíncrona... |
| [`ThreadDownloadResources`](#class-threaddownloadresources) | `class` | `ext Thread` | 163 | Hilo en segundo plano para descargar la música y efectos de sonid... |
| [`ThreadRunIsoClient`](#class-threadrunisoclient) | `class` | `ext Thread` | 29 | Hilo de procesamiento para la generación del mapa isométrico. |
| [`ThreadSleepForever`](#class-threadsleepforever) | `class` | `ext Thread` | 32 | Hilo utilitario que mantiene un hilo durmiendo indefinidamente co... |
| [`Timer`](#class-timer) | `class` | `-` | 68 | Sincronizador del reloj de juego (20 TPS) desacoplado de la tasa ... |
| [`UnexpectedThrowable`](#class-unexpectedthrowable) | `class` | `-` | 18 | Envoltorio estructurado para excepciones imprevistas. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-canvascrashreport"></a>[`CanvasCrashReport`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CanvasCrashReport.java)
- **Ruta y Paquete**: `net.minecraft.src.CanvasCrashReport` (`net/minecraft/src/CanvasCrashReport.java`) | **Líneas de código**: 17
- **Declaración**: `class CanvasCrashReport extends Canvas`
- **Rol**: Lienzo de dibujo para la pantalla de reporte de fallo (Crash Report).
- **Descripción Técnica**: Dibuja el texto del error y la traza de pila de la excepción si Minecraft se detiene inesperadamente.
- **Guía de Modificación**: Permite personalizar el aspecto de la pantalla de error.

##### <a id="class-canvasisompreview"></a>[`CanvasIsomPreview`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CanvasIsomPreview.java)
- **Ruta y Paquete**: `net.minecraft.src.CanvasIsomPreview` (`net/minecraft/src/CanvasIsomPreview.java`) | **Líneas de código**: 466
- **Declaración**: `class CanvasIsomPreview extends Canvas implements KeyListener, MouseListener, MouseMotionListener, Runnable
`
- **Rol**: Lienzo de dibujo del visor de mapas isométrico integrado en Alpha.
- **Descripción Técnica**: Renderiza la proyección isométrica del mundo en tiempo real.
- **Guía de Modificación**: Permite modificar la escala y resolución del visor isométrico.
- **Métodos Clave Destacados**:
  - `File getDataFolder()`
  - `File func_1264_a(String s)`
  - `void func_1270_b(String s)`
  - `void func_1272_b()`
  - `void func_1273_c()`
  - `void run()`

##### <a id="class-canvasmajonglogo"></a>[`CanvasMajongLogo`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CanvasMajongLogo.java)
- **Ruta y Paquete**: `net.minecraft.src.CanvasMajongLogo` (`net/minecraft/src/CanvasMajongLogo.java`) | **Líneas de código**: 33
- **Declaración**: `class CanvasMajongLogo extends Canvas`
- **Rol**: Lienzo gráfico para dibujar el logotipo inicial de Mojang.
- **Descripción Técnica**: Muestra el logo corporativo durante la carga inicial del cliente.
- **Guía de Modificación**: Permite personalizar el logotipo de arranque de un modpack.
- **Métodos Clave Destacados**:
  - `void paint(Graphics g)`

##### <a id="class-canvasminecraftapplet"></a>[`CanvasMinecraftApplet`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CanvasMinecraftApplet.java)
- **Ruta y Paquete**: `net.minecraft.src.CanvasMinecraftApplet` (`net/minecraft/src/CanvasMinecraftApplet.java`) | **Líneas de código**: 30
- **Declaración**: `class CanvasMinecraftApplet extends Canvas`
- **Rol**: Componente Canvas de AWT para el lienzo de renderizado del Applet.
- **Descripción Técnica**: Controla el repintado y dimensiones del área gráfica dentro del navegador.
- **Guía de Modificación**: Modificar si se ajusta la integración con la ventana AWT.

##### <a id="class-enumos1"></a>[`EnumOS1`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumOS1.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumOS1` (`net/minecraft/src/EnumOS1.java`) | **Líneas de código**: 49
- **Declaración**: `enum EnumOS1`
- **Rol**: Enumeración de sistemas operativos compatibles (Windows, macOS, Solaris, Linux).
- **Descripción Técnica**: Utilizada en la detección del entorno de ejecución.
- **Guía de Modificación**: Parte del subsistema de compatibilidad multiplataforma.
- **Métodos Clave Destacados**:
  - `EnumOS1[] func_1064_values()`
  - `EnumOS1 valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumOS1 linux`
  - `EnumOS1 solaris`
  - `EnumOS1 windows`
  - `EnumOS1 macos`
  - `EnumOS1 unknown`

##### <a id="class-enumos2"></a>[`EnumOS2`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumOS2.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumOS2` (`net/minecraft/src/EnumOS2.java`) | **Líneas de código**: 50
- **Declaración**: `enum EnumOS2`
- **Rol**: Enumeración auxiliar de variantes de sistemas operativos.
- **Descripción Técnica**: Complementa EnumOS1 para la clasificación detallada de la plataforma.
- **Guía de Modificación**: Parte del subsistema de detección del entorno.
- **Métodos Clave Destacados**:
  - `EnumOS2[] values()`
  - `EnumOS2 valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumOS2 linux`
  - `EnumOS2 solaris`
  - `EnumOS2 windows`
  - `EnumOS2 macos`
  - `EnumOS2 unknown`

##### <a id="class-enumosmappinghelper"></a>[`EnumOSMappingHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumOSMappingHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumOSMappingHelper` (`net/minecraft/src/EnumOSMappingHelper.java`) | **Líneas de código**: 37
- **Declaración**: `class EnumOSMappingHelper`
- **Rol**: Ayudante de mapeo entre cadenas de arquitectura y valores de enumeración de SO.
- **Descripción Técnica**: Normaliza las propiedades de sistema 'os.name' de Java.
- **Guía de Modificación**: Garantiza la portabilidad de rutas entre Windows, Linux y Mac.

##### <a id="class-gamesettings"></a>[`GameSettings`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GameSettings.java)
- **Ruta y Paquete**: `net.minecraft.src.GameSettings` (`net/minecraft/src/GameSettings.java`) | **Líneas de código**: 391
- **Declaración**: `class GameSettings`
- **Rol**: Gestor del archivo de configuración options.txt y preferencias del usuario.
- **Descripción Técnica**: Serializa y almacena opciones gráficas, distancia de renderizado, FOV, volumen y enlaces de teclas (KeyBinding).
- **Guía de Modificación**: Añade nuevas opciones para mods registrándolas en esta clase.
- **Métodos Clave Destacados**:
  - `String getKeyBinding(int i)`
  - `void setKeyBinding(int i, int j)`
  - `void setOptionFloatValue(int i, float f)`
  - `void setOptionValue(int i, int j)`
  - `int getOptionControlType(int i)`
  - `float getOptionFloatValue(int i)`
- **Campos Clave Destacados**:
  - `float musicVolume`
  - `float soundVolume`
  - `float mouseSensitivity`
  - `boolean invertMouse`
  - `int renderDistance`
  - `boolean viewBobbing`

##### <a id="class-gamewindowlistener"></a>[`GameWindowListener`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GameWindowListener.java)
- **Ruta y Paquete**: `net.minecraft.src.GameWindowListener` (`net/minecraft/src/GameWindowListener.java`) | **Líneas de código**: 35
- **Declaración**: `class GameWindowListener extends WindowAdapter`
- **Rol**: Oyente de eventos de la ventana gráfica de AWT/LWJGL.
- **Descripción Técnica**: Detecta el cierre de la ventana para guardar el mundo y pausar la partida si se pierde el foco.
- **Guía de Modificación**: Permite interceptar el evento de apagado para liberar recursos personalizados.
- **Métodos Clave Destacados**:
  - `void windowClosing(WindowEvent windowevent)`

##### <a id="class-keybinding"></a>[`KeyBinding`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/KeyBinding.java)
- **Ruta y Paquete**: `net.minecraft.src.KeyBinding` (`net/minecraft/src/KeyBinding.java`) | **Líneas de código**: 18
- **Declaración**: `class KeyBinding`
- **Rol**: Asociación entre una acción de juego y una tecla física del teclado.
- **Descripción Técnica**: Almacena el código de tecla de LWJGL, la descripción textual y el estado actual de pulsación.
- **Guía de Modificación**: Crear nuevas instancias de KeyBinding para añadir teclas de acceso rápido a mods.
- **Campos Clave Destacados**:
  - `String keyDescription`
  - `int keyCode`

##### <a id="class-minecraft"></a>[`Minecraft`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/client/Minecraft.java)
- **Ruta y Paquete**: `net.minecraft.client.Minecraft` (`net/minecraft/client/Minecraft.java`) | **Líneas de código**: 1492
- **Declaración**: `class Minecraft implements Runnable
`
- **Rol**: El orquestador maestro y corazón del cliente de Minecraft.
- **Descripción Técnica**: Inicializa la máquina virtual, crea la ventana LWJGL Display, gestiona el bucle principal de ejecución 'while(running)', coordina el reloj de ticks a 20 TPS fijos mediante Timer, maneja la carga de texturas, sonido, entrada de teclado y ratón, cambios de pantalla GUI (displayGuiScreen) y renderizado general.
- **Guía de Modificación**: La clase más importante del cliente: punto de partida para inyectar inicializaciones de mods o ganchos globales.
- **Métodos Clave Destacados**:
  - `void func_6258_a(String s, int i)`
  - `void startGame()`
  - `void func_6274_a(int i, int j, int k, int l, int i1, int j1)`
  - `File getMinecraftDir()`
  - `File getAppDir(String s)`
  - `void displayGuiScreen(GuiScreen guiscreen)`
- **Campos Clave Destacados**:
  - `PlayerController field_6327_b`
  - `int displayWidth`
  - `int displayHeight`
  - `World theWorld`
  - `RenderGlobal field_6323_f`
  - `EntityPlayerSP thePlayer`

##### <a id="class-minecraftapplet"></a>[`MinecraftApplet`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/client/MinecraftApplet.java)
- **Ruta y Paquete**: `net.minecraft.client.MinecraftApplet` (`net/minecraft/client/MinecraftApplet.java`) | **Líneas de código**: 145
- **Declaración**: `class MinecraftApplet extends Applet`
- **Rol**: Punto de entrada para ejecutar Minecraft dentro de un navegador web mediante Java Applet.
- **Descripción Técnica**: Inicializa el contenedor Canvas de AWT y arranca el hilo principal de ejecución delegando en MinecraftAppletImpl.
- **Guía de Modificación**: Relevante si se desea recrear o emular la ejecución web clásica de Minecraft en navegadores antiguos.
- **Métodos Clave Destacados**:
  - `void init()`
  - `void func_6233_a()`
  - `void start()`
  - `void stop()`
  - `void destroy()`
  - `void shutdown()`

##### <a id="class-minecraftappletimpl"></a>[`MinecraftAppletImpl`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MinecraftAppletImpl.java)
- **Ruta y Paquete**: `net.minecraft.src.MinecraftAppletImpl` (`net/minecraft/src/MinecraftAppletImpl.java`) | **Líneas de código**: 28
- **Declaración**: `class MinecraftAppletImpl extends Minecraft`
- **Rol**: Implementación concreta de Minecraft asociada a la ventana Canvas del Applet.
- **Descripción Técnica**: Sobrescribe métodos de resolución de pantalla y manejo de errores para adaptar el juego al marco del navegador.
- **Guía de Modificación**: Maneja la integración entre el ciclo de vida del Applet y el bucle principal de Minecraft.
- **Métodos Clave Destacados**:
  - `void func_4007_a(UnexpectedThrowable unexpectedthrowable)`

##### <a id="class-minecrafterror"></a>[`MinecraftError`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MinecraftError.java)
- **Ruta y Paquete**: `net.minecraft.src.MinecraftError` (`net/minecraft/src/MinecraftError.java`) | **Líneas de código**: 13
- **Declaración**: `class MinecraftError extends Error`
- **Rol**: Error irrecuperable lanzado por el motor cuando ocurre un fallo crítico de Java.
- **Descripción Técnica**: Indica que el hilo de ejecución debe detenerse de inmediato para evitar corrupción del mundo.
- **Guía de Modificación**: Capturado por el gestor de crashes del juego.

##### <a id="class-minecraftexception"></a>[`MinecraftException`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MinecraftException.java)
- **Ruta y Paquete**: `net.minecraft.src.MinecraftException` (`net/minecraft/src/MinecraftException.java`) | **Líneas de código**: 14
- **Declaración**: `class MinecraftException extends RuntimeException`
- **Rol**: Excepción general del juego para fallos de carga o guardado.
- **Descripción Técnica**: Lanzada ante errores de lectura de chunks o recursos.
- **Guía de Modificación**: Útil para manejar errores controlados en lógica de mods.

##### <a id="class-minecraftimpl"></a>[`MinecraftImpl`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MinecraftImpl.java)
- **Ruta y Paquete**: `net.minecraft.src.MinecraftImpl` (`net/minecraft/src/MinecraftImpl.java`) | **Líneas de código**: 27
- **Declaración**: `class MinecraftImpl extends Minecraft`
- **Rol**: Implementación estándar de Minecraft para ejecución nativa en escritorio.
- **Descripción Técnica**: Adapta la creación del marco principal y manejo de fallos para entornos de escritorio autónomos.
- **Guía de Modificación**: Punto de entrada alternativo al cliente standalone.
- **Métodos Clave Destacados**:
  - `void func_4007_a(UnexpectedThrowable unexpectedthrowable)`

##### <a id="class-mousehelper"></a>[`MouseHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MouseHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.MouseHelper` (`net/minecraft/src/MouseHelper.java`) | **Líneas de código**: 57
- **Declaración**: `class MouseHelper`
- **Rol**: Controlador de bajo nivel para el puntero del ratón.
- **Descripción Técnica**: Captura y centra el cursor dentro de la ventana de juego para permitir el movimiento continuo de cámara.
- **Guía de Modificación**: Modificar si se implementa control táctil o sensibilidad no lineal.
- **Métodos Clave Destacados**:
  - `void func_774_a()`
  - `void func_773_b()`
  - `void func_772_c()`
- **Campos Clave Destacados**:
  - `int field_1114_a`
  - `int field_1113_b`

##### <a id="class-movementinput"></a>[`MovementInput`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MovementInput.java)
- **Ruta y Paquete**: `net.minecraft.src.MovementInput` (`net/minecraft/src/MovementInput.java`) | **Líneas de código**: 36
- **Declaración**: `class MovementInput`
- **Rol**: Clase base de entrada de movimiento para el jugador.
- **Descripción Técnica**: Almacena las variables cinemáticas moveStrafe (lateral) y moveForward (adelante/atrás), y los estados de salto y agachado.
- **Guía de Modificación**: Heredar de esta clase para crear sistemas de control por IA, mando (gamepad) o cinemáticas automáticas.
- **Métodos Clave Destacados**:
  - `void func_797_a(EntityPlayer entityplayer)`
  - `void func_798_a()`
  - `void func_796_a(int i, boolean flag)`
- **Campos Clave Destacados**:
  - `float field_1174_a`
  - `float field_1173_b`
  - `boolean field_1177_c`
  - `boolean field_1176_d`
  - `boolean field_1175_e`

##### <a id="class-movementinputfromoptions"></a>[`MovementInputFromOptions`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MovementInputFromOptions.java)
- **Ruta y Paquete**: `net.minecraft.src.MovementInputFromOptions` (`net/minecraft/src/MovementInputFromOptions.java`) | **Líneas de código**: 89
- **Declaración**: `class MovementInputFromOptions extends MovementInput`
- **Rol**: Lee el estado físico del teclado según las opciones configuradas en GameSettings.
- **Descripción Técnica**: Transforma la pulsación de WASD, espacio y shift en vectores de movimiento para EntityPlayerSP.
- **Guía de Modificación**: Permite remapear o invertir controles.
- **Métodos Clave Destacados**:
  - `void func_796_a(int i, boolean flag)`
  - `void func_798_a()`
  - `void func_797_a(EntityPlayer entityplayer)`

##### <a id="class-osmap"></a>[`OsMap`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/OsMap.java)
- **Ruta y Paquete**: `net.minecraft.src.OsMap` (`net/minecraft/src/OsMap.java`) | **Líneas de código**: 36
- **Declaración**: `class OsMap`
- **Rol**: Mapeo de opciones y rutas según el sistema operativo detectado.
- **Descripción Técnica**: Proporciona la ruta canónica del directorio de datos (.minecraft / Application Support).
- **Guía de Modificación**: Modificar si se desea reubicar el directorio raíz del juego.

##### <a id="class-panelcrashreport"></a>[`PanelCrashReport`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PanelCrashReport.java)
- **Ruta y Paquete**: `net.minecraft.src.PanelCrashReport` (`net/minecraft/src/PanelCrashReport.java`) | **Líneas de código**: 92
- **Declaración**: `class PanelCrashReport extends Panel`
- **Rol**: Panel contenedor AWT para la presentación de informes de errores.
- **Descripción Técnica**: Muestra los detalles técnicos del fallo y botones para copiar el informe.
- **Guía de Modificación**: Permite añadir información diagnóstica adicional a los informes de error.

##### <a id="class-playercontroller"></a>[`PlayerController`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PlayerController.java)
- **Ruta y Paquete**: `net.minecraft.src.PlayerController` (`net/minecraft/src/PlayerController.java`) | **Líneas de código**: 126
- **Declaración**: `class PlayerController`
- **Rol**: Clase base abstracta para controladores de modo de juego del jugador.
- **Descripción Técnica**: Desacopla la interacción con bloques y mobs según las reglas del modo activo.
- **Guía de Modificación**: Base de PlayerControllerSP y PlayerControllerMP.
- **Métodos Clave Destacados**:
  - `void func_717_a(World world)`
  - `void clickBlock(int i, int j, int k, int l)`
  - `boolean sendBlockRemoved(int i, int j, int k, int l)`
  - `void sendBlockRemoving(int i, int j, int k, int l)`
  - `void func_6468_a()`
  - `void func_6467_a(float f)`
- **Campos Clave Destacados**:
  - `Minecraft mc`
  - `boolean field_1064_b`

##### <a id="class-playercontrollermp"></a>[`PlayerControllerMP`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PlayerControllerMP.java)
- **Ruta y Paquete**: `net.minecraft.src.PlayerControllerMP` (`net/minecraft/src/PlayerControllerMP.java`) | **Líneas de código**: 212
- **Declaración**: `class PlayerControllerMP extends PlayerController`
- **Rol**: Controlador del jugador para partidas multijugador.
- **Descripción Técnica**: Empaqueta las acciones de rotura y colocación en paquetes Packet14BlockDig y Packet15Place para validación en el servidor.
- **Guía de Modificación**: Modificar para implementar acciones predictivas o validaciones cliente en multijugador.
- **Métodos Clave Destacados**:
  - `void flipPlayer(EntityPlayer entityplayer)`
  - `boolean sendBlockRemoved(int i, int j, int k, int l)`
  - `void clickBlock(int i, int j, int k, int l)`
  - `void func_6468_a()`
  - `void sendBlockRemoving(int i, int j, int k, int l)`
  - `void func_6467_a(float f)`

##### <a id="class-playercontrollersp"></a>[`PlayerControllerSP`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PlayerControllerSP.java)
- **Ruta y Paquete**: `net.minecraft.src.PlayerControllerSP` (`net/minecraft/src/PlayerControllerSP.java`) | **Líneas de código**: 147
- **Declaración**: `class PlayerControllerSP extends PlayerController`
- **Rol**: Controlador del modo Supervivencia para un solo jugador.
- **Descripción Técnica**: Gestiona los tiempos de rotura continua de bloques según herramienta, alcance del brazo y colocación inmediata.
- **Guía de Modificación**: Modificar para alterar mecánicas de minado o combate en singleplayer.
- **Métodos Clave Destacados**:
  - `void flipPlayer(EntityPlayer entityplayer)`
  - `boolean sendBlockRemoved(int i, int j, int k, int l)`
  - `void clickBlock(int i, int j, int k, int l)`
  - `void func_6468_a()`
  - `void sendBlockRemoving(int i, int j, int k, int l)`
  - `void func_6467_a(float f)`

##### <a id="class-playercontrollertest"></a>[`PlayerControllerTest`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PlayerControllerTest.java)
- **Ruta y Paquete**: `net.minecraft.src.PlayerControllerTest` (`net/minecraft/src/PlayerControllerTest.java`) | **Líneas de código**: 46
- **Declaración**: `class PlayerControllerTest extends PlayerController`
- **Rol**: Controlador de pruebas experimentales y modo creativo preliminar.
- **Descripción Técnica**: Permite rotura instantánea de bloques y recursos ilimitados para depuración interna.
- **Guía de Modificación**: El precursor del modo creativo en Alpha.
- **Métodos Clave Destacados**:
  - `void func_6473_b(EntityPlayer entityplayer)`
  - `boolean func_6469_d()`
  - `void func_717_a(World world)`
  - `void func_6474_c()`

##### <a id="class-session"></a>[`Session`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Session.java)
- **Ruta y Paquete**: `net.minecraft.src.Session` (`net/minecraft/src/Session.java`) | **Líneas de código**: 57
- **Declaración**: `class Session`
- **Rol**: Almacena las credenciales de la sesión activa del jugador.
- **Descripción Técnica**: Contiene el nombre de usuario ('username') y el ID de sesión ('sessionId') validados ante Minecraft.net.
- **Guía de Modificación**: Modificar si se desea cambiar de identidad en pruebas de desarrollo o implementar autenticación offline.
- **Campos Clave Destacados**:
  - `List registeredBlocksList`
  - `String inventory`
  - `String field_6543_c`
  - `String field_6542_d`

##### <a id="class-threadconnecttoserver"></a>[`ThreadConnectToServer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadConnectToServer.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadConnectToServer` (`net/minecraft/src/ThreadConnectToServer.java`) | **Líneas de código**: 63
- **Declaración**: `class ThreadConnectToServer extends Thread`
- **Rol**: Hilo asíncrono para negociar la conexión con un servidor multijugador.
- **Descripción Técnica**: Abre el socket TCP sin congelar la pantalla de 'Connecting to server...'.
- **Guía de Modificación**: Permite gestionar tiempos de espera (timeouts) en la conexión.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-threaddownloadimage"></a>[`ThreadDownloadImage`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadDownloadImage.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadDownloadImage` (`net/minecraft/src/ThreadDownloadImage.java`) | **Líneas de código**: 55
- **Declaración**: `class ThreadDownloadImage extends Thread`
- **Rol**: Hilo secundario para descargar imágenes de texturas en segundo plano.
- **Descripción Técnica**: Evita pausas en el renderizado mientras se obtienen imágenes remotas.
- **Guía de Modificación**: Utilizado en la descarga de contenidos dinámicos.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-threaddownloadimagedata"></a>[`ThreadDownloadImageData`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadDownloadImageData.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadDownloadImageData` (`net/minecraft/src/ThreadDownloadImageData.java`) | **Líneas de código**: 23
- **Declaración**: `class ThreadDownloadImageData`
- **Rol**: Contenedor y gestor de estado para imágenes descargadas asíncronamente.
- **Descripción Técnica**: Supervisa la carga de bytes y notifica al renderizador cuando la textura está lista para subirse a la GPU.
- **Guía de Modificación**: Maneja la textura por defecto mientras la descarga remota se completa.
- **Campos Clave Destacados**:
  - `BufferedImage image`
  - `int referenceCount`
  - `int textureName`
  - `boolean textureSetupComplete`

##### <a id="class-threaddownloadresources"></a>[`ThreadDownloadResources`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadDownloadResources.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadDownloadResources` (`net/minecraft/src/ThreadDownloadResources.java`) | **Líneas de código**: 163
- **Declaración**: `class ThreadDownloadResources extends Thread`
- **Rol**: Hilo en segundo plano para descargar la música y efectos de sonido oficiales.
- **Descripción Técnica**: Descarga canciones de C418 y efectos sonoros desde los servidores de recursos de Mojang en la carpeta .minecraft/resources.
- **Guía de Modificación**: Controlar si se desean precargar sonidos locales en un mod.
- **Métodos Clave Destacados**:
  - `void run()`
  - `void reloadResources()`
  - `void closeMinecraft()`
- **Campos Clave Destacados**:
  - `File resourcesFolder`

##### <a id="class-threadrunisoclient"></a>[`ThreadRunIsoClient`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadRunIsoClient.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadRunIsoClient` (`net/minecraft/src/ThreadRunIsoClient.java`) | **Líneas de código**: 29
- **Declaración**: `class ThreadRunIsoClient extends Thread`
- **Rol**: Hilo de procesamiento para la generación del mapa isométrico.
- **Descripción Técnica**: Coordina el renderizado continuo del visor ortogonal.
- **Guía de Modificación**: Permite generar mapas en segundo plano.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-threadsleepforever"></a>[`ThreadSleepForever`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ThreadSleepForever.java)
- **Ruta y Paquete**: `net.minecraft.src.ThreadSleepForever` (`net/minecraft/src/ThreadSleepForever.java`) | **Líneas de código**: 32
- **Declaración**: `class ThreadSleepForever extends Thread`
- **Rol**: Hilo utilitario que mantiene un hilo durmiendo indefinidamente con Thread.sleep().
- **Descripción Técnica**: Utilizado como mecanismo de sincronización para mantener vivos hilos demonio en segundo plano.
- **Guía de Modificación**: Utilidad interna de concurrencia.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-timer"></a>[`Timer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Timer.java)
- **Ruta y Paquete**: `net.minecraft.src.Timer` (`net/minecraft/src/Timer.java`) | **Líneas de código**: 68
- **Declaración**: `class Timer`
- **Rol**: Sincronizador del reloj de juego (20 TPS) desacoplado de la tasa de cuadros (FPS).
- **Descripción Técnica**: Mide los nanosegundos transcurridos para disparar exactamente 20 ticks lógicos por segundo mientras interpola 'renderPartialTicks' para que las animaciones a altos FPS sean fluidas.
- **Guía de Modificación**: Ajusta timerSpeed para crear efectos de cámara lenta (slow-motion) o aceleración temporal.
- **Métodos Clave Destacados**:
  - `void updateTimer()`
- **Campos Clave Destacados**:
  - `float ticksPerSecond`
  - `int elapsedTicks`
  - `float renderPartialTicks`
  - `float timerSpeed`
  - `float elapsedPartialTicks`

##### <a id="class-unexpectedthrowable"></a>[`UnexpectedThrowable`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/UnexpectedThrowable.java)
- **Ruta y Paquete**: `net.minecraft.src.UnexpectedThrowable` (`net/minecraft/src/UnexpectedThrowable.java`) | **Líneas de código**: 18
- **Declaración**: `class UnexpectedThrowable`
- **Rol**: Envoltorio estructurado para excepciones imprevistas.
- **Descripción Técnica**: Almacena la descripción del fallo y el objeto Throwable para generar el volcado del Crash Report.
- **Guía de Modificación**: Utilizado para reportar fallos de forma amigable.
- **Campos Clave Destacados**:
  - `String description`
  - `Throwable exception`


### 02. Bloques y Materiales (Blocks, Materials & Mechanics)
> **Resumen del subsistema**: El sistema de construcción y mundo voxel. Define los 256 IDs posibles de bloques, sus propiedades físicas (dureza, resistencia a explosiones, opacidad y emisión lumínica), mecánicas interactivas, gravedad, propagación de fluidos y redstone.

**Total de clases**: 66

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`Block`](#class-block) | `class` | `-` | 719 | Clase base abstracta de todos los bloques en el mundo voxel. |
| [`BlockBloodStone`](#class-blockbloodstone) | `class` | `ext Block` | 14 | Netherrack / Piedra del Nether que mantiene el fuego perpetuo. |
| [`BlockBookshelf`](#class-blockbookshelf) | `class` | `ext Block` | 31 | Manejador del bloque 'BlockBookshelf' en el mundo voxel. |
| [`BlockBreakable`](#class-blockbreakable) | `class` | `ext Block` | 34 | Manejador del bloque 'BlockBreakable' en el mundo voxel. |
| [`BlockButton`](#class-blockbutton) | `class` | `ext Block` | 329 | Manejador del bloque 'BlockButton' en el mundo voxel. |
| [`BlockCactus`](#class-blockcactus) | `class` | `ext Block` | 128 | Manejador del bloque de cactus con daño por espinas y crecimiento... |
| [`BlockChest`](#class-blockchest) | `class` | `ext BlockContainer` | 273 | Bloque de cofre para almacenamiento persistente de objetos. |
| [`BlockClay`](#class-blockclay) | `class` | `ext Block` | 25 | Manejador del bloque 'BlockClay' en el mundo voxel. |
| [`BlockContainer`](#class-blockcontainer) | `class` | `ext Block` | 34 | Manejador del bloque 'BlockContainer' en el mundo voxel. |
| [`BlockCrops`](#class-blockcrops) | `class` | `ext BlockFlower` | 138 | Cultivo de trigo por etapas de crecimiento (de 0 a 7). |
| [`BlockDirt`](#class-blockdirt) | `class` | `ext Block` | 14 | Manejador del bloque 'BlockDirt' en el mundo voxel. |
| [`BlockDoor`](#class-blockdoor) | `class` | `ext Block` | 250 | Mecánica de puertas de 2 bloques de altura con bisagras y apertur... |
| [`BlockFence`](#class-blockfence) | `class` | `ext Block` | 50 | Manejador del bloque 'BlockFence' en el mundo voxel. |
| [`BlockFire`](#class-blockfire) | `class` | `ext Block` | 317 | Sistema de propagación de fuego, combustión y extinción. |
| [`BlockFlower`](#class-blockflower) | `class` | `ext Block` | 74 | Manejador del bloque 'BlockFlower' en el mundo voxel. |
| [`BlockFlowing`](#class-blockflowing) | `class` | `ext BlockFluids` | 322 | Manejador del bloque 'BlockFlowing' en el mundo voxel. |
| [`BlockFluids`](#class-blockfluids) | `class` | `ext Block` | 349 | Manejador del bloque 'BlockFluids' en el mundo voxel. |
| [`BlockFurnace`](#class-blockfurnace) | `class` | `ext BlockContainer` | 181 | Bloque de horno para fundición de minerales y cocinado de comida. |
| [`BlockGlass`](#class-blockglass) | `class` | `ext BlockBreakable` | 20 | Manejador del bloque 'BlockGlass' en el mundo voxel. |
| [`BlockGrass`](#class-blockgrass) | `class` | `ext Block` | 66 | Bloque de césped con propagación y degradación a tierra. |
| [`BlockGravel`](#class-blockgravel) | `class` | `ext BlockSand` | 26 | Manejador del bloque 'BlockGravel' en el mundo voxel. |
| [`BlockIce`](#class-blockice) | `class` | `ext BlockBreakable` | 50 | Manejador del bloque 'BlockIce' en el mundo voxel. |
| [`BlockJukeBox`](#class-blockjukebox) | `class` | `ext Block` | 60 | Manejador del bloque 'BlockJukeBox' en el mundo voxel. |
| [`BlockLadder`](#class-blockladder) | `class` | `ext Block` | 148 | Manejador del bloque 'BlockLadder' en el mundo voxel. |
| [`BlockLeaves`](#class-blockleaves) | `class` | `ext BlockLeavesBase` | 170 | Bloque de hojas de árbol con descomposición por distancia al tron... |
| [`BlockLeavesBase`](#class-blockleavesbase) | `class` | `ext Block` | 34 | Manejador del bloque 'BlockLeavesBase' en el mundo voxel. |
| [`BlockLever`](#class-blocklever) | `class` | `ext Block` | 289 | Manejador del bloque 'BlockLever' en el mundo voxel. |
| [`BlockLightStone`](#class-blocklightstone) | `class` | `ext Block` | 20 | Glowstone / Piedra luminosa del Nether. |
| [`BlockLog`](#class-blocklog) | `class` | `ext Block` | 35 | Bloque de tronco / madera natural de árbol. |
| [`BlockMinecartTrack`](#class-blockminecarttrack) | `class` | `ext Block` | 135 | Manejador del bloque 'BlockMinecartTrack' en el mundo voxel. |
| [`BlockMobSpawner`](#class-blockmobspawner) | `class` | `ext BlockContainer` | 35 | Manejador del bloque 'BlockMobSpawner' en el mundo voxel. |
| [`BlockMushroom`](#class-blockmushroom) | `class` | `ext BlockFlower` | 26 | Manejador del bloque 'BlockMushroom' en el mundo voxel. |
| [`BlockObsidian`](#class-blockobsidian) | `class` | `ext BlockStone` | 25 | Manejador del bloque 'BlockObsidian' en el mundo voxel. |
| [`BlockOre`](#class-blockore) | `class` | `ext Block` | 35 | Manejador del bloque 'BlockOre' en el mundo voxel. |
| [`BlockOreBlock`](#class-blockoreblock) | `class` | `ext Block` | 20 | Manejador del bloque 'BlockOreBlock' en el mundo voxel. |
| [`BlockPortal`](#class-blockportal) | `class` | `ext BlockBreakable` | 207 | Bloque de portal dimensional al Inframundo (Nether). |
| [`BlockPressurePlate`](#class-blockpressureplate) | `class` | `ext Block` | 194 | Manejador del bloque 'BlockPressurePlate' en el mundo voxel. |
| [`BlockPumpkin`](#class-blockpumpkin) | `class` | `ext Block` | 91 | Manejador del bloque 'BlockPumpkin' en el mundo voxel. |
| [`BlockRedstoneOre`](#class-blockredstoneore) | `class` | `ext Block` | 121 | Manejador del bloque 'BlockRedstoneOre' en el mundo voxel. |
| [`BlockRedstoneTorch`](#class-blockredstonetorch) | `class` | `ext BlockTorch` | 224 | Antorcha de redstone que actúa como inversor lógico NOT y fuente ... |
| [`BlockRedstoneWire`](#class-blockredstonewire) | `class` | `ext Block` | 402 | Cable conductor de energía de redstone con atenuación de 15 a 0. |
| [`BlockReed`](#class-blockreed) | `class` | `ext Block` | 110 | Manejador del bloque 'BlockReed' en el mundo voxel. |
| [`BlockSand`](#class-blocksand) | `class` | `ext Block` | 78 | Manejador del bloque 'BlockSand' en el mundo voxel. |
| [`BlockSapling`](#class-blocksapling) | `class` | `ext BlockFlower` | 42 | Manejador del bloque 'BlockSapling' en el mundo voxel. |
| [`BlockSign`](#class-blocksign) | `class` | `ext BlockContainer` | 136 | Manejador del bloque 'BlockSign' en el mundo voxel. |
| [`BlockSlowSand`](#class-blockslowsand) | `class` | `ext Block` | 26 | Soul Sand / Arena de almas del Nether con reducción de velocidad. |
| [`BlockSnow`](#class-blocksnow) | `class` | `ext Block` | 110 | Manejador del bloque 'BlockSnow' en el mundo voxel. |
| [`BlockSnowBlock`](#class-blocksnowblock) | `class` | `ext Block` | 35 | Manejador del bloque 'BlockSnowBlock' en el mundo voxel. |
| [`BlockSoil`](#class-blocksoil) | `class` | `ext Block` | 133 | Manejador del bloque 'BlockSoil' en el mundo voxel. |
| [`BlockSponge`](#class-blocksponge) | `class` | `ext Block` | 51 | Manejador del bloque 'BlockSponge' en el mundo voxel. |
| [`BlockStairs`](#class-blockstairs) | `class` | `ext Block` | 233 | Manejador del bloque 'BlockStairs' en el mundo voxel. |
| [`BlockStationary`](#class-blockstationary) | `class` | `ext BlockFluids` | 73 | Manejador del bloque 'BlockStationary' en el mundo voxel. |
| [`BlockStep`](#class-blockstep) | `class` | `ext Block` | 91 | Manejador del bloque 'BlockStep' en el mundo voxel. |
| [`BlockStone`](#class-blockstone) | `class` | `ext Block` | 20 | Manejador del bloque 'BlockStone' en el mundo voxel. |
| [`BlockTNT`](#class-blocktnt) | `class` | `ext Block` | 65 | Bloque de dinamita activable por golpe o redstone. |
| [`BlockTorch`](#class-blocktorch) | `class` | `ext Block` | 225 | Manejador del bloque 'BlockTorch' en el mundo voxel. |
| [`BlockWorkbench`](#class-blockworkbench) | `class` | `ext Block` | 40 | Mesa de trabajo para crafteo en cuadrícula extendida de 3x3. |
| [`Material`](#class-material) | `class` | `-` | 71 | Definición de propiedades intrínsecas de la materia en el mundo d... |
| [`MaterialLiquid`](#class-materialliquid) | `class` | `ext Material` | 28 | Especialización de Material para fluidos (agua y lava). |
| [`MaterialLogic`](#class-materiallogic) | `class` | `ext Material` | 28 | Especialización de Material para componentes lógicos y circuitos. |
| [`MaterialTransparent`](#class-materialtransparent) | `class` | `ext Material` | 28 | Especialización de Material para bloques transparentes que no ocl... |
| [`MinecartTrackLogic`](#class-minecarttracklogic) | `class` | `-` | 405 | Lógica geométrica de conexión y curvatura de raíles de vagoneta. |
| [`RedstoneUpdateInfo`](#class-redstoneupdateinfo) | `class` | `-` | 22 | Registro de propagación para limitar actualizaciones cíclicas de ... |
| [`StepSound`](#class-stepsound) | `class` | `-` | 40 | Efectos sonoros emitidos al caminar, colocar o destruir bloques. |
| [`StepSoundSand`](#class-stepsoundsand) | `class` | `ext StepSound` | 19 | Variante de StepSound para arena, grava y nieve. |
| [`StepSoundStone`](#class-stepsoundstone) | `class` | `ext StepSound` | 19 | Variante de StepSound para piedra, minerales y adoquín. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-block"></a>[`Block`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Block.java)
- **Ruta y Paquete**: `net.minecraft.src.Block` (`net/minecraft/src/Block.java`) | **Líneas de código**: 719
- **Declaración**: `class Block`
- **Rol**: Clase base abstracta de todos los bloques en el mundo voxel.
- **Descripción Técnica**: Define el registro global 'blocksList[256]', propiedades de opacidad, resistencia a explosiones, dureza de minado (blockHardness), emisión de luz y sonido. Gestiona el ciclo de vida (onBlockAdded, onBlockRemoval), ticks periódicos (updateTick), cajas AABB de colisión y selección, y drops de ítems.
- **Guía de Modificación**: Para añadir un bloque nuevo, se instancia una subclase con un ID libre menor a 256 en blocksList. Para modificar la velocidad de rotura o drops globales, se tocan sus métodos base.
- **Métodos Clave Destacados**:
  - `Block setStepSound(StepSound stepsound)`
  - `Block setLightOpacity(int i)`
  - `Block setLightValue(float f)`
  - `Block setResistance(float f)`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
- **Campos Clave Destacados**:
  - `StepSound soundPowderFootstep`
  - `StepSound soundWoodFootstep`
  - `StepSound soundGravelFootstep`
  - `StepSound soundGrassFootstep`
  - `StepSound soundStoneFootstep`
  - `StepSound soundMetalFootstep`

##### <a id="class-blockbloodstone"></a>[`BlockBloodStone`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockBloodStone.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockBloodStone` (`net/minecraft/src/BlockBloodStone.java`) | **Líneas de código**: 14
- **Declaración**: `class BlockBloodStone extends Block`
- **Rol**: Netherrack / Piedra del Nether que mantiene el fuego perpetuo.
- **Descripción Técnica**: El bloque geológico base del Inframundo. Tiene baja dureza de minado y la propiedad única de no extinguir jamás el fuego que arda en su cara superior.
- **Guía de Modificación**: Ideal como plantilla para bloques con propiedades pirotécnicas o de dimensiones alternativas.

##### <a id="class-blockbookshelf"></a>[`BlockBookshelf`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockBookshelf.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockBookshelf` (`net/minecraft/src/BlockBookshelf.java`) | **Líneas de código**: 31
- **Declaración**: `class BlockBookshelf extends Block`
- **Rol**: Manejador del bloque 'BlockBookshelf' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockBookshelf'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`
  - `int quantityDropped(Random random)`

##### <a id="class-blockbreakable"></a>[`BlockBreakable`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockBreakable.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockBreakable` (`net/minecraft/src/BlockBreakable.java`) | **Líneas de código**: 34
- **Declaración**: `class BlockBreakable extends Block`
- **Rol**: Manejador del bloque 'BlockBreakable' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockBreakable'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `boolean isOpaqueCube()`
  - `boolean shouldSideBeRendered(IBlockAccess iblockaccess, int i, int j, int k, int l)`

##### <a id="class-blockbutton"></a>[`BlockButton`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockButton.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockButton` (`net/minecraft/src/BlockButton.java`) | **Líneas de código**: 329
- **Declaración**: `class BlockButton extends Block`
- **Rol**: Manejador del bloque 'BlockButton' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockButton'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `int tickRate()`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockPlaced(World world, int i, int j, int k, int l)`

##### <a id="class-blockcactus"></a>[`BlockCactus`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockCactus.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockCactus` (`net/minecraft/src/BlockCactus.java`) | **Líneas de código**: 128
- **Declaración**: `class BlockCactus extends Block`
- **Rol**: Manejador del bloque de cactus con daño por espinas y crecimiento sobre arena.
- **Descripción Técnica**: Controla el comportamiento del cactus. Requiere arena inferior como apoyo y ningún bloque sólido vecino en las caras norte, sur, este u oeste. Crece verticalmente en updateTick hasta 3 bloques de altura e inflige 1 punto de daño en onEntityCollidedWithBlock a quien toque su caja reducida.
- **Guía de Modificación**: Modifica onEntityCollidedWithBlock para alterar el daño de espinas, o canBlockStay() para permitir que crezca sobre tierra u otros materiales.
- **Métodos Clave Destacados**:
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `AxisAlignedBB getSelectedBoundingBoxFromPool(World world, int i, int j, int k)`
  - `int getBlockTextureFromSide(int i)`
  - `boolean renderAsNormalBlock()`
  - `boolean isOpaqueCube()`

##### <a id="class-blockchest"></a>[`BlockChest`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockChest.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockChest` (`net/minecraft/src/BlockChest.java`) | **Líneas de código**: 273
- **Declaración**: `class BlockChest extends BlockContainer`
- **Rol**: Bloque de cofre para almacenamiento persistente de objetos.
- **Descripción Técnica**: Hereda de BlockContainer y se asocia a TileEntityChest. Gestiona la detección de cofres contiguos para formar un 'cofre doble' de 54 casillas, bloquea la apertura si hay un bloque opaco encima, y expulsa todos los ítems contenidos al destruirse.
- **Guía de Modificación**: Para crear cofres con mayor capacidad o con restricciones de acceso, se modifica esta clase junto con TileEntityChest y ContainerChest.
- **Métodos Clave Destacados**:
  - `int getBlockTexture(IBlockAccess iblockaccess, int i, int j, int k, int l)`
  - `int getBlockTextureFromSide(int i)`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockRemoval(World world, int i, int j, int k)`
  - `boolean blockActivated(World world, int i, int j, int k, EntityPlayer entityplayer)`
  - `TileEntity SetBlockEntity()`

##### <a id="class-blockclay"></a>[`BlockClay`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockClay.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockClay` (`net/minecraft/src/BlockClay.java`) | **Líneas de código**: 25
- **Declaración**: `class BlockClay extends Block`
- **Rol**: Manejador del bloque 'BlockClay' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockClay'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`
  - `int quantityDropped(Random random)`

##### <a id="class-blockcontainer"></a>[`BlockContainer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockContainer.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockContainer` (`net/minecraft/src/BlockContainer.java`) | **Líneas de código**: 34
- **Declaración**: `class BlockContainer extends Block`
- **Rol**: Manejador del bloque 'BlockContainer' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockContainer'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `void onBlockRemoval(World world, int i, int j, int k)`

##### <a id="class-blockcrops"></a>[`BlockCrops`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockCrops.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockCrops` (`net/minecraft/src/BlockCrops.java`) | **Líneas de código**: 138
- **Declaración**: `class BlockCrops extends BlockFlower`
- **Rol**: Cultivo de trigo por etapas de crecimiento (de 0 a 7).
- **Descripción Técnica**: Requiere tierra arada (BlockFarmland) inferior e hidratación. En updateTick, si la luz es >= 9, avanza sus metadatos hasta la etapa madura 7, donde suelta trigo y semillas.
- **Guía de Modificación**: Ajusta la velocidad de crecimiento en updateTick o añade nuevos tipos de semillas y hortalizas.
- **Métodos Clave Destacados**:
  - `boolean canThisPlantGrowOnThisBlockID(int i)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `int getRenderType()`
  - `void onBlockDestroyedByPlayer(World world, int i, int j, int k, int l)`
  - `int idDropped(int i, Random random)`

##### <a id="class-blockdirt"></a>[`BlockDirt`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockDirt.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockDirt` (`net/minecraft/src/BlockDirt.java`) | **Líneas de código**: 14
- **Declaración**: `class BlockDirt extends Block`
- **Rol**: Manejador del bloque 'BlockDirt' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockDirt'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.

##### <a id="class-blockdoor"></a>[`BlockDoor`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockDoor.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockDoor` (`net/minecraft/src/BlockDoor.java`) | **Líneas de código**: 250
- **Declaración**: `class BlockDoor extends Block`
- **Rol**: Mecánica de puertas de 2 bloques de altura con bisagras y apertura.
- **Descripción Técnica**: Controla puertas de madera y hierro. Ocupa dos bloques verticales sincronizados mediante metadatos (mitad inferior y superior). Gestiona la apertura manual por clic o eléctrica por señal de redstone.
- **Guía de Modificación**: Permite crear puertas de nuevos materiales ajustando canBlockStay y onPoweredBlockChange.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `AxisAlignedBB getSelectedBoundingBoxFromPool(World world, int i, int j, int k)`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`

##### <a id="class-blockfence"></a>[`BlockFence`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFence.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFence` (`net/minecraft/src/BlockFence.java`) | **Líneas de código**: 50
- **Declaración**: `class BlockFence extends Block`
- **Rol**: Manejador del bloque 'BlockFence' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockFence'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void getCollidingBoundingBoxes(World world, int i, int j, int k, AxisAlignedBB axisalignedbb, ArrayList arraylist)`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`

##### <a id="class-blockfire"></a>[`BlockFire`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFire.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFire` (`net/minecraft/src/BlockFire.java`) | **Líneas de código**: 317
- **Declaración**: `class BlockFire extends Block`
- **Rol**: Sistema de propagación de fuego, combustión y extinción.
- **Descripción Técnica**: Controla la física destructiva del fuego. Mantiene tablas de inflamabilidad (chanceToEncourageFire) y combustibilidad (abilityToCatchFire). En updateTick propaga llamas a vecinos combustibles, calcula la extinción por lluvia o falta de combustible, y arde indefinidamente si la base es bloodStone (Netherrack).
- **Guía de Modificación**: Añade nuevos bloques a setBurnRate() para hacerlos inflamables, o ajusta la velocidad de propagación en updateTick().
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `int quantityDropped(Random random)`
  - `int tickRate()`

##### <a id="class-blockflower"></a>[`BlockFlower`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFlower.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFlower` (`net/minecraft/src/BlockFlower.java`) | **Líneas de código**: 74
- **Declaración**: `class BlockFlower extends Block`
- **Rol**: Manejador del bloque 'BlockFlower' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockFlower'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `boolean canThisPlantGrowOnThisBlockID(int i)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `void func_268_h(World world, int i, int j, int k)`
  - `boolean canBlockStay(World world, int i, int j, int k)`

##### <a id="class-blockflowing"></a>[`BlockFlowing`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFlowing.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFlowing` (`net/minecraft/src/BlockFlowing.java`) | **Líneas de código**: 322
- **Declaración**: `class BlockFlowing extends BlockFluids`
- **Rol**: Manejador del bloque 'BlockFlowing' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockFlowing'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int func_296_f(World world, int i, int j, int k, int l)`
  - `void onBlockAdded(World world, int i, int j, int k)`

##### <a id="class-blockfluids"></a>[`BlockFluids`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFluids.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFluids` (`net/minecraft/src/BlockFluids.java`) | **Líneas de código**: 349
- **Declaración**: `class BlockFluids extends Block`
- **Rol**: Manejador del bloque 'BlockFluids' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockFluids'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `float func_288_b(int i)`
  - `int getBlockTextureFromSide(int i)`
  - `int func_290_h(World world, int i, int j, int k)`
  - `int func_289_b(IBlockAccess iblockaccess, int i, int j, int k)`
  - `boolean renderAsNormalBlock()`
  - `boolean isOpaqueCube()`

##### <a id="class-blockfurnace"></a>[`BlockFurnace`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockFurnace.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockFurnace` (`net/minecraft/src/BlockFurnace.java`) | **Líneas de código**: 181
- **Declaración**: `class BlockFurnace extends BlockContainer`
- **Rol**: Bloque de horno para fundición de minerales y cocinado de comida.
- **Descripción Técnica**: Representa tanto el horno inactivo como el activo con fuego (usando dos IDs de bloque distintos para alternar la textura iluminada frontal). Es un BlockContainer asociado a TileEntityFurnace. Al interactuar con él (blockActivated), abre la interfaz de fundición y suelta sus contenidos al destruirse.
- **Guía de Modificación**: Para cambiar la interfaz o la rotación al colocarse, se modifican blockActivated y onBlockPlaced.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `int getBlockTexture(IBlockAccess iblockaccess, int i, int j, int k, int l)`
  - `void randomDisplayTick(World world, int i, int j, int k, Random random)`
  - `int getBlockTextureFromSide(int i)`
  - `boolean blockActivated(World world, int i, int j, int k, EntityPlayer entityplayer)`

##### <a id="class-blockglass"></a>[`BlockGlass`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockGlass.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockGlass` (`net/minecraft/src/BlockGlass.java`) | **Líneas de código**: 20
- **Declaración**: `class BlockGlass extends BlockBreakable`
- **Rol**: Manejador del bloque 'BlockGlass' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockGlass'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int quantityDropped(Random random)`

##### <a id="class-blockgrass"></a>[`BlockGrass`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockGrass.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockGrass` (`net/minecraft/src/BlockGrass.java`) | **Líneas de código**: 66
- **Declaración**: `class BlockGrass extends Block`
- **Rol**: Bloque de césped con propagación y degradación a tierra.
- **Descripción Técnica**: Define el césped superior del terreno. En updateTick, si la luz cae por debajo de 4 y tiene un bloque opaco encima, se transforma en tierra (Block.dirt). Si la luz es suficiente (>= 9), se propaga aleatoriamente a bloques de tierra adyacentes. Modula su color con ColorizerGrass.
- **Guía de Modificación**: Modifica updateTick para cambiar la velocidad de propagación o getBlockTexture para personalizar su textura lateral/superior.
- **Métodos Clave Destacados**:
  - `int getBlockTexture(IBlockAccess iblockaccess, int i, int j, int k, int l)`
  - `int colorMultiplier(IBlockAccess iblockaccess, int i, int j, int k)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int idDropped(int i, Random random)`

##### <a id="class-blockgravel"></a>[`BlockGravel`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockGravel.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockGravel` (`net/minecraft/src/BlockGravel.java`) | **Líneas de código**: 26
- **Declaración**: `class BlockGravel extends BlockSand`
- **Rol**: Manejador del bloque 'BlockGravel' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockGravel'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`

##### <a id="class-blockice"></a>[`BlockIce`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockIce.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockIce` (`net/minecraft/src/BlockIce.java`) | **Líneas de código**: 50
- **Declaración**: `class BlockIce extends BlockBreakable`
- **Rol**: Manejador del bloque 'BlockIce' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockIce'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int func_234_g()`
  - `boolean shouldSideBeRendered(IBlockAccess iblockaccess, int i, int j, int k, int l)`
  - `void onBlockRemoval(World world, int i, int j, int k)`
  - `int quantityDropped(Random random)`
  - `void updateTick(World world, int i, int j, int k, Random random)`

##### <a id="class-blockjukebox"></a>[`BlockJukeBox`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockJukeBox.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockJukeBox` (`net/minecraft/src/BlockJukeBox.java`) | **Líneas de código**: 60
- **Declaración**: `class BlockJukeBox extends Block`
- **Rol**: Manejador del bloque 'BlockJukeBox' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockJukeBox'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`
  - `boolean blockActivated(World world, int i, int j, int k, EntityPlayer entityplayer)`
  - `void ejectRecord(World world, int i, int j, int k, int l)`
  - `void dropBlockAsItemWithChance(World world, int i, int j, int k, int l, float f)`

##### <a id="class-blockladder"></a>[`BlockLadder`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLadder.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLadder` (`net/minecraft/src/BlockLadder.java`) | **Líneas de código**: 148
- **Declaración**: `class BlockLadder extends Block`
- **Rol**: Manejador del bloque 'BlockLadder' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockLadder'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `AxisAlignedBB getSelectedBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`

##### <a id="class-blockleaves"></a>[`BlockLeaves`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLeaves.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLeaves` (`net/minecraft/src/BlockLeaves.java`) | **Líneas de código**: 170
- **Declaración**: `class BlockLeaves extends BlockLeavesBase`
- **Rol**: Bloque de hojas de árbol con descomposición por distancia al tronco.
- **Descripción Técnica**: Gestiona las hojas de los árboles con soporte para gráficos 'Fancy' (transparentes) y 'Fast' (opacos). Comprueba si existe un bloque de madera (Block.wood) en un radio de 4 bloques; si no lo halla, decaen y tienen probabilidad de soltar retoños (Block.sapling).
- **Guía de Modificación**: Ajusta la probabilidad de drop de saplings en idDropped/quantityDropped, o amplía la distancia de soporte para árboles colosales.
- **Métodos Clave Destacados**:
  - `int colorMultiplier(IBlockAccess iblockaccess, int i, int j, int k)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void func_6361_f(World world, int i, int j, int k, int l)`
  - `void func_308_h(World world, int i, int j, int k)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int quantityDropped(Random random)`

##### <a id="class-blockleavesbase"></a>[`BlockLeavesBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLeavesBase.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLeavesBase` (`net/minecraft/src/BlockLeavesBase.java`) | **Líneas de código**: 34
- **Declaración**: `class BlockLeavesBase extends Block`
- **Rol**: Manejador del bloque 'BlockLeavesBase' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockLeavesBase'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `boolean isOpaqueCube()`
  - `boolean shouldSideBeRendered(IBlockAccess iblockaccess, int i, int j, int k, int l)`
- **Campos Clave Destacados**:
  - `boolean graphicsLevel`

##### <a id="class-blocklever"></a>[`BlockLever`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLever.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLever` (`net/minecraft/src/BlockLever.java`) | **Líneas de código**: 289
- **Declaración**: `class BlockLever extends Block`
- **Rol**: Manejador del bloque 'BlockLever' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockLever'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockPlaced(World world, int i, int j, int k, int l)`

##### <a id="class-blocklightstone"></a>[`BlockLightStone`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLightStone.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLightStone` (`net/minecraft/src/BlockLightStone.java`) | **Líneas de código**: 20
- **Declaración**: `class BlockLightStone extends Block`
- **Rol**: Glowstone / Piedra luminosa del Nether.
- **Descripción Técnica**: Bloque frágil que emite el nivel máximo de luz (15) y se genera colgado de los techos del Nether. Al destruirse suelta polvo de piedra brillante.
- **Guía de Modificación**: Cambia idDropped y quantityDropped para alterar los recursos obtenidos al picarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`

##### <a id="class-blocklog"></a>[`BlockLog`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockLog.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockLog` (`net/minecraft/src/BlockLog.java`) | **Líneas de código**: 35
- **Declaración**: `class BlockLog extends Block`
- **Rol**: Bloque de tronco / madera natural de árbol.
- **Descripción Técnica**: Representa los troncos de madera sólida en el mundo. Proporciona madera talable que sirve de sustento a las hojas circundantes. Mapea la textura circular en las caras superior e inferior y la corteza en los laterales.
- **Guía de Modificación**: Modifica getBlockTextureFromSide para añadir orientaciones de colocación o nuevos tipos de madera.
- **Métodos Clave Destacados**:
  - `int quantityDropped(Random random)`
  - `int idDropped(int i, Random random)`
  - `int getBlockTextureFromSide(int i)`

##### <a id="class-blockminecarttrack"></a>[`BlockMinecartTrack`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockMinecartTrack.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockMinecartTrack` (`net/minecraft/src/BlockMinecartTrack.java`) | **Líneas de código**: 135
- **Declaración**: `class BlockMinecartTrack extends Block`
- **Rol**: Manejador del bloque 'BlockMinecartTrack' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockMinecartTrack'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `MovingObjectPosition collisionRayTrace(World world, int i, int j, int k, Vec3D vec3d, Vec3D vec3d1)`
  - `void setBlockBoundsBasedOnState(IBlockAccess iblockaccess, int i, int j, int k)`
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `boolean renderAsNormalBlock()`

##### <a id="class-blockmobspawner"></a>[`BlockMobSpawner`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockMobSpawner.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockMobSpawner` (`net/minecraft/src/BlockMobSpawner.java`) | **Líneas de código**: 35
- **Declaración**: `class BlockMobSpawner extends BlockContainer`
- **Rol**: Manejador del bloque 'BlockMobSpawner' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockMobSpawner'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `TileEntity SetBlockEntity()`
  - `int idDropped(int i, Random random)`
  - `int quantityDropped(Random random)`
  - `boolean isOpaqueCube()`

##### <a id="class-blockmushroom"></a>[`BlockMushroom`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockMushroom.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockMushroom` (`net/minecraft/src/BlockMushroom.java`) | **Líneas de código**: 26
- **Declaración**: `class BlockMushroom extends BlockFlower`
- **Rol**: Manejador del bloque 'BlockMushroom' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockMushroom'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `boolean canThisPlantGrowOnThisBlockID(int i)`
  - `boolean canBlockStay(World world, int i, int j, int k)`

##### <a id="class-blockobsidian"></a>[`BlockObsidian`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockObsidian.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockObsidian` (`net/minecraft/src/BlockObsidian.java`) | **Líneas de código**: 25
- **Declaración**: `class BlockObsidian extends BlockStone`
- **Rol**: Manejador del bloque 'BlockObsidian' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockObsidian'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int quantityDropped(Random random)`
  - `int idDropped(int i, Random random)`

##### <a id="class-blockore"></a>[`BlockOre`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockOre.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockOre` (`net/minecraft/src/BlockOre.java`) | **Líneas de código**: 35
- **Declaración**: `class BlockOre extends Block`
- **Rol**: Manejador del bloque 'BlockOre' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockOre'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`
  - `int quantityDropped(Random random)`

##### <a id="class-blockoreblock"></a>[`BlockOreBlock`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockOreBlock.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockOreBlock` (`net/minecraft/src/BlockOreBlock.java`) | **Líneas de código**: 20
- **Declaración**: `class BlockOreBlock extends Block`
- **Rol**: Manejador del bloque 'BlockOreBlock' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockOreBlock'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`

##### <a id="class-blockportal"></a>[`BlockPortal`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockPortal.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockPortal` (`net/minecraft/src/BlockPortal.java`) | **Líneas de código**: 207
- **Declaración**: `class BlockPortal extends BlockBreakable`
- **Rol**: Bloque de portal dimensional al Inframundo (Nether).
- **Descripción Técnica**: Representa la lámina púrpura del portal dentro de un marco de obsidiana. Al colisionar una entidad con él, acumula tiempo para teletransportarla al Nether o de vuelta al Overworld usando Teleporter.
- **Guía de Modificación**: Permite crear dimensiones adicionales personalizadas cambiando el teletransporte en onEntityCollidedWithBlock.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `void setBlockBoundsBasedOnState(IBlockAccess iblockaccess, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `boolean tryToCreatePortal(World world, int i, int j, int k)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`

##### <a id="class-blockpressureplate"></a>[`BlockPressurePlate`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockPressurePlate.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockPressurePlate` (`net/minecraft/src/BlockPressurePlate.java`) | **Líneas de código**: 194
- **Declaración**: `class BlockPressurePlate extends Block`
- **Rol**: Manejador del bloque 'BlockPressurePlate' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockPressurePlate'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int tickRate()`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockAdded(World world, int i, int j, int k)`

##### <a id="class-blockpumpkin"></a>[`BlockPumpkin`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockPumpkin.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockPumpkin` (`net/minecraft/src/BlockPumpkin.java`) | **Líneas de código**: 91
- **Declaración**: `class BlockPumpkin extends Block`
- **Rol**: Manejador del bloque 'BlockPumpkin' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockPumpkin'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `int getBlockTextureFromSide(int i)`
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockPlacedBy(World world, int i, int j, int k, EntityLiving entityliving)`

##### <a id="class-blockredstoneore"></a>[`BlockRedstoneOre`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockRedstoneOre.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockRedstoneOre` (`net/minecraft/src/BlockRedstoneOre.java`) | **Líneas de código**: 121
- **Declaración**: `class BlockRedstoneOre extends Block`
- **Rol**: Manejador del bloque 'BlockRedstoneOre' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockRedstoneOre'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int tickRate()`
  - `void onBlockClicked(World world, int i, int j, int k, EntityPlayer entityplayer)`
  - `void onEntityWalking(World world, int i, int j, int k, Entity entity)`
  - `boolean blockActivated(World world, int i, int j, int k, EntityPlayer entityplayer)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int idDropped(int i, Random random)`

##### <a id="class-blockredstonetorch"></a>[`BlockRedstoneTorch`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockRedstoneTorch.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockRedstoneTorch` (`net/minecraft/src/BlockRedstoneTorch.java`) | **Líneas de código**: 224
- **Declaración**: `class BlockRedstoneTorch extends BlockTorch`
- **Rol**: Antorcha de redstone que actúa como inversor lógico NOT y fuente de energía.
- **Descripción Técnica**: Alterna entre estado encendido y apagado. Cuando el bloque al que está anclada recibe energía, se apaga en el siguiente tick. Incluye un sistema contra burnout (quemado) si conmuta más de 8 veces en pocos ticks.
- **Guía de Modificación**: Modifica updateTick para ajustar el retardo de conmutación o la sensibilidad al quemado.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `int tickRate()`
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `void onBlockRemoval(World world, int i, int j, int k)`
  - `boolean isPoweringTo(IBlockAccess iblockaccess, int i, int j, int k, int l)`
  - `void updateTick(World world, int i, int j, int k, Random random)`

##### <a id="class-blockredstonewire"></a>[`BlockRedstoneWire`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockRedstoneWire.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockRedstoneWire` (`net/minecraft/src/BlockRedstoneWire.java`) | **Líneas de código**: 402
- **Declaración**: `class BlockRedstoneWire extends Block`
- **Rol**: Cable conductor de energía de redstone con atenuación de 15 a 0.
- **Descripción Técnica**: El núcleo del sistema lógico de redstone. Transmite energía entre bloques adyacentes calculando la intensidad de señal (0 a 15) que se degrada 1 nivel por cada bloque de distancia. Notifica actualizaciones a los 6 bloques vecinos.
- **Guía de Modificación**: Para modificar la distancia máxima de transmisión o la lógica de conexiones cruzadas, se reescribe updateSurroundingWires.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`

##### <a id="class-blockreed"></a>[`BlockReed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockReed.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockReed` (`net/minecraft/src/BlockReed.java`) | **Líneas de código**: 110
- **Declaración**: `class BlockReed extends Block`
- **Rol**: Manejador del bloque 'BlockReed' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockReed'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void checkBlockCoordValid(World world, int i, int j, int k)`
  - `boolean canBlockStay(World world, int i, int j, int k)`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`

##### <a id="class-blocksand"></a>[`BlockSand`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSand.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSand` (`net/minecraft/src/BlockSand.java`) | **Líneas de código**: 78
- **Declaración**: `class BlockSand extends Block`
- **Rol**: Manejador del bloque 'BlockSand' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSand'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `int tickRate()`
  - `boolean canFallBelow(World world, int i, int j, int k)`
- **Campos Clave Destacados**:
  - `boolean fallInstantly`

##### <a id="class-blocksapling"></a>[`BlockSapling`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSapling.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSapling` (`net/minecraft/src/BlockSapling.java`) | **Líneas de código**: 42
- **Declaración**: `class BlockSapling extends BlockFlower`
- **Rol**: Manejador del bloque 'BlockSapling' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSapling'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void updateTick(World world, int i, int j, int k, Random random)`

##### <a id="class-blocksign"></a>[`BlockSign`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSign.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSign` (`net/minecraft/src/BlockSign.java`) | **Líneas de código**: 136
- **Declaración**: `class BlockSign extends BlockContainer`
- **Rol**: Manejador del bloque 'BlockSign' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSign'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `AxisAlignedBB getSelectedBoundingBoxFromPool(World world, int i, int j, int k)`
  - `void setBlockBoundsBasedOnState(IBlockAccess iblockaccess, int i, int j, int k)`
  - `int getRenderType()`
  - `boolean renderAsNormalBlock()`
  - `boolean isOpaqueCube()`

##### <a id="class-blockslowsand"></a>[`BlockSlowSand`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSlowSand.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSlowSand` (`net/minecraft/src/BlockSlowSand.java`) | **Líneas de código**: 26
- **Declaración**: `class BlockSlowSand extends Block`
- **Rol**: Soul Sand / Arena de almas del Nether con reducción de velocidad.
- **Descripción Técnica**: Ralentiza a cualquier entidad que camine sobre ella reduciendo su caja de colisión a 0.875 de altura, provocando que el jugador se hunda ligeramente.
- **Guía de Modificación**: Ajusta getCollisionBoundingBoxFromPool para aumentar o reducir la fricción y hundimiento.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `void onEntityCollidedWithBlock(World world, int i, int j, int k, Entity entity)`

##### <a id="class-blocksnow"></a>[`BlockSnow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSnow.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSnow` (`net/minecraft/src/BlockSnow.java`) | **Líneas de código**: 110
- **Declaración**: `class BlockSnow extends Block`
- **Rol**: Manejador del bloque 'BlockSnow' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSnow'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void harvestBlock(World world, int i, int j, int k, int l)`

##### <a id="class-blocksnowblock"></a>[`BlockSnowBlock`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSnowBlock.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSnowBlock` (`net/minecraft/src/BlockSnowBlock.java`) | **Líneas de código**: 35
- **Declaración**: `class BlockSnowBlock extends Block`
- **Rol**: Manejador del bloque 'BlockSnowBlock' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSnowBlock'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`
  - `int quantityDropped(Random random)`
  - `void updateTick(World world, int i, int j, int k, Random random)`

##### <a id="class-blocksoil"></a>[`BlockSoil`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSoil.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSoil` (`net/minecraft/src/BlockSoil.java`) | **Líneas de código**: 133
- **Declaración**: `class BlockSoil extends Block`
- **Rol**: Manejador del bloque 'BlockSoil' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSoil'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getBlockTextureFromSideAndMetadata(int i, int j)`
  - `void updateTick(World world, int i, int j, int k, Random random)`
  - `void onEntityWalking(World world, int i, int j, int k, Entity entity)`

##### <a id="class-blocksponge"></a>[`BlockSponge`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockSponge.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockSponge` (`net/minecraft/src/BlockSponge.java`) | **Líneas de código**: 51
- **Declaración**: `class BlockSponge extends Block`
- **Rol**: Manejador del bloque 'BlockSponge' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockSponge'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `void onBlockRemoval(World world, int i, int j, int k)`

##### <a id="class-blockstairs"></a>[`BlockStairs`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockStairs.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockStairs` (`net/minecraft/src/BlockStairs.java`) | **Líneas de código**: 233
- **Declaración**: `class BlockStairs extends Block`
- **Rol**: Manejador del bloque 'BlockStairs' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockStairs'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void setBlockBoundsBasedOnState(IBlockAccess iblockaccess, int i, int j, int k)`
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `boolean shouldSideBeRendered(IBlockAccess iblockaccess, int i, int j, int k, int l)`

##### <a id="class-blockstationary"></a>[`BlockStationary`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockStationary.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockStationary` (`net/minecraft/src/BlockStationary.java`) | **Líneas de código**: 73
- **Declaración**: `class BlockStationary extends BlockFluids`
- **Rol**: Manejador del bloque 'BlockStationary' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockStationary'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void updateTick(World world, int i, int j, int k, Random random)`

##### <a id="class-blockstep"></a>[`BlockStep`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockStep.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockStep` (`net/minecraft/src/BlockStep.java`) | **Líneas de código**: 91
- **Declaración**: `class BlockStep extends Block`
- **Rol**: Manejador del bloque 'BlockStep' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockStep'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`
  - `boolean isOpaqueCube()`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `void onBlockAdded(World world, int i, int j, int k)`
  - `int idDropped(int i, Random random)`
  - `boolean renderAsNormalBlock()`

##### <a id="class-blockstone"></a>[`BlockStone`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockStone.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockStone` (`net/minecraft/src/BlockStone.java`) | **Líneas de código**: 20
- **Declaración**: `class BlockStone extends Block`
- **Rol**: Manejador del bloque 'BlockStone' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockStone'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `int idDropped(int i, Random random)`

##### <a id="class-blocktnt"></a>[`BlockTNT`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockTNT.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockTNT` (`net/minecraft/src/BlockTNT.java`) | **Líneas de código**: 65
- **Declaración**: `class BlockTNT extends Block`
- **Rol**: Bloque de dinamita activable por golpe o redstone.
- **Descripción Técnica**: Al romperse con la mano en Alpha o al recibir corriente de redstone, se destruye y crea una entidad EntityTNTPrimed con mecha activa de 80 ticks.
- **Guía de Modificación**: Modifica onBlockDestroyedByPlayer para evitar que explote al golpearlo o cambiar la potencia.
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`
  - `void onNeighborBlockChange(World world, int i, int j, int k, int l)`
  - `int quantityDropped(Random random)`
  - `void onBlockDestroyedByExplosion(World world, int i, int j, int k)`
  - `void onBlockDestroyedByPlayer(World world, int i, int j, int k, int l)`

##### <a id="class-blocktorch"></a>[`BlockTorch`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockTorch.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockTorch` (`net/minecraft/src/BlockTorch.java`) | **Líneas de código**: 225
- **Declaración**: `class BlockTorch extends Block`
- **Rol**: Manejador del bloque 'BlockTorch' en el mundo voxel.
- **Descripción Técnica**: Define el comportamiento físico, renderizado, drops y colisiones para el bloque 'BlockTorch'. Gestiona su interacción con bloques vecinos y su ciclo de vida en el mundo.
- **Guía de Modificación**: Sobrescribe métodos de interacción (blockActivated) o drops (idDropped/quantityDropped) para modificarlo.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getCollisionBoundingBoxFromPool(World world, int i, int j, int k)`
  - `boolean isOpaqueCube()`
  - `boolean renderAsNormalBlock()`
  - `int getRenderType()`
  - `boolean canPlaceBlockAt(World world, int i, int j, int k)`
  - `void onBlockPlaced(World world, int i, int j, int k, int l)`

##### <a id="class-blockworkbench"></a>[`BlockWorkbench`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/BlockWorkbench.java)
- **Ruta y Paquete**: `net.minecraft.src.BlockWorkbench` (`net/minecraft/src/BlockWorkbench.java`) | **Líneas de código**: 40
- **Declaración**: `class BlockWorkbench extends Block`
- **Rol**: Mesa de trabajo para crafteo en cuadrícula extendida de 3x3.
- **Descripción Técnica**: Al hacer clic derecho (blockActivated), abre la pantalla GuiCrafting con el contenedor ContainerWorkbench de 9 casillas.
- **Guía de Modificación**: Punto de referencia para crear mesas de artesanía especializadas (alquimia, herrería, magia).
- **Métodos Clave Destacados**:
  - `int getBlockTextureFromSide(int i)`
  - `boolean blockActivated(World world, int i, int j, int k, EntityPlayer entityplayer)`

##### <a id="class-material"></a>[`Material`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Material.java)
- **Ruta y Paquete**: `net.minecraft.src.Material` (`net/minecraft/src/Material.java`) | **Líneas de código**: 71
- **Declaración**: `class Material`
- **Rol**: Definición de propiedades intrínsecas de la materia en el mundo de bloques.
- **Descripción Técnica**: Indica si un material es inflamable, si requiere herramientas para dropear ítems, si es líquido o bloquea el movimiento.
- **Guía de Modificación**: Añade constantes de material personalizadas para nuevos grupos de bloques con físicas afines.
- **Métodos Clave Destacados**:
  - `boolean getIsLiquid()`
  - `boolean func_878_a()`
  - `boolean getCanBlockGrass()`
  - `boolean func_880_c()`
  - `boolean getBurning()`
- **Campos Clave Destacados**:
  - `Material air`
  - `Material ground`
  - `Material wood`
  - `Material rock`
  - `Material iron`
  - `Material water`

##### <a id="class-materialliquid"></a>[`MaterialLiquid`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MaterialLiquid.java)
- **Ruta y Paquete**: `net.minecraft.src.MaterialLiquid` (`net/minecraft/src/MaterialLiquid.java`) | **Líneas de código**: 28
- **Declaración**: `class MaterialLiquid extends Material`
- **Rol**: Especialización de Material para fluidos (agua y lava).
- **Descripción Técnica**: Declara que el material es líquido y no bloquea el movimiento sólido pero sí ejerce resistencia al avance.
- **Guía de Modificación**: Utilizado por bloques fluidos.
- **Métodos Clave Destacados**:
  - `boolean getIsLiquid()`
  - `boolean func_880_c()`
  - `boolean func_878_a()`

##### <a id="class-materiallogic"></a>[`MaterialLogic`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MaterialLogic.java)
- **Ruta y Paquete**: `net.minecraft.src.MaterialLogic` (`net/minecraft/src/MaterialLogic.java`) | **Líneas de código**: 28
- **Declaración**: `class MaterialLogic extends Material`
- **Rol**: Especialización de Material para componentes lógicos y circuitos.
- **Descripción Técnica**: Define materiales que se destruyen inmediatamente al fluir agua sobre ellos (antorchas, redstone, repetidores).
- **Guía de Modificación**: Usar para cables o circuitos que deban ser vulnerables a inundaciones.
- **Métodos Clave Destacados**:
  - `boolean func_878_a()`
  - `boolean getCanBlockGrass()`
  - `boolean func_880_c()`

##### <a id="class-materialtransparent"></a>[`MaterialTransparent`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MaterialTransparent.java)
- **Ruta y Paquete**: `net.minecraft.src.MaterialTransparent` (`net/minecraft/src/MaterialTransparent.java`) | **Líneas de código**: 28
- **Declaración**: `class MaterialTransparent extends Material`
- **Rol**: Especialización de Material para bloques transparentes que no ocluyen luz.
- **Descripción Técnica**: Utilizado por cristal, hielo y hojas de árboles.
- **Guía de Modificación**: Usar para bloques translúcidos.
- **Métodos Clave Destacados**:
  - `boolean func_878_a()`
  - `boolean getCanBlockGrass()`
  - `boolean func_880_c()`

##### <a id="class-minecarttracklogic"></a>[`MinecartTrackLogic`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MinecartTrackLogic.java)
- **Ruta y Paquete**: `net.minecraft.src.MinecartTrackLogic` (`net/minecraft/src/MinecartTrackLogic.java`) | **Líneas de código**: 405
- **Declaración**: `class MinecartTrackLogic`
- **Rol**: Lógica geométrica de conexión y curvatura de raíles de vagoneta.
- **Descripción Técnica**: Calcula curvas, pendientes y bifurcaciones entre raíles contiguos basándose en las coordenadas y alturas de bloques vecinos.
- **Guía de Modificación**: Crucial para implementar raíles propulsores, cambios de aguja o cruces complejos.
- **Métodos Clave Destacados**:
  - `void func_792_a(boolean flag)`

##### <a id="class-redstoneupdateinfo"></a>[`RedstoneUpdateInfo`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RedstoneUpdateInfo.java)
- **Ruta y Paquete**: `net.minecraft.src.RedstoneUpdateInfo` (`net/minecraft/src/RedstoneUpdateInfo.java`) | **Líneas de código**: 22
- **Declaración**: `class RedstoneUpdateInfo`
- **Rol**: Registro de propagación para limitar actualizaciones cíclicas de redstone.
- **Descripción Técnica**: Almacena coordenadas y timestamp para evitar bucles infinitos de encendido/apagado que congelen el juego.
- **Guía de Modificación**: Protección crítica contra cierres inesperados por circuitos de reloj ultrarrápidos.

##### <a id="class-stepsound"></a>[`StepSound`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/StepSound.java)
- **Ruta y Paquete**: `net.minecraft.src.StepSound` (`net/minecraft/src/StepSound.java`) | **Líneas de código**: 40
- **Declaración**: `class StepSound`
- **Rol**: Efectos sonoros emitidos al caminar, colocar o destruir bloques.
- **Descripción Técnica**: Asocia identificadores de audio a las acciones físicas realizadas sobre el bloque.
- **Guía de Modificación**: Crea sonidos personalizados para bloques con acústica singular.
- **Métodos Clave Destacados**:
  - `float func_1147_b()`
  - `float func_1144_c()`
  - `String func_1146_a()`
  - `String func_1145_d()`
- **Campos Clave Destacados**:
  - `String field_1678_a`
  - `float field_1677_b`
  - `float field_1679_c`

##### <a id="class-stepsoundsand"></a>[`StepSoundSand`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/StepSoundSand.java)
- **Ruta y Paquete**: `net.minecraft.src.StepSoundSand` (`net/minecraft/src/StepSoundSand.java`) | **Líneas de código**: 19
- **Declaración**: `class StepSoundSand extends StepSound`
- **Rol**: Variante de StepSound para arena, grava y nieve.
- **Descripción Técnica**: Genera sonidos amortiguados y suaves al pisar o romper bloques granulares.
- **Guía de Modificación**: Ajusta el tono y volumen para superficies arenosas.
- **Métodos Clave Destacados**:
  - `String func_1146_a()`

##### <a id="class-stepsoundstone"></a>[`StepSoundStone`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/StepSoundStone.java)
- **Ruta y Paquete**: `net.minecraft.src.StepSoundStone` (`net/minecraft/src/StepSoundStone.java`) | **Líneas de código**: 19
- **Declaración**: `class StepSoundStone extends StepSound`
- **Rol**: Variante de StepSound para piedra, minerales y adoquín.
- **Descripción Técnica**: Genera sonidos secos y rotundos característicos de la roca sólida.
- **Guía de Modificación**: Ajusta el sonido de impacto en materiales pétreos.
- **Métodos Clave Destacados**:
  - `String func_1146_a()`


### 03. Objetos, Crafteo e Inventarios (Items, Crafting & Inventory)
> **Resumen del subsistema**: Definición de herramientas, armas, armaduras, consumibles y materiales. Maneja la durabilidad, pilas (stacks), recetas en rejillas de 2x2 y 3x3 (CraftingManager), fundición en hornos y gestión de inventarios de jugador.

**Total de clases**: 47

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`CraftingInventoryCB`](#class-craftinginventorycb) | `class` | `-` | 31 | Contenedor abstracto base para interfaces con cuadrículas de craf... |
| [`CraftingInventoryPlayerCB`](#class-craftinginventoryplayercb) | `class` | `ext CraftingInventoryCB` | 57 | Contenedor para la cuadrícula de crafteo básica de 2x2 en el inve... |
| [`CraftingInventoryWorkbenchCB`](#class-craftinginventoryworkbenchcb) | `class` | `ext CraftingInventoryCB` | 55 | Contenedor para la mesa de trabajo de 3x3 (Workbench). |
| [`CraftingManager`](#class-craftingmanager) | `class` | `-` | 229 | Gestor y registro central de recetas de crafteo del juego. |
| [`CraftingRecipe`](#class-craftingrecipe) | `class` | `-` | 85 | Definición y validación de una receta de crafteo específica. |
| [`EnumArt`](#class-enumart) | `enum` | `-` | 119 | Enumeración de cuadros artísticos disponibles (pinturas decorativ... |
| [`GuiCrafting`](#class-guicrafting) | `class` | `ext GuiContainer` | 65 | Pantalla o elemento de interfaz gráfica: 'GuiCrafting'. |
| [`InventoryCraftResult`](#class-inventorycraftresult) | `class` | `impl IInventory
` | 59 | Inventario de 1 sola casilla que alberga el producto resultante d... |
| [`InventoryCrafting`](#class-inventorycrafting) | `class` | `impl IInventory
` | 82 | Inventario de entrada para la cuadrícula de fabricación (2x2 o 3x... |
| [`InventoryLargeChest`](#class-inventorylargechest) | `class` | `impl IInventory
` | 75 | Contenedor virtual que fusiona dos TileEntityChest en un cofre do... |
| [`InventoryPlayer`](#class-inventoryplayer) | `class` | `impl IInventory
` | 502 | El inventario completo del jugador (44 casillas en total). |
| [`Item`](#class-item) | `class` | `-` | 213 | Clase base de todos los objetos, herramientas y consumibles del j... |
| [`ItemArmor`](#class-itemarmor) | `class` | `ext Item` | 32 | Piezas de armadura corporal (casco, peto, pantalones, botas). |
| [`ItemAxe`](#class-itemaxe) | `class` | `ext ItemTool` | 23 | Ítem especializado 'ItemAxe' en el sistema de inventario. |
| [`ItemBlock`](#class-itemblock) | `class` | `ext Item` | 68 | Ítem especializado 'ItemBlock' en el sistema de inventario. |
| [`ItemBoat`](#class-itemboat) | `class` | `ext Item` | 52 | Ítem especializado 'ItemBoat' en el sistema de inventario. |
| [`ItemBow`](#class-itembow) | `class` | `ext Item` | 29 | Arco para disparo balístico de flechas. |
| [`ItemBucket`](#class-itembucket) | `class` | `ext Item` | 119 | Ítem especializado 'ItemBucket' en el sistema de inventario. |
| [`ItemDoor`](#class-itemdoor) | `class` | `ext Item` | 83 | Ítem especializado 'ItemDoor' en el sistema de inventario. |
| [`ItemFishingRod`](#class-itemfishingrod) | `class` | `ext Item` | 45 | Ítem especializado 'ItemFishingRod' en el sistema de inventario. |
| [`ItemFlintAndSteel`](#class-itemflintandsteel) | `class` | `ext Item` | 53 | Ítem especializado 'ItemFlintAndSteel' en el sistema de inventari... |
| [`ItemFood`](#class-itemfood) | `class` | `ext Item` | 25 | Alimento consumible que restaura salud directamente. |
| [`ItemHoe`](#class-itemhoe) | `class` | `ext Item` | 58 | Ítem especializado 'ItemHoe' en el sistema de inventario. |
| [`ItemMinecart`](#class-itemminecart) | `class` | `ext Item` | 35 | Ítem especializado 'ItemMinecart' en el sistema de inventario. |
| [`ItemPainting`](#class-itempainting) | `class` | `ext Item` | 47 | Ítem especializado 'ItemPainting' en el sistema de inventario. |
| [`ItemPickaxe`](#class-itempickaxe) | `class` | `ext ItemTool` | 55 | Herramienta de pico para minar piedra y minerales. |
| [`ItemRecord`](#class-itemrecord) | `class` | `ext Item` | 32 | Ítem especializado 'ItemRecord' en el sistema de inventario. |
| [`ItemRedstone`](#class-itemredstone) | `class` | `ext Item` | 52 | Ítem especializado 'ItemRedstone' en el sistema de inventario. |
| [`ItemReed`](#class-itemreed) | `class` | `ext Item` | 66 | Ítem especializado 'ItemReed' en el sistema de inventario. |
| [`ItemRenderer`](#class-itemrenderer) | `class` | `-` | 364 | Ítem especializado 'ItemRenderer' en el sistema de inventario. |
| [`ItemSaddle`](#class-itemsaddle) | `class` | `ext Item` | 34 | Ítem especializado 'ItemSaddle' en el sistema de inventario. |
| [`ItemSeeds`](#class-itemseeds) | `class` | `ext Item` | 35 | Ítem especializado 'ItemSeeds' en el sistema de inventario. |
| [`ItemSign`](#class-itemsign) | `class` | `ext Item` | 66 | Ítem especializado 'ItemSign' en el sistema de inventario. |
| [`ItemSnowball`](#class-itemsnowball) | `class` | `ext Item` | 27 | Ítem especializado 'ItemSnowball' en el sistema de inventario. |
| [`ItemSoup`](#class-itemsoup) | `class` | `ext ItemFood` | 20 | Ítem especializado 'ItemSoup' en el sistema de inventario. |
| [`ItemSpade`](#class-itemspade) | `class` | `ext ItemTool` | 32 | Ítem especializado 'ItemSpade' en el sistema de inventario. |
| [`ItemStack`](#class-itemstack) | `class` | `-` | 164 | Instancia concreta de un ítem con cantidad y metadato/daño. |
| [`ItemSword`](#class-itemsword) | `class` | `ext Item` | 48 | Arma de combate cuerpo a cuerpo con bonificación de daño. |
| [`ItemTool`](#class-itemtool) | `class` | `ext Item` | 63 | Clase base para herramientas de trabajo (picos, hachas, palas, az... |
| [`RecipeSorter`](#class-recipesorter) | `class` | `impl Comparator
` | 32 | Comparador que ordena las recetas en CraftingManager de mayor a m... |
| [`RecipesArmor`](#class-recipesarmor) | `class` | `-` | 56 | Definición de recetas de piezas de armadura. |
| [`RecipesCrafting`](#class-recipescrafting) | `class` | `-` | 26 | Definición de recetas básicas de construcción y utilidades. |
| [`RecipesFood`](#class-recipesfood) | `class` | `-` | 23 | Definición de recetas de alimentos compuestos. |
| [`RecipesIngots`](#class-recipesingots) | `class` | `-` | 40 | Definición de recetas de compactación y descompactación de lingot... |
| [`RecipesTools`](#class-recipestools) | `class` | `-` | 56 | Definición de recetas de herramientas de trabajo. |
| [`RecipesWeapons`](#class-recipesweapons) | `class` | `-` | 50 | Definición de recetas de espadas y armas. |
| [`SlotCrafting`](#class-slotcrafting) | `class` | `ext SlotInventory` | 34 | Casilla especializada para el producto resultante de la fabricaci... |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-craftinginventorycb"></a>[`CraftingInventoryCB`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingInventoryCB.java)
- **Ruta y Paquete**: `net.minecraft.src.CraftingInventoryCB` (`net/minecraft/src/CraftingInventoryCB.java`) | **Líneas de código**: 31
- **Declaración**: `class CraftingInventoryCB`
- **Rol**: Contenedor abstracto base para interfaces con cuadrículas de crafteo.
- **Descripción Técnica**: Gestiona la transferencia de ítems entre el inventario del jugador y la rejilla de fabricación.
- **Guía de Modificación**: Base para crear mesas de crafteo con tamaños alternativos (4x4, 5x5).
- **Métodos Clave Destacados**:
  - `void onCraftGuiClosed(EntityPlayer entityplayer)`
  - `void onCraftMatrixChanged(IInventory iinventory)`
- **Campos Clave Destacados**:
  - `List unusedList`

##### <a id="class-craftinginventoryplayercb"></a>[`CraftingInventoryPlayerCB`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingInventoryPlayerCB.java)
- **Ruta y Paquete**: `net.minecraft.src.CraftingInventoryPlayerCB` (`net/minecraft/src/CraftingInventoryPlayerCB.java`) | **Líneas de código**: 57
- **Declaración**: `class CraftingInventoryPlayerCB extends CraftingInventoryCB`
- **Rol**: Contenedor para la cuadrícula de crafteo básica de 2x2 en el inventario del jugador.
- **Descripción Técnica**: Gestiona las 4 casillas de crafteo personal y el slot de salida cuando se abre el inventario (tecla E).
- **Guía de Modificación**: Modificar para alterar el inventario personal del jugador.
- **Métodos Clave Destacados**:
  - `void onCraftMatrixChanged(IInventory iinventory)`
  - `void onCraftGuiClosed(EntityPlayer entityplayer)`
- **Campos Clave Destacados**:
  - `InventoryCrafting craftMatrix`
  - `IInventory craftResult`

##### <a id="class-craftinginventoryworkbenchcb"></a>[`CraftingInventoryWorkbenchCB`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingInventoryWorkbenchCB.java)
- **Ruta y Paquete**: `net.minecraft.src.CraftingInventoryWorkbenchCB` (`net/minecraft/src/CraftingInventoryWorkbenchCB.java`) | **Líneas de código**: 55
- **Declaración**: `class CraftingInventoryWorkbenchCB extends CraftingInventoryCB`
- **Rol**: Contenedor para la mesa de trabajo de 3x3 (Workbench).
- **Descripción Técnica**: Gestiona las 9 casillas de fabricación y la casilla de resultado de crafteo.
- **Guía de Modificación**: Modificar para añadir funciones especiales a la mesa de trabajo.
- **Métodos Clave Destacados**:
  - `void onCraftMatrixChanged(IInventory iinventory)`
  - `void onCraftGuiClosed(EntityPlayer entityplayer)`
- **Campos Clave Destacados**:
  - `InventoryCrafting craftMatrix`
  - `IInventory craftResult`

##### <a id="class-craftingmanager"></a>[`CraftingManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingManager.java)
- **Ruta y Paquete**: `net.minecraft.src.CraftingManager` (`net/minecraft/src/CraftingManager.java`) | **Líneas de código**: 229
- **Declaración**: `class CraftingManager`
- **Rol**: Gestor y registro central de recetas de crafteo del juego.
- **Descripción Técnica**: Mantiene la lista 'recipes' con todas las combinaciones en cuadrículas 2x2 y 3x3. Implementa addRecipe y addShapelessRecipe.
- **Guía de Modificación**: Llama a CraftingManager.getInstance().addRecipe(...) durante la inicialización para registrar recetas de mods.
- **Métodos Clave Destacados**:
  - `CraftingManager getInstance()`
  - `ItemStack craft(int ai[])`

##### <a id="class-craftingrecipe"></a>[`CraftingRecipe`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CraftingRecipe.java)
- **Ruta y Paquete**: `net.minecraft.src.CraftingRecipe` (`net/minecraft/src/CraftingRecipe.java`) | **Líneas de código**: 85
- **Declaración**: `class CraftingRecipe`
- **Rol**: Definición y validación de una receta de crafteo específica.
- **Descripción Técnica**: Compara la disposición de ítems en la cuadrícula con el patrón configurado para generar el resultado.
- **Guía de Modificación**: Modificar si se implementan recetas complejas con condiciones dinámicas.
- **Métodos Clave Destacados**:
  - `boolean matchRecipe(int ai[])`
  - `ItemStack createResult(int ai[])`
  - `int getRecipeSize()`
- **Campos Clave Destacados**:
  - `int resultId`

##### <a id="class-enumart"></a>[`EnumArt`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumArt.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumArt` (`net/minecraft/src/EnumArt.java`) | **Líneas de código**: 119
- **Declaración**: `enum EnumArt`
- **Rol**: Enumeración de cuadros artísticos disponibles (pinturas decorativas).
- **Descripción Técnica**: Define dimensiones en bloques (1x1, 2x1, 4x4, etc.) y coordenadas UV en 'art/kz.png' para cada cuadro.
- **Guía de Modificación**: Añade nuevos valores a esta enumeración para incorporar cuadros y pósteres adicionales.
- **Métodos Clave Destacados**:
  - `EnumArt[] func_1105_values()`
  - `EnumArt valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumArt Kebab`
  - `EnumArt Aztec`
  - `EnumArt Alban`
  - `EnumArt Aztec2`
  - `EnumArt Bomb`
  - `EnumArt Plant`

##### <a id="class-guicrafting"></a>[`GuiCrafting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiCrafting.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiCrafting` (`net/minecraft/src/GuiCrafting.java`) | **Líneas de código**: 65
- **Declaración**: `class GuiCrafting extends GuiContainer`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiCrafting'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiCrafting'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void onGuiClosed()`
  - `void drawGuiContainerForegroundLayer()`
  - `void drawGuiContainerBackgroundLayer(float f)`
- **Campos Clave Destacados**:
  - `CraftingInventoryWorkbenchCB field_979_j`

##### <a id="class-inventorycraftresult"></a>[`InventoryCraftResult`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/InventoryCraftResult.java)
- **Ruta y Paquete**: `net.minecraft.src.InventoryCraftResult` (`net/minecraft/src/InventoryCraftResult.java`) | **Líneas de código**: 59
- **Declaración**: `class InventoryCraftResult implements IInventory
`
- **Rol**: Inventario de 1 sola casilla que alberga el producto resultante del crafteo.
- **Descripción Técnica**: Se actualiza automáticamente cada vez que los ítems en la rejilla de fabricación forman una receta válida.
- **Guía de Modificación**: Maneja la entrega del ítem final al jugador cuando hace clic.
- **Métodos Clave Destacados**:
  - `int getSizeInventory()`
  - `ItemStack getStackInSlot(int i)`
  - `String getInvName()`
  - `ItemStack decrStackSize(int i, int j)`
  - `void setInventorySlotContents(int i, ItemStack itemstack)`
  - `int getInventoryStackLimit()`

##### <a id="class-inventorycrafting"></a>[`InventoryCrafting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/InventoryCrafting.java)
- **Ruta y Paquete**: `net.minecraft.src.InventoryCrafting` (`net/minecraft/src/InventoryCrafting.java`) | **Líneas de código**: 82
- **Declaración**: `class InventoryCrafting implements IInventory
`
- **Rol**: Inventario de entrada para la cuadrícula de fabricación (2x2 o 3x3).
- **Descripción Técnica**: Almacena los ítems colocados por el jugador y notifica al contenedor para recalcular la receta en tiempo real.
- **Guía de Modificación**: Maneja la disposición espacial de los ingredientes.
- **Métodos Clave Destacados**:
  - `int getSizeInventory()`
  - `ItemStack getStackInSlot(int i)`
  - `String getInvName()`
  - `ItemStack decrStackSize(int i, int j)`
  - `void setInventorySlotContents(int i, ItemStack itemstack)`
  - `int getInventoryStackLimit()`

##### <a id="class-inventorylargechest"></a>[`InventoryLargeChest`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/InventoryLargeChest.java)
- **Ruta y Paquete**: `net.minecraft.src.InventoryLargeChest` (`net/minecraft/src/InventoryLargeChest.java`) | **Líneas de código**: 75
- **Declaración**: `class InventoryLargeChest implements IInventory
`
- **Rol**: Contenedor virtual que fusiona dos TileEntityChest en un cofre doble de 54 casillas.
- **Descripción Técnica**: Redirige las operaciones a uno u otro cofre según el índice del slot solicitado.
- **Guía de Modificación**: Permite tratar dos inventarios independientes como uno solo.
- **Métodos Clave Destacados**:
  - `int getSizeInventory()`
  - `String getInvName()`
  - `ItemStack getStackInSlot(int i)`
  - `ItemStack decrStackSize(int i, int j)`
  - `void setInventorySlotContents(int i, ItemStack itemstack)`
  - `int getInventoryStackLimit()`

##### <a id="class-inventoryplayer"></a>[`InventoryPlayer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/InventoryPlayer.java)
- **Ruta y Paquete**: `net.minecraft.src.InventoryPlayer` (`net/minecraft/src/InventoryPlayer.java`) | **Líneas de código**: 502
- **Declaración**: `class InventoryPlayer implements IInventory
`
- **Rol**: El inventario completo del jugador (44 casillas en total).
- **Descripción Técnica**: Gestiona las 36 casillas de almacenamiento, 4 de armadura, 1 de cursor en mano, el slot seleccionado en la hotbar y el descarte de ítems.
- **Guía de Modificación**: Clase central para manipular el inventario del usuario desde el código.
- **Métodos Clave Destacados**:
  - `ItemStack getCurrentItem()`
  - `void setCurrentItem(int i, boolean flag)`
  - `void changeCurrentItem(int i)`
  - `void decrementAnimations()`
  - `boolean consumeInventoryItem(int i)`
  - `boolean addItemStackToInventory(ItemStack itemstack)`
- **Campos Clave Destacados**:
  - `int currentItem`
  - `ItemStack draggingItemStack`
  - `boolean field_845_f`

##### <a id="class-item"></a>[`Item`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Item.java)
- **Ruta y Paquete**: `net.minecraft.src.Item` (`net/minecraft/src/Item.java`) | **Líneas de código**: 213
- **Declaración**: `class Item`
- **Rol**: Clase base de todos los objetos, herramientas y consumibles del juego.
- **Descripción Técnica**: Define el registro global 'itemsList[32000]'. Los IDs de ítems arrancan en el índice 256 para no colisionar con los bloques. Define durabilidad (maxDamage), tamaño de pila (maxStackSize), textura en /gui/items.png (iconIndex) e interacciones onItemRightClick, onItemUse, hitEntity y onBlockDestroyed.
- **Guía de Modificación**: Para crear un nuevo ítem, se instancia una subclase asignándole un ID >= 256 configurando sus propiedades.
- **Métodos Clave Destacados**:
  - `Item setIconIndex(int i)`
  - `int getIconIndex(ItemStack itemstack)`
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`
  - `float getStrVsBlock(ItemStack itemstack, Block block)`
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`
  - `int getItemStackLimit()`
- **Campos Clave Destacados**:
  - `Random itemRand`
  - `Item shovelSteel`
  - `Item pickaxeSteel`
  - `Item axeSteel`
  - `Item flintAndSteel`
  - `Item appleRed`

##### <a id="class-itemarmor"></a>[`ItemArmor`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemArmor.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemArmor` (`net/minecraft/src/ItemArmor.java`) | **Líneas de código**: 32
- **Declaración**: `class ItemArmor extends Item`
- **Rol**: Piezas de armadura corporal (casco, peto, pantalones, botas).
- **Descripción Técnica**: Proporciona puntos de reducción de daño (damageReduceAmount), define el slot de armadura que ocupa (0 a 3) y la textura asignada al modelo del jugador.
- **Guía de Modificación**: Modifica damageReduceAmount o crea conjuntos de armaduras de nuevos materiales.
- **Campos Clave Destacados**:
  - `int armorLevel`
  - `int armorType`
  - `int damageReduceAmmount`
  - `int renderIndex`

##### <a id="class-itemaxe"></a>[`ItemAxe`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemAxe.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemAxe` (`net/minecraft/src/ItemAxe.java`) | **Líneas de código**: 23
- **Declaración**: `class ItemAxe extends ItemTool`
- **Rol**: Ítem especializado 'ItemAxe' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemAxe'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.

##### <a id="class-itemblock"></a>[`ItemBlock`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemBlock.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemBlock` (`net/minecraft/src/ItemBlock.java`) | **Líneas de código**: 68
- **Declaración**: `class ItemBlock extends Item`
- **Rol**: Ítem especializado 'ItemBlock' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemBlock'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemboat"></a>[`ItemBoat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemBoat.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemBoat` (`net/minecraft/src/ItemBoat.java`) | **Líneas de código**: 52
- **Declaración**: `class ItemBoat extends Item`
- **Rol**: Ítem especializado 'ItemBoat' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemBoat'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itembow"></a>[`ItemBow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemBow.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemBow` (`net/minecraft/src/ItemBow.java`) | **Líneas de código**: 29
- **Declaración**: `class ItemBow extends Item`
- **Rol**: Arco para disparo balístico de flechas.
- **Descripción Técnica**: En onItemRightClick comprueba si el inventario del jugador contiene flechas (Item.arrow); si es así, consume una y genera la entidad EntityArrow con impulso parabólico.
- **Guía de Modificación**: Modifica la velocidad del proyectil o implementa arcos con munición mágica o explosiva.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itembucket"></a>[`ItemBucket`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemBucket.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemBucket` (`net/minecraft/src/ItemBucket.java`) | **Líneas de código**: 119
- **Declaración**: `class ItemBucket extends Item`
- **Rol**: Ítem especializado 'ItemBucket' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemBucket'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itemdoor"></a>[`ItemDoor`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemDoor.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemDoor` (`net/minecraft/src/ItemDoor.java`) | **Líneas de código**: 83
- **Declaración**: `class ItemDoor extends Item`
- **Rol**: Ítem especializado 'ItemDoor' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemDoor'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemfishingrod"></a>[`ItemFishingRod`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemFishingRod.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemFishingRod` (`net/minecraft/src/ItemFishingRod.java`) | **Líneas de código**: 45
- **Declaración**: `class ItemFishingRod extends Item`
- **Rol**: Ítem especializado 'ItemFishingRod' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemFishingRod'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean isFull3D()`
  - `boolean shouldRotateAroundWhenRendering()`
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itemflintandsteel"></a>[`ItemFlintAndSteel`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemFlintAndSteel.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemFlintAndSteel` (`net/minecraft/src/ItemFlintAndSteel.java`) | **Líneas de código**: 53
- **Declaración**: `class ItemFlintAndSteel extends Item`
- **Rol**: Ítem especializado 'ItemFlintAndSteel' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemFlintAndSteel'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemfood"></a>[`ItemFood`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemFood.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemFood` (`net/minecraft/src/ItemFood.java`) | **Líneas de código**: 25
- **Declaración**: `class ItemFood extends Item`
- **Rol**: Alimento consumible que restaura salud directamente.
- **Descripción Técnica**: En Alpha 1.2.6 no existía barra de hambre: la comida cura directamente corazones de vida (healAmount) al consumirse con clic derecho.
- **Guía de Modificación**: Crea nuevos alimentos ajustando healAmount o implementando efectos secundarios.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itemhoe"></a>[`ItemHoe`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemHoe.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemHoe` (`net/minecraft/src/ItemHoe.java`) | **Líneas de código**: 58
- **Declaración**: `class ItemHoe extends Item`
- **Rol**: Ítem especializado 'ItemHoe' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemHoe'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`
  - `boolean isFull3D()`

##### <a id="class-itemminecart"></a>[`ItemMinecart`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemMinecart.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemMinecart` (`net/minecraft/src/ItemMinecart.java`) | **Líneas de código**: 35
- **Declaración**: `class ItemMinecart extends Item`
- **Rol**: Ítem especializado 'ItemMinecart' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemMinecart'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`
- **Campos Clave Destacados**:
  - `int field_317_a`

##### <a id="class-itempainting"></a>[`ItemPainting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemPainting.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemPainting` (`net/minecraft/src/ItemPainting.java`) | **Líneas de código**: 47
- **Declaración**: `class ItemPainting extends Item`
- **Rol**: Ítem especializado 'ItemPainting' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemPainting'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itempickaxe"></a>[`ItemPickaxe`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemPickaxe.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemPickaxe` (`net/minecraft/src/ItemPickaxe.java`) | **Líneas de código**: 55
- **Declaración**: `class ItemPickaxe extends ItemTool`
- **Rol**: Herramienta de pico para minar piedra y minerales.
- **Descripción Técnica**: Aumenta drásticamente la velocidad al picar bloques de roca y permite recolectar minerales duros que con la mano no dropean nada.
- **Guía de Modificación**: Añade nuevos bloques de piedra al array de eficacia para que este pico pueda extraerlos.
- **Métodos Clave Destacados**:
  - `boolean canHarvestBlock(Block block)`

##### <a id="class-itemrecord"></a>[`ItemRecord`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemRecord.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemRecord` (`net/minecraft/src/ItemRecord.java`) | **Líneas de código**: 32
- **Declaración**: `class ItemRecord extends Item`
- **Rol**: Ítem especializado 'ItemRecord' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemRecord'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemredstone"></a>[`ItemRedstone`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemRedstone.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemRedstone` (`net/minecraft/src/ItemRedstone.java`) | **Líneas de código**: 52
- **Declaración**: `class ItemRedstone extends Item`
- **Rol**: Ítem especializado 'ItemRedstone' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemRedstone'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemreed"></a>[`ItemReed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemReed.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemReed` (`net/minecraft/src/ItemReed.java`) | **Líneas de código**: 66
- **Declaración**: `class ItemReed extends Item`
- **Rol**: Ítem especializado 'ItemReed' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemReed'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemrenderer"></a>[`ItemRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemRenderer` (`net/minecraft/src/ItemRenderer.java`) | **Líneas de código**: 364
- **Declaración**: `class ItemRenderer`
- **Rol**: Ítem especializado 'ItemRenderer' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemRenderer'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `void renderItem(ItemStack itemstack)`
  - `void renderItemInFirstPerson(float f)`
  - `void renderOverlays(float f)`
  - `void func_895_a()`
  - `void func_9449_b()`
  - `void func_9450_c()`

##### <a id="class-itemsaddle"></a>[`ItemSaddle`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSaddle.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSaddle` (`net/minecraft/src/ItemSaddle.java`) | **Líneas de código**: 34
- **Declaración**: `class ItemSaddle extends Item`
- **Rol**: Ítem especializado 'ItemSaddle' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSaddle'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `void func_4019_b(ItemStack itemstack, EntityLiving entityliving)`
  - `void hitEntity(ItemStack itemstack, EntityLiving entityliving)`

##### <a id="class-itemseeds"></a>[`ItemSeeds`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSeeds.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSeeds` (`net/minecraft/src/ItemSeeds.java`) | **Líneas de código**: 35
- **Declaración**: `class ItemSeeds extends Item`
- **Rol**: Ítem especializado 'ItemSeeds' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSeeds'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemsign"></a>[`ItemSign`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSign.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSign` (`net/minecraft/src/ItemSign.java`) | **Líneas de código**: 66
- **Declaración**: `class ItemSign extends Item`
- **Rol**: Ítem especializado 'ItemSign' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSign'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean onItemUse(ItemStack itemstack, EntityPlayer entityplayer, World world, int i, int j, int k, int l)`

##### <a id="class-itemsnowball"></a>[`ItemSnowball`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSnowball.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSnowball` (`net/minecraft/src/ItemSnowball.java`) | **Líneas de código**: 27
- **Declaración**: `class ItemSnowball extends Item`
- **Rol**: Ítem especializado 'ItemSnowball' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSnowball'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itemsoup"></a>[`ItemSoup`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSoup.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSoup` (`net/minecraft/src/ItemSoup.java`) | **Líneas de código**: 20
- **Declaración**: `class ItemSoup extends ItemFood`
- **Rol**: Ítem especializado 'ItemSoup' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSoup'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `ItemStack onItemRightClick(ItemStack itemstack, World world, EntityPlayer entityplayer)`

##### <a id="class-itemspade"></a>[`ItemSpade`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSpade.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSpade` (`net/minecraft/src/ItemSpade.java`) | **Líneas de código**: 32
- **Declaración**: `class ItemSpade extends ItemTool`
- **Rol**: Ítem especializado 'ItemSpade' en el sistema de inventario.
- **Descripción Técnica**: Define las propiedades (durabilidad, stack máximo, icono) e interacción al usar o equipar 'ItemSpade'. Registrado en el índice global de ítems.
- **Guía de Modificación**: Sobrescribe onItemRightClick u onItemUse para dotarlo de efectos o habilidades personalizadas.
- **Métodos Clave Destacados**:
  - `boolean canHarvestBlock(Block block)`

##### <a id="class-itemstack"></a>[`ItemStack`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemStack.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemStack` (`net/minecraft/src/ItemStack.java`) | **Líneas de código**: 164
- **Declaración**: `class ItemStack`
- **Rol**: Instancia concreta de un ítem con cantidad y metadato/daño.
- **Descripción Técnica**: Representa la unidad básica de almacenamiento en inventarios. Contiene itemID, stackSize (cantidad actual) e itemDamage (durabilidad consumida o subtipo). Gestiona el desgaste por uso (damageItem), división de pilas y serialización NBT.
- **Guía de Modificación**: Clase crítica: cualquier manipulación de casillas de inventario pasa por crear o modificar instancias de ItemStack.
- **Métodos Clave Destacados**:
  - `ItemStack splitStack(int i)`
  - `Item getItem()`
  - `int getIconIndex()`
  - `boolean useItem(EntityPlayer entityplayer, World world, int i, int j, int k, int l)`
  - `float getStrVsBlock(Block block)`
  - `ItemStack useItemRightClick(World world, EntityPlayer entityplayer)`
- **Campos Clave Destacados**:
  - `int stackSize`
  - `int animationsToGo`
  - `int itemID`
  - `int itemDamage`

##### <a id="class-itemsword"></a>[`ItemSword`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemSword.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemSword` (`net/minecraft/src/ItemSword.java`) | **Líneas de código**: 48
- **Declaración**: `class ItemSword extends Item`
- **Rol**: Arma de combate cuerpo a cuerpo con bonificación de daño.
- **Descripción Técnica**: Calcula el daño adicional infligido a las entidades y sufre el doble de desgaste si se emplea para picar bloques sólidos.
- **Guía de Modificación**: Ajusta weaponDamage para crear armas de mayor impacto o con efectos elementales.
- **Métodos Clave Destacados**:
  - `float getStrVsBlock(ItemStack itemstack, Block block)`
  - `void hitEntity(ItemStack itemstack, EntityLiving entityliving)`
  - `void hitBlock(ItemStack itemstack, int i, int j, int k, int l)`
  - `int getDamageVsEntity(Entity entity)`
  - `boolean isFull3D()`

##### <a id="class-itemtool"></a>[`ItemTool`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ItemTool.java)
- **Ruta y Paquete**: `net.minecraft.src.ItemTool` (`net/minecraft/src/ItemTool.java`) | **Líneas de código**: 63
- **Declaración**: `class ItemTool extends Item`
- **Rol**: Clase base para herramientas de trabajo (picos, hachas, palas, azadas).
- **Descripción Técnica**: Define la velocidad de minado frente a bloques específicos según el material (madera, piedra, hierro, diamante, oro). Aplica desgaste de durabilidad con cada bloque minado o entidad atacada.
- **Guía de Modificación**: Modifica blocksEffectiveAgainst o el multiplicador de velocidad para balancear herramientas.
- **Métodos Clave Destacados**:
  - `float getStrVsBlock(ItemStack itemstack, Block block)`
  - `void hitEntity(ItemStack itemstack, EntityLiving entityliving)`
  - `void hitBlock(ItemStack itemstack, int i, int j, int k, int l)`
  - `int getDamageVsEntity(Entity entity)`
  - `boolean isFull3D()`
- **Campos Clave Destacados**:
  - `int ingredientQuality`

##### <a id="class-recipesorter"></a>[`RecipeSorter`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipeSorter.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipeSorter` (`net/minecraft/src/RecipeSorter.java`) | **Líneas de código**: 32
- **Declaración**: `class RecipeSorter implements Comparator
`
- **Rol**: Comparador que ordena las recetas en CraftingManager de mayor a menor complejidad.
- **Descripción Técnica**: Asegura que recetas que ocupan 3x3 se evalúen antes que recetas que ocupan 2x2 para evitar colisiones erróneas.
- **Guía de Modificación**: Crucial para mantener la precedencia correcta al registrar nuevas recetas.
- **Métodos Clave Destacados**:
  - `int func_1040_a(CraftingRecipe craftingrecipe, CraftingRecipe craftingrecipe1)`
  - `int compare(Object obj, Object obj1)`

##### <a id="class-recipesarmor"></a>[`RecipesArmor`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesArmor.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesArmor` (`net/minecraft/src/RecipesArmor.java`) | **Líneas de código**: 56
- **Declaración**: `class RecipesArmor`
- **Rol**: Definición de recetas de piezas de armadura.
- **Descripción Técnica**: Registra cascos, petos, pantalones y botas de cuero, hierro, oro y diamante.
- **Guía de Modificación**: Referencia para registrar conjuntos de armaduras de nuevos materiales.
- **Métodos Clave Destacados**:
  - `void func_1148_a(CraftingManager craftingmanager)`

##### <a id="class-recipescrafting"></a>[`RecipesCrafting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesCrafting.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesCrafting` (`net/minecraft/src/RecipesCrafting.java`) | **Líneas de código**: 26
- **Declaración**: `class RecipesCrafting`
- **Rol**: Definición de recetas básicas de construcción y utilidades.
- **Descripción Técnica**: Registra recetas para bloques de construcción, antorchas, escaleras, cofres y mesas de trabajo.
- **Guía de Modificación**: Ejemplo directo de sintaxis de recetas estándar.
- **Métodos Clave Destacados**:
  - `void func_1051_a(CraftingManager craftingmanager)`

##### <a id="class-recipesfood"></a>[`RecipesFood`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesFood.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesFood` (`net/minecraft/src/RecipesFood.java`) | **Líneas de código**: 23
- **Declaración**: `class RecipesFood`
- **Rol**: Definición de recetas de alimentos compuestos.
- **Descripción Técnica**: Registra la preparación de pan, pasteles, estofados de setas y galletas.
- **Guía de Modificación**: Usar como plantilla para nuevas recetas culinarias.
- **Métodos Clave Destacados**:
  - `void func_976_a(CraftingManager craftingmanager)`

##### <a id="class-recipesingots"></a>[`RecipesIngots`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesIngots.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesIngots` (`net/minecraft/src/RecipesIngots.java`) | **Líneas de código**: 40
- **Declaración**: `class RecipesIngots`
- **Rol**: Definición de recetas de compactación y descompactación de lingotes/bloques.
- **Descripción Técnica**: Registra la conversión 9 lingotes <-> 1 bloque para hierro, oro, diamante y lapislázuli.
- **Guía de Modificación**: Copiar para nuevos metales o minerales compactables.
- **Métodos Clave Destacados**:
  - `void func_810_a(CraftingManager craftingmanager)`

##### <a id="class-recipestools"></a>[`RecipesTools`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesTools.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesTools` (`net/minecraft/src/RecipesTools.java`) | **Líneas de código**: 56
- **Declaración**: `class RecipesTools`
- **Rol**: Definición de recetas de herramientas de trabajo.
- **Descripción Técnica**: Registra picos, hachas, palas y azadas de todos los materiales.
- **Guía de Modificación**: Referencia para registrar familias completas de herramientas.
- **Métodos Clave Destacados**:
  - `void func_1122_a(CraftingManager craftingmanager)`

##### <a id="class-recipesweapons"></a>[`RecipesWeapons`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RecipesWeapons.java)
- **Ruta y Paquete**: `net.minecraft.src.RecipesWeapons` (`net/minecraft/src/RecipesWeapons.java`) | **Líneas de código**: 50
- **Declaración**: `class RecipesWeapons`
- **Rol**: Definición de recetas de espadas y armas.
- **Descripción Técnica**: Registra las recetas de espadas de madera, piedra, hierro, diamante y oro.
- **Guía de Modificación**: Modificar o copiar para registrar armas personalizadas.
- **Métodos Clave Destacados**:
  - `void func_766_a(CraftingManager craftingmanager)`

##### <a id="class-slotcrafting"></a>[`SlotCrafting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SlotCrafting.java)
- **Ruta y Paquete**: `net.minecraft.src.SlotCrafting` (`net/minecraft/src/SlotCrafting.java`) | **Líneas de código**: 34
- **Declaración**: `class SlotCrafting extends SlotInventory`
- **Rol**: Casilla especializada para el producto resultante de la fabricación.
- **Descripción Técnica**: Al retirar el ítem de esta casilla, descuenta automáticamente los ingredientes de la cuadrícula de crafteo.
- **Guía de Modificación**: Gestiona el consumo de materiales de fabricación.
- **Métodos Clave Destacados**:
  - `boolean isItemValid(ItemStack itemstack)`
  - `void onPickupFromSlot()`


### 04. Entidades, Mobs y Físicas (Entities, Living Mobs, Vehicles & AI)
> **Resumen del subsistema**: Objetos dinámicos en el espacio 3D. Abarca al jugador, criaturas pasivas y hostiles, proyectiles (flechas, bolas de nieve), vehículos (vagonetas, barcas), entidades de bloques que caen (arena/grava), dinamita activa y el sistema de navegación por nodos A* (Pathfinder).

**Total de clases**: 60

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`Entity`](#class-entity) | `class` | `-` | 1057 | Clase base abstracta de todas las entidades y objetos físicos tri... |
| [`EntityAnimals`](#class-entityanimals) | `class` | `ext EntityCreature` | 48 | Entidad dinámica 'EntityAnimals' en el espacio 3D. |
| [`EntityArrow`](#class-entityarrow) | `class` | `ext Entity` | 287 | Proyectil de flecha con física balística y anclaje en bloques. |
| [`EntityBoat`](#class-entityboat) | `class` | `ext Entity` | 344 | Entidad dinámica 'EntityBoat' en el espacio 3D. |
| [`EntityBubbleFX`](#class-entitybubblefx) | `class` | `ext EntityFX` | 46 | Entidad dinámica 'EntityBubbleFX' en el espacio 3D. |
| [`EntityChicken`](#class-entitychicken) | `class` | `ext EntityAnimals` | 97 | Criatura pasiva de gallina que pone huevos y aletea al caer. |
| [`EntityClientPlayerMP`](#class-entityclientplayermp) | `class` | `ext EntityPlayerSP` | 183 | Entidad dinámica 'EntityClientPlayerMP' en el espacio 3D. |
| [`EntityCow`](#class-entitycow) | `class` | `ext EntityAnimals` | 67 | Criatura pasiva de vaca ordeñable con cubos vacíos. |
| [`EntityCreature`](#class-entitycreature) | `class` | `ext EntityLiving` | 170 | Entidad dinámica 'EntityCreature' en el espacio 3D. |
| [`EntityCreeper`](#class-entitycreeper) | `class` | `ext EntityMobs` | 153 | Mob hostil icónico: Creeper con detonación suicida por proximidad... |
| [`EntityDiggingFX`](#class-entitydiggingfx) | `class` | `ext EntityFX` | 60 | Entidad dinámica 'EntityDiggingFX' en el espacio 3D. |
| [`EntityExplodeFX`](#class-entityexplodefx) | `class` | `ext EntityFX` | 49 | Entidad dinámica 'EntityExplodeFX' en el espacio 3D. |
| [`EntityFX`](#class-entityfx) | `class` | `ext Entity` | 117 | Entidad dinámica 'EntityFX' en el espacio 3D. |
| [`EntityFallingSand`](#class-entityfallingsand) | `class` | `ext Entity` | 102 | Entidad dinámica 'EntityFallingSand' en el espacio 3D. |
| [`EntityFireball`](#class-entityfireball) | `class` | `ext Entity` | 237 | Entidad dinámica 'EntityFireball' en el espacio 3D. |
| [`EntityFish`](#class-entityfish) | `class` | `ext Entity` | 390 | Entidad dinámica 'EntityFish' en el espacio 3D. |
| [`EntityFlameFX`](#class-entityflamefx) | `class` | `ext EntityFX` | 71 | Entidad dinámica 'EntityFlameFX' en el espacio 3D. |
| [`EntityFlying`](#class-entityflying) | `class` | `ext EntityLiving` | 81 | Entidad dinámica 'EntityFlying' en el espacio 3D. |
| [`EntityGhast`](#class-entityghast) | `class` | `ext EntityFlying, impl IMobs
` | 173 | Mob hostil gigante volador del Inframundo (Nether). |
| [`EntityItem`](#class-entityitem) | `class` | `ext Entity` | 219 | Entidad dinámica 'EntityItem' en el espacio 3D. |
| [`EntityLavaFX`](#class-entitylavafx) | `class` | `ext EntityFX` | 65 | Entidad dinámica 'EntityLavaFX' en el espacio 3D. |
| [`EntityList`](#class-entitylist) | `class` | `-` | 143 | Entidad dinámica 'EntityList' en el espacio 3D. |
| [`EntityLiving`](#class-entityliving) | `class` | `ext Entity` | 867 | Clase base para criaturas vivientes (con salud, respiración e IA)... |
| [`EntityMinecart`](#class-entityminecart) | `class` | `ext Entity, impl IInventory
` | 855 | Entidad dinámica 'EntityMinecart' en el espacio 3D. |
| [`EntityMobs`](#class-entitymobs) | `class` | `ext EntityCreature, impl IMobs
` | 109 | Entidad dinámica 'EntityMobs' en el espacio 3D. |
| [`EntityOtherPlayerMP`](#class-entityotherplayermp) | `class` | `ext EntityPlayer` | 112 | Entidad dinámica 'EntityOtherPlayerMP' en el espacio 3D. |
| [`EntityPainting`](#class-entitypainting) | `class` | `ext Entity` | 238 | Entidad dinámica 'EntityPainting' en el espacio 3D. |
| [`EntityPickupFX`](#class-entitypickupfx) | `class` | `ext EntityFX` | 65 | Entidad dinámica 'EntityPickupFX' en el espacio 3D. |
| [`EntityPig`](#class-entitypig) | `class` | `ext EntityAnimals` | 64 | Criatura pasiva de cerdo montable con silla de montar. |
| [`EntityPigZombie`](#class-entitypigzombie) | `class` | `ext EntityZombie` | 126 | Hombre cerdo zombi neutral del Nether con agresión colectiva en m... |
| [`EntityPlayer`](#class-entityplayer) | `class` | `ext EntityLiving` | 342 | Representación del jugador dentro del mundo de Minecraft. |
| [`EntityPlayerSP`](#class-entityplayersp) | `class` | `ext EntityPlayer` | 200 | Instancia del jugador local en el cliente (procesa entrada de tec... |
| [`EntityPortalFX`](#class-entityportalfx) | `class` | `ext EntityFX` | 72 | Entidad dinámica 'EntityPortalFX' en el espacio 3D. |
| [`EntityRainFX`](#class-entityrainfx) | `class` | `ext EntityFX` | 64 | Entidad dinámica 'EntityRainFX' en el espacio 3D. |
| [`EntityReddustFX`](#class-entityreddustfx) | `class` | `ext EntityFX` | 74 | Entidad dinámica 'EntityReddustFX' en el espacio 3D. |
| [`EntityRenderer`](#class-entityrenderer) | `class` | `-` | 641 | Entidad dinámica 'EntityRenderer' en el espacio 3D. |
| [`EntitySheep`](#class-entitysheep) | `class` | `ext EntityAnimals` | 65 | Criatura pasiva de oveja con lana esquilable y tintada. |
| [`EntitySkeleton`](#class-entityskeleton) | `class` | `ext EntityMobs` | 93 | Mob hostil a distancia armado con arco y flechas. |
| [`EntitySlime`](#class-entityslime) | `class` | `ext EntityLiving, impl IMobs
` | 163 | Entidad dinámica 'EntitySlime' en el espacio 3D. |
| [`EntitySlimeFX`](#class-entityslimefx) | `class` | `ext EntityFX` | 42 | Entidad dinámica 'EntitySlimeFX' en el espacio 3D. |
| [`EntitySmokeFX`](#class-entitysmokefx) | `class` | `ext EntityFX` | 74 | Entidad dinámica 'EntitySmokeFX' en el espacio 3D. |
| [`EntitySnowball`](#class-entitysnowball) | `class` | `ext Entity` | 270 | Entidad dinámica 'EntitySnowball' en el espacio 3D. |
| [`EntitySorter`](#class-entitysorter) | `class` | `impl Comparator
` | 28 | Entidad dinámica 'EntitySorter' en el espacio 3D. |
| [`EntitySpider`](#class-entityspider) | `class` | `ext EntityMobs` | 91 | Mob hostil arácnido con capacidad para escalar paredes verticales... |
| [`EntitySplashFX`](#class-entitysplashfx) | `class` | `ext EntityRainFX` | 23 | Entidad dinámica 'EntitySplashFX' en el espacio 3D. |
| [`EntityTNTPrimed`](#class-entitytntprimed) | `class` | `ext Entity` | 87 | Entidad dinámica 'EntityTNTPrimed' en el espacio 3D. |
| [`EntityZombie`](#class-entityzombie) | `class` | `ext EntityMobs` | 51 | Mob hostil cuerpo a cuerpo que arde bajo la luz del sol. |
| [`EntityZombieSimple`](#class-entityzombiesimple) | `class` | `ext EntityMobs` | 25 | Entidad dinámica 'EntityZombieSimple' en el espacio 3D. |
| [`EnumCreatureType`](#class-enumcreaturetype) | `enum` | `-` | 45 | Clasificación biológica de criaturas (Monstruos, Criaturas pasiva... |
| [`EnumMobType`](#class-enummobtype) | `enum` | `-` | 43 | Clasificación del tipo de mob según comportamiento. |
| [`IMobs`](#class-imobs) | `interface` | `-` | 9 | Interfaz marcadora que identifica a una entidad como monstruo hos... |
| [`MobSpawnerBase`](#class-mobspawnerbase) | `class` | `-` | 175 | Clase base para la ecología de spawn de criaturas en biomas. |
| [`MobSpawnerDesert`](#class-mobspawnerdesert) | `class` | `ext MobSpawnerBase` | 13 | Definición ecológica para el bioma de Desierto. |
| [`MobSpawnerHell`](#class-mobspawnerhell) | `class` | `ext MobSpawnerBase` | 17 | Definición ecológica para el Inframundo (Nether). |
| [`MobSpawnerSwamp`](#class-mobspawnerswamp) | `class` | `ext MobSpawnerBase` | 13 | Definición ecológica para el bioma de Pantano. |
| [`Path`](#class-path) | `class` | `-` | 144 | Encapsulador de una ruta de navegación calculada para criaturas. |
| [`PathEntity`](#class-pathentity) | `class` | `-` | 37 | Secuencia ordenada de puntos de ruta (PathPoint) asignada a una e... |
| [`PathPoint`](#class-pathpoint) | `class` | `-` | 58 | Nodo espacial tridimensional con coste de recorrido para el algor... |
| [`Pathfinder`](#class-pathfinder) | `class` | `-` | 218 | Motor de búsqueda de caminos sobre la malla de bloques navegables... |
| [`SpawnerAnimals`](#class-spawneranimals) | `class` | `-` | 125 | Motor biológico de spawn periódico de criaturas. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-entity"></a>[`Entity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Entity.java)
- **Ruta y Paquete**: `net.minecraft.src.Entity` (`net/minecraft/src/Entity.java`) | **Líneas de código**: 1057
- **Declaración**: `class Entity`
- **Rol**: Clase base abstracta de todas las entidades y objetos físicos tridimensionales.
- **Descripción Técnica**: La entidad raíz del motor. Gestiona posición (posX, posY, posZ), velocidad vectorial (motionX, motionY, motionZ), orientación (rotationYaw, rotationPitch), caja de colisión (AxisAlignedBB), estado en el suelo (onGround), fuego, inmersión en fluidos y persistencia NBT.
- **Guía de Modificación**: Cualquier nuevo mob, proyectil o vehículo debe heredar directa o indirectamente de Entity e implementar writeToNBT/readFromNBT.
- **Métodos Clave Destacados**:
  - `boolean equals(Object obj)`
  - `int hashCode()`
  - `void preparePlayerToSpawn()`
  - `void setEntityDead()`
  - `void setSize(float f, float f1)`
  - `void setRotation(float f, float f1)`
- **Campos Clave Destacados**:
  - `int field_620_ab`
  - `double field_619_ac`
  - `boolean field_618_ad`
  - `Entity riddenByEntity`
  - `Entity ridingEntity`
  - `World worldObj`

##### <a id="class-entityanimals"></a>[`EntityAnimals`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityAnimals.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityAnimals` (`net/minecraft/src/EntityAnimals.java`) | **Líneas de código**: 48
- **Declaración**: `class EntityAnimals extends EntityCreature`
- **Rol**: Entidad dinámica 'EntityAnimals' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityAnimals'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `float func_439_a(int i, int j, int k)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `boolean getCanSpawnHere()`
  - `int func_421_b()`

##### <a id="class-entityarrow"></a>[`EntityArrow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityArrow.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityArrow` (`net/minecraft/src/EntityArrow.java`) | **Líneas de código**: 287
- **Declaración**: `class EntityArrow extends Entity`
- **Rol**: Proyectil de flecha con física balística y anclaje en bloques.
- **Descripción Técnica**: Calcula la trayectoria parabólica con gravedad y resistencia del aire. Realiza raycasting continuo; al impactar en una criatura le inflige daño, y al chocar contra un bloque se clava físicamente pudiendo ser recogida.
- **Guía de Modificación**: Modifica la velocidad inicial, gravedad o añade efectos de impacto (explosión, fuego, teletransporte).
- **Métodos Clave Destacados**:
  - `void func_408_a(double d, double d1, double d2, float f, float f1)`
  - `void setVelocity(double d, double d1, double d2)`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `void onCollideWithPlayer(EntityPlayer entityplayer)`
- **Campos Clave Destacados**:
  - `int arrowShake`
  - `EntityLiving field_682_g`

##### <a id="class-entityboat"></a>[`EntityBoat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityBoat.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityBoat` (`net/minecraft/src/EntityBoat.java`) | **Líneas de código**: 344
- **Declaración**: `class EntityBoat extends Entity`
- **Rol**: Entidad dinámica 'EntityBoat' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityBoat'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB func_383_b_(Entity entity)`
  - `AxisAlignedBB func_372_f_()`
  - `boolean canBePushed()`
  - `double func_402_h()`
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void func_9280_g()`
- **Campos Clave Destacados**:
  - `int field_807_a`
  - `int field_806_b`
  - `int field_808_c`

##### <a id="class-entitybubblefx"></a>[`EntityBubbleFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityBubbleFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityBubbleFX` (`net/minecraft/src/EntityBubbleFX.java`) | **Líneas de código**: 46
- **Declaración**: `class EntityBubbleFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityBubbleFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityBubbleFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void onUpdate()`

##### <a id="class-entitychicken"></a>[`EntityChicken`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityChicken.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityChicken` (`net/minecraft/src/EntityChicken.java`) | **Líneas de código**: 97
- **Declaración**: `class EntityChicken extends EntityAnimals`
- **Rol**: Criatura pasiva de gallina que pone huevos y aletea al caer.
- **Descripción Técnica**: Tiene caída lenta sin daño de caída aleteando sus alas. En cada intervalo de tiempo (timeUntilNextEgg) pone un huevo (Item.egg).
- **Guía de Modificación**: Modifica la frecuencia de puesta de huevos o los drops cárnicos.
- **Métodos Clave Destacados**:
  - `void onLivingUpdate()`
  - `void fall(float f)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `String getLivingSound()`
  - `String getHurtSound()`
- **Campos Clave Destacados**:
  - `boolean field_753_a`
  - `float field_752_b`
  - `float field_758_c`
  - `float field_757_d`
  - `float field_756_e`
  - `float field_755_h`

##### <a id="class-entityclientplayermp"></a>[`EntityClientPlayerMP`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityClientPlayerMP.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityClientPlayerMP` (`net/minecraft/src/EntityClientPlayerMP.java`) | **Líneas de código**: 183
- **Declaración**: `class EntityClientPlayerMP extends EntityPlayerSP`
- **Rol**: Entidad dinámica 'EntityClientPlayerMP' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityClientPlayerMP'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void heal(int i)`
  - `void onUpdate()`
  - `void func_6420_o()`
  - `void func_4056_N()`
  - `void joinEntityItemWithWorld(EntityItem entityitem)`

##### <a id="class-entitycow"></a>[`EntityCow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityCow.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityCow` (`net/minecraft/src/EntityCow.java`) | **Líneas de código**: 67
- **Declaración**: `class EntityCow extends EntityAnimals`
- **Rol**: Criatura pasiva de vaca ordeñable con cubos vacíos.
- **Descripción Técnica**: Al hacer clic derecho con un cubo vacío (Item.bucketEmpty), llena el cubo con leche (Item.bucketMilk). Al morir suelta cuero.
- **Guía de Modificación**: Modifica interact() para recolectar nuevos productos de la vaca.
- **Métodos Clave Destacados**:
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `float getSoundVolume()`
- **Campos Clave Destacados**:
  - `boolean unusedBoolean`

##### <a id="class-entitycreature"></a>[`EntityCreature`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityCreature.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityCreature` (`net/minecraft/src/EntityCreature.java`) | **Líneas de código**: 170
- **Declaración**: `class EntityCreature extends EntityLiving`
- **Rol**: Entidad dinámica 'EntityCreature' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityCreature'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_418_b_()`
  - `void attackEntity(Entity entity, float f)`
  - `float func_439_a(int i, int j, int k)`
  - `Entity findPlayerToAttack()`
  - `boolean getCanSpawnHere()`
- **Campos Clave Destacados**:
  - `Entity playerToAttack`
  - `boolean hasAttacked`

##### <a id="class-entitycreeper"></a>[`EntityCreeper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityCreeper.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityCreeper` (`net/minecraft/src/EntityCreeper.java`) | **Líneas de código**: 153
- **Declaración**: `class EntityCreeper extends EntityMobs`
- **Rol**: Mob hostil icónico: Creeper con detonación suicida por proximidad.
- **Descripción Técnica**: Detecta al jugador en un radio cercano, silba mientras se hincha visualmente durante 30 ticks (timeSinceIgnited), y detona una explosión de potencia 3.0 al completar el temporizador. Si es alcanzado por un rayo, se sobrecarga.
- **Guía de Modificación**: Modifica fuseTime o el radio de explosión para crear creepers con detonación modificada.
- **Métodos Clave Destacados**:
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `void func_9282_a(byte byte0)`
  - `void onUpdate()`
  - `void func_418_b_()`
  - `String getHurtSound()`

##### <a id="class-entitydiggingfx"></a>[`EntityDiggingFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityDiggingFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityDiggingFX` (`net/minecraft/src/EntityDiggingFX.java`) | **Líneas de código**: 60
- **Declaración**: `class EntityDiggingFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityDiggingFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityDiggingFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `EntityDiggingFX func_4041_a(int i, int j, int k)`
  - `int func_404_c()`
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-entityexplodefx"></a>[`EntityExplodeFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityExplodeFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityExplodeFX` (`net/minecraft/src/EntityExplodeFX.java`) | **Líneas de código**: 49
- **Declaración**: `class EntityExplodeFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityExplodeFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityExplodeFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`

##### <a id="class-entityfx"></a>[`EntityFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFX` (`net/minecraft/src/EntityFX.java`) | **Líneas de código**: 117
- **Declaración**: `class EntityFX extends Entity`
- **Rol**: Entidad dinámica 'EntityFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `EntityFX func_407_b(float f)`
  - `EntityFX func_405_d(float f)`
  - `void onUpdate()`
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `int func_404_c()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int field_670_b`
  - `float field_669_c`
  - `float field_668_d`
  - `int field_667_e`
  - `int field_666_f`
  - `float field_665_g`

##### <a id="class-entityfallingsand"></a>[`EntityFallingSand`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFallingSand.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFallingSand` (`net/minecraft/src/EntityFallingSand.java`) | **Líneas de código**: 102
- **Declaración**: `class EntityFallingSand extends Entity`
- **Rol**: Entidad dinámica 'EntityFallingSand' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFallingSand'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean canBeCollidedWith()`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `float func_392_h_()`
  - `World func_465_i()`
- **Campos Clave Destacados**:
  - `int field_799_a`
  - `int field_798_b`

##### <a id="class-entityfireball"></a>[`EntityFireball`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFireball.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFireball` (`net/minecraft/src/EntityFireball.java`) | **Líneas de código**: 237
- **Declaración**: `class EntityFireball extends Entity`
- **Rol**: Entidad dinámica 'EntityFireball' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFireball'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean func_384_a(double d)`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `boolean canBeCollidedWith()`
  - `float func_4035_j_()`
- **Campos Clave Destacados**:
  - `int field_9406_a`
  - `double field_9405_b`
  - `double field_9404_c`
  - `double field_9403_d`

##### <a id="class-entityfish"></a>[`EntityFish`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFish.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFish` (`net/minecraft/src/EntityFish.java`) | **Líneas de código**: 390
- **Declaración**: `class EntityFish extends Entity`
- **Rol**: Entidad dinámica 'EntityFish' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFish'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean func_384_a(double d)`
  - `void func_4042_a(double d, double d1, double d2, float f, float f1)`
  - `void setPositionAndRotation2(double d, double d1, double d2, float f, float f1, int i)`
  - `void setVelocity(double d, double d1, double d2)`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int field_4098_a`
  - `EntityPlayer field_4097_b`
  - `Entity field_4096_c`

##### <a id="class-entityflamefx"></a>[`EntityFlameFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFlameFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFlameFX` (`net/minecraft/src/EntityFlameFX.java`) | **Líneas de código**: 71
- **Declaración**: `class EntityFlameFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityFlameFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFlameFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `float getEntityBrightness(float f)`
  - `void onUpdate()`

##### <a id="class-entityflying"></a>[`EntityFlying`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityFlying.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityFlying` (`net/minecraft/src/EntityFlying.java`) | **Líneas de código**: 81
- **Declaración**: `class EntityFlying extends EntityLiving`
- **Rol**: Entidad dinámica 'EntityFlying' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityFlying'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void fall(float f)`
  - `void func_435_b(float f, float f1)`
  - `boolean isOnLadder()`

##### <a id="class-entityghast"></a>[`EntityGhast`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityGhast.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityGhast` (`net/minecraft/src/EntityGhast.java`) | **Líneas de código**: 173
- **Declaración**: `class EntityGhast extends EntityFlying implements IMobs
`
- **Rol**: Mob hostil gigante volador del Inframundo (Nether).
- **Descripción Técnica**: Vuela flotando por los espacios cavernosos del Nether, emite alaridos espectrales y dispara proyectiles ígneos explosivos cuando tiene línea de visión con el jugador.
- **Guía de Modificación**: Ajusta la trayectoria de vuelo, la frecuencia de disparo o los proyectiles.
- **Métodos Clave Destacados**:
  - `void func_418_b_()`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `int getDropItemId()`
  - `float getSoundVolume()`
- **Campos Clave Destacados**:
  - `int field_4121_a`
  - `double field_4120_b`
  - `double field_4127_c`
  - `double field_4126_d`
  - `int field_4125_e`
  - `int field_4124_f`

##### <a id="class-entityitem"></a>[`EntityItem`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityItem.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityItem` (`net/minecraft/src/EntityItem.java`) | **Líneas de código**: 219
- **Declaración**: `class EntityItem extends Entity`
- **Rol**: Entidad dinámica 'EntityItem' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityItem'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void onUpdate()`
  - `boolean handleWaterMovement()`
  - `void func_355_a(int i)`
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `ItemStack item`
  - `int age`
  - `int field_805_c`
  - `float field_804_d`

##### <a id="class-entitylavafx"></a>[`EntityLavaFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityLavaFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityLavaFX` (`net/minecraft/src/EntityLavaFX.java`) | **Líneas de código**: 65
- **Declaración**: `class EntityLavaFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityLavaFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityLavaFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `float getEntityBrightness(float f)`
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`

##### <a id="class-entitylist"></a>[`EntityList`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityList.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityList` (`net/minecraft/src/EntityList.java`) | **Líneas de código**: 143
- **Declaración**: `class EntityList`
- **Rol**: Entidad dinámica 'EntityList' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityList'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `Entity createEntityInWorld(String s, World world)`
  - `Entity createEntityFromNBT(NBTTagCompound nbttagcompound, World world)`
  - `Entity createEntity(int i, World world)`
  - `int getEntityID(Entity entity)`
  - `String getEntityString(Entity entity)`

##### <a id="class-entityliving"></a>[`EntityLiving`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityLiving.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityLiving` (`net/minecraft/src/EntityLiving.java`) | **Líneas de código**: 867
- **Declaración**: `class EntityLiving extends Entity`
- **Rol**: Clase base para criaturas vivientes (con salud, respiración e IA).
- **Descripción Técnica**: Extiende Entity añadiendo salud (health), aire bajo el agua, cálculo de daño recibido con invulnerabilidad temporal (hurtTime), animación de ataque (swingProgress), movimiento con inercia, salto y drops al morir en onDeath.
- **Guía de Modificación**: Base fundamental para crear cualquier mob personalizado.
- **Métodos Clave Destacados**:
  - `boolean canEntityBeSeen(Entity entity)`
  - `String getEntityTexture()`
  - `boolean canBeCollidedWith()`
  - `boolean canBePushed()`
  - `float func_373_s()`
  - `int func_421_b()`
- **Campos Clave Destacados**:
  - `int field_9366_o`
  - `float field_9365_p`
  - `float field_9364_q`
  - `float field_9363_r`
  - `float field_735_n`
  - `float field_734_o`

##### <a id="class-entityminecart"></a>[`EntityMinecart`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityMinecart.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityMinecart` (`net/minecraft/src/EntityMinecart.java`) | **Líneas de código**: 855
- **Declaración**: `class EntityMinecart extends Entity implements IInventory
`
- **Rol**: Entidad dinámica 'EntityMinecart' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityMinecart'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB func_383_b_(Entity entity)`
  - `AxisAlignedBB func_372_f_()`
  - `boolean canBePushed()`
  - `double func_402_h()`
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void func_9280_g()`
- **Campos Clave Destacados**:
  - `int field_864_a`
  - `int field_863_b`
  - `int field_862_c`
  - `int minecartType`
  - `int fuel`
  - `double pushX`

##### <a id="class-entitymobs"></a>[`EntityMobs`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityMobs.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityMobs` (`net/minecraft/src/EntityMobs.java`) | **Líneas de código**: 109
- **Declaración**: `class EntityMobs extends EntityCreature implements IMobs
`
- **Rol**: Entidad dinámica 'EntityMobs' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityMobs'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void onLivingUpdate()`
  - `void onUpdate()`
  - `Entity findPlayerToAttack()`
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void attackEntity(Entity entity, float f)`
  - `float func_439_a(int i, int j, int k)`
- **Campos Clave Destacados**:
  - `int field_762_e`

##### <a id="class-entityotherplayermp"></a>[`EntityOtherPlayerMP`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityOtherPlayerMP.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityOtherPlayerMP` (`net/minecraft/src/EntityOtherPlayerMP.java`) | **Líneas de código**: 112
- **Declaración**: `class EntityOtherPlayerMP extends EntityPlayer`
- **Rol**: Entidad dinámica 'EntityOtherPlayerMP' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityOtherPlayerMP'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void setPositionAndRotation2(double d, double d1, double d2, float f, float f1, int i)`
  - `void onUpdate()`
  - `float func_392_h_()`
  - `void onLivingUpdate()`
  - `boolean func_381_o()`

##### <a id="class-entitypainting"></a>[`EntityPainting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPainting.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPainting` (`net/minecraft/src/EntityPainting.java`) | **Líneas de código**: 238
- **Declaración**: `class EntityPainting extends Entity`
- **Rol**: Entidad dinámica 'EntityPainting' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityPainting'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_412_b(int i)`
  - `void onUpdate()`
  - `boolean func_410_i()`
  - `boolean canBeCollidedWith()`
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int field_691_a`
  - `EnumArt field_690_b`

##### <a id="class-entitypickupfx"></a>[`EntityPickupFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPickupFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPickupFX` (`net/minecraft/src/EntityPickupFX.java`) | **Líneas de código**: 65
- **Declaración**: `class EntityPickupFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityPickupFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityPickupFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`
  - `int func_404_c()`

##### <a id="class-entitypig"></a>[`EntityPig`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPig.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPig` (`net/minecraft/src/EntityPig.java`) | **Líneas de código**: 64
- **Declaración**: `class EntityPig extends EntityAnimals`
- **Rol**: Criatura pasiva de cerdo montable con silla de montar.
- **Descripción Técnica**: Camina pasivamente evitando precipicios. Si se le coloca una silla de montar (ItemSaddle), el jugador puede subirse sobre él.
- **Guía de Modificación**: Modifica interact() para cambiar las condiciones de montura o doma.
- **Métodos Clave Destacados**:
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `boolean interact(EntityPlayer entityplayer)`
- **Campos Clave Destacados**:
  - `boolean rideable`

##### <a id="class-entitypigzombie"></a>[`EntityPigZombie`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPigZombie.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPigZombie` (`net/minecraft/src/EntityPigZombie.java`) | **Líneas de código**: 126
- **Declaración**: `class EntityPigZombie extends EntityZombie`
- **Rol**: Hombre cerdo zombi neutral del Nether con agresión colectiva en manada.
- **Descripción Técnica**: Camina pasivamente por el Nether armado con una espada de oro. Si el jugador ataca a uno, todos los PigZombies en un amplio radio se vuelven inmediatamente hostiles.
- **Guía de Modificación**: Estudia su sistema de venganza en manada para implementarlo en otras criaturas.
- **Métodos Clave Destacados**:
  - `void onUpdate()`
  - `boolean getCanSpawnHere()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `Entity findPlayerToAttack()`
  - `void onLivingUpdate()`

##### <a id="class-entityplayer"></a>[`EntityPlayer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPlayer.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPlayer` (`net/minecraft/src/EntityPlayer.java`) | **Líneas de código**: 342
- **Declaración**: `class EntityPlayer extends EntityLiving`
- **Rol**: Representación del jugador dentro del mundo de Minecraft.
- **Descripción Técnica**: Controla el inventario completo (InventoryPlayer), sistema de puntuación, interacción con bloques y mobs, capacidad de montar vehículos o cerdos, y cámara en primera y tercera persona.
- **Guía de Modificación**: Punto de enganche para añadir mecánicas RPG, habilidades o inventarios extendidos.
- **Métodos Clave Destacados**:
  - `void func_350_p()`
  - `void preparePlayerToSpawn()`
  - `void func_418_b_()`
  - `void onLivingUpdate()`
  - `int func_6417_t()`
  - `void onDeath(Entity entity)`
- **Campos Clave Destacados**:
  - `InventoryPlayer inventory`
  - `byte field_9371_f`
  - `int score`
  - `float field_775_e`
  - `float field_774_f`
  - `boolean field_9369_j`

##### <a id="class-entityplayersp"></a>[`EntityPlayerSP`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPlayerSP.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPlayerSP` (`net/minecraft/src/EntityPlayerSP.java`) | **Líneas de código**: 200
- **Declaración**: `class EntityPlayerSP extends EntityPlayer`
- **Rol**: Instancia del jugador local en el cliente (procesa entrada de teclado/ratón).
- **Descripción Técnica**: Conecta la entrada física del usuario (MovementInput) con la cinemática de EntityPlayer. En multijugador, empaqueta el movimiento en paquetes de red Packet10Flying.
- **Guía de Modificación**: Permite modificar el control de movimiento, vuelo o interceptar comandos del cliente.
- **Métodos Clave Destacados**:
  - `void func_418_b_()`
  - `void onLivingUpdate()`
  - `void func_458_k()`
  - `void func_460_a(int i, boolean flag)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `MovementInput field_787_a`
  - `int field_9373_b`
  - `float field_4134_c`
  - `float field_4133_d`

##### <a id="class-entityportalfx"></a>[`EntityPortalFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityPortalFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityPortalFX` (`net/minecraft/src/EntityPortalFX.java`) | **Líneas de código**: 72
- **Declaración**: `class EntityPortalFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityPortalFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityPortalFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `float getEntityBrightness(float f)`
  - `void onUpdate()`

##### <a id="class-entityrainfx"></a>[`EntityRainFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityRainFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityRainFX` (`net/minecraft/src/EntityRainFX.java`) | **Líneas de código**: 64
- **Declaración**: `class EntityRainFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityRainFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityRainFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`

##### <a id="class-entityreddustfx"></a>[`EntityReddustFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityReddustFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityReddustFX` (`net/minecraft/src/EntityReddustFX.java`) | **Líneas de código**: 74
- **Declaración**: `class EntityReddustFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntityReddustFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityReddustFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`

##### <a id="class-entityrenderer"></a>[`EntityRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityRenderer` (`net/minecraft/src/EntityRenderer.java`) | **Líneas de código**: 641
- **Declaración**: `class EntityRenderer`
- **Rol**: Entidad dinámica 'EntityRenderer' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityRenderer'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_911_a()`
  - `void func_910_a(float f)`
  - `void func_4136_b(float f)`
  - `void func_4134_c(float f)`
  - `void func_905_b()`
- **Campos Clave Destacados**:
  - `ItemRenderer field_1395_a`

##### <a id="class-entitysheep"></a>[`EntitySheep`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySheep.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySheep` (`net/minecraft/src/EntitySheep.java`) | **Líneas de código**: 65
- **Declaración**: `class EntitySheep extends EntityAnimals`
- **Rol**: Criatura pasiva de oveja con lana esquilable y tintada.
- **Descripción Técnica**: Almacena en metadatos el color de su lana y el estado esquilado (sheared). Al ser atacada o esquilada con tijeras suelta bloques de lana (Block.cloth).
- **Guía de Modificación**: Añade regeneración de lana al comer césped o nuevas paletas cromáticas.
- **Métodos Clave Destacados**:
  - `boolean canAttackEntity(Entity entity, int i)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
- **Campos Clave Destacados**:
  - `boolean sheared`

##### <a id="class-entityskeleton"></a>[`EntitySkeleton`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySkeleton.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySkeleton` (`net/minecraft/src/EntitySkeleton.java`) | **Líneas de código**: 93
- **Declaración**: `class EntitySkeleton extends EntityMobs`
- **Rol**: Mob hostil a distancia armado con arco y flechas.
- **Descripción Técnica**: Mantiene distancia con el jugador mientras dispara ráfagas de flechas (EntityArrow). También arde bajo la luz del sol diurna.
- **Guía de Modificación**: Modifica la cadencia de disparo o la precisión de sus proyectiles.
- **Métodos Clave Destacados**:
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `void onLivingUpdate()`
  - `void attackEntity(Entity entity, float f)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`

##### <a id="class-entityslime"></a>[`EntitySlime`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySlime.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySlime` (`net/minecraft/src/EntitySlime.java`) | **Líneas de código**: 163
- **Declaración**: `class EntitySlime extends EntityLiving implements IMobs
`
- **Rol**: Entidad dinámica 'EntitySlime' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySlime'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_441_c(int i)`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `void onUpdate()`
  - `void func_418_b_()`
  - `void setEntityDead()`
- **Campos Clave Destacados**:
  - `float field_768_a`
  - `float field_767_b`
  - `int field_770_c`

##### <a id="class-entityslimefx"></a>[`EntitySlimeFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySlimeFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySlimeFX` (`net/minecraft/src/EntitySlimeFX.java`) | **Líneas de código**: 42
- **Declaración**: `class EntitySlimeFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntitySlimeFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySlimeFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `int func_404_c()`
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-entitysmokefx"></a>[`EntitySmokeFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySmokeFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySmokeFX` (`net/minecraft/src/EntitySmokeFX.java`) | **Líneas de código**: 74
- **Declaración**: `class EntitySmokeFX extends EntityFX`
- **Rol**: Entidad dinámica 'EntitySmokeFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySmokeFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `void func_406_a(Tessellator tessellator, float f, float f1, float f2, float f3, float f4, float f5)`
  - `void onUpdate()`

##### <a id="class-entitysnowball"></a>[`EntitySnowball`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySnowball.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySnowball` (`net/minecraft/src/EntitySnowball.java`) | **Líneas de código**: 270
- **Declaración**: `class EntitySnowball extends Entity`
- **Rol**: Entidad dinámica 'EntitySnowball' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySnowball'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean func_384_a(double d)`
  - `void func_467_a(double d, double d1, double d2, float f, float f1)`
  - `void setVelocity(double d, double d1, double d2)`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int field_817_a`

##### <a id="class-entitysorter"></a>[`EntitySorter`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySorter.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySorter` (`net/minecraft/src/EntitySorter.java`) | **Líneas de código**: 28
- **Declaración**: `class EntitySorter implements Comparator
`
- **Rol**: Entidad dinámica 'EntitySorter' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySorter'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `int func_1063_a(WorldRenderer worldrenderer, WorldRenderer worldrenderer1)`
  - `int compare(Object obj, Object obj1)`

##### <a id="class-entityspider"></a>[`EntitySpider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySpider.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySpider` (`net/minecraft/src/EntitySpider.java`) | **Líneas de código**: 91
- **Declaración**: `class EntitySpider extends EntityMobs`
- **Rol**: Mob hostil arácnido con capacidad para escalar paredes verticales.
- **Descripción Técnica**: Detecta colisiones laterales con bloques para ascender verticalmente. En la oscuridad es agresiva, pero bajo la luz solar diurna se vuelve neutral a menos que sea atacada.
- **Guía de Modificación**: Modifica findPlayerToAttack para alterar su agresividad según la luminosidad.
- **Métodos Clave Destacados**:
  - `double func_402_h()`
  - `Entity findPlayerToAttack()`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `void attackEntity(Entity entity, float f)`

##### <a id="class-entitysplashfx"></a>[`EntitySplashFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntitySplashFX.java)
- **Ruta y Paquete**: `net.minecraft.src.EntitySplashFX` (`net/minecraft/src/EntitySplashFX.java`) | **Líneas de código**: 23
- **Declaración**: `class EntitySplashFX extends EntityRainFX`
- **Rol**: Entidad dinámica 'EntitySplashFX' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntitySplashFX'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.

##### <a id="class-entitytntprimed"></a>[`EntityTNTPrimed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityTNTPrimed.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityTNTPrimed` (`net/minecraft/src/EntityTNTPrimed.java`) | **Líneas de código**: 87
- **Declaración**: `class EntityTNTPrimed extends Entity`
- **Rol**: Entidad dinámica 'EntityTNTPrimed' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityTNTPrimed'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `boolean canBeCollidedWith()`
  - `void onUpdate()`
  - `void writeEntityToNBT(NBTTagCompound nbttagcompound)`
  - `void readEntityFromNBT(NBTTagCompound nbttagcompound)`
  - `float func_392_h_()`
- **Campos Clave Destacados**:
  - `int fuse`

##### <a id="class-entityzombie"></a>[`EntityZombie`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityZombie.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityZombie` (`net/minecraft/src/EntityZombie.java`) | **Líneas de código**: 51
- **Declaración**: `class EntityZombie extends EntityMobs`
- **Rol**: Mob hostil cuerpo a cuerpo que arde bajo la luz del sol.
- **Descripción Técnica**: Persigue al jugador en línea recta mediante Pathfinder, ataca por contacto directo y entra en combustión si se encuentra expuesto a la luz solar diurna.
- **Guía de Modificación**: Añade equipamiento, resistencia a la luz solar o drops especiales.
- **Métodos Clave Destacados**:
  - `void onLivingUpdate()`
  - `String getLivingSound()`
  - `String getHurtSound()`
  - `String getDeathSound()`
  - `int getDropItemId()`

##### <a id="class-entityzombiesimple"></a>[`EntityZombieSimple`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EntityZombieSimple.java)
- **Ruta y Paquete**: `net.minecraft.src.EntityZombieSimple` (`net/minecraft/src/EntityZombieSimple.java`) | **Líneas de código**: 25
- **Declaración**: `class EntityZombieSimple extends EntityMobs`
- **Rol**: Entidad dinámica 'EntityZombieSimple' en el espacio 3D.
- **Descripción Técnica**: Gestiona el movimiento, colisiones, renderizado y estados de vida para la entidad 'EntityZombieSimple'. Sincroniza su posición y datos mediante etiquetas NBT.
- **Guía de Modificación**: Sobrescribe onUpdate() para modificar su comportamiento tick a tick o su IA.
- **Métodos Clave Destacados**:
  - `float func_439_a(int i, int j, int k)`

##### <a id="class-enumcreaturetype"></a>[`EnumCreatureType`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumCreatureType.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumCreatureType` (`net/minecraft/src/EnumCreatureType.java`) | **Líneas de código**: 45
- **Declaración**: `enum EnumCreatureType`
- **Rol**: Clasificación biológica de criaturas (Monstruos, Criaturas pasivas, Acuáticas).
- **Descripción Técnica**: Establece el límite máximo de entidades simultáneas por chunk y el tipo de material donde pueden spawnear.
- **Guía de Modificación**: Añade nuevas categorías ecológicas si desarrollas mods con fauna exótica.
- **Métodos Clave Destacados**:
  - `EnumCreatureType[] func_4145_values()`
  - `EnumCreatureType valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumCreatureType monster`
  - `EnumCreatureType creature`
  - `Class field_4278_c`
  - `int maxNumberOfEntityType`

##### <a id="class-enummobtype"></a>[`EnumMobType`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumMobType.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumMobType` (`net/minecraft/src/EnumMobType.java`) | **Líneas de código**: 43
- **Declaración**: `enum EnumMobType`
- **Rol**: Clasificación del tipo de mob según comportamiento.
- **Descripción Técnica**: Distingue entre entidades pasivas y hostiles para la asignación de objetivos.
- **Guía de Modificación**: Utilizado en la lógica de IA y selección de objetivos.
- **Métodos Clave Destacados**:
  - `EnumMobType[] func_4131_values()`
  - `EnumMobType valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumMobType everything`
  - `EnumMobType mobs`
  - `EnumMobType players`

##### <a id="class-imobs"></a>[`IMobs`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IMobs.java)
- **Ruta y Paquete**: `net.minecraft.src.IMobs` (`net/minecraft/src/IMobs.java`) | **Líneas de código**: 9
- **Declaración**: `interface IMobs`
- **Rol**: Interfaz marcadora que identifica a una entidad como monstruo hostil.
- **Descripción Técnica**: Utilizada por el selector de objetivos de criaturas y por la dificultad para permitir o prohibir su spawn.
- **Guía de Modificación**: Implementar en cualquier criatura personalizada que deba comportarse como monstruo.

##### <a id="class-mobspawnerbase"></a>[`MobSpawnerBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MobSpawnerBase.java)
- **Ruta y Paquete**: `net.minecraft.src.MobSpawnerBase` (`net/minecraft/src/MobSpawnerBase.java`) | **Líneas de código**: 175
- **Declaración**: `class MobSpawnerBase`
- **Rol**: Clase base para la ecología de spawn de criaturas en biomas.
- **Descripción Técnica**: Define las listas de monstruos agresivos, criaturas pacíficas y criaturas acuáticas que pueden aparecer.
- **Guía de Modificación**: Punto de inyección para que nuevos mobs aparezcan naturalmente en el mundo.
- **Métodos Clave Destacados**:
  - `void generateBiomeLookup()`
  - `MobSpawnerBase func_4122_b()`
  - `MobSpawnerBase setBiomeName(String s)`
  - `MobSpawnerBase func_4124_a(int i)`
  - `MobSpawnerBase func_4123_b(int i)`
  - `MobSpawnerBase getBiomeFromLookup(double d, double d1)`
- **Campos Clave Destacados**:
  - `MobSpawnerBase rainforest`
  - `MobSpawnerBase swampland`
  - `MobSpawnerBase seasonalForest`
  - `MobSpawnerBase forest`
  - `MobSpawnerBase savanna`
  - `MobSpawnerBase shrubland`

##### <a id="class-mobspawnerdesert"></a>[`MobSpawnerDesert`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MobSpawnerDesert.java)
- **Ruta y Paquete**: `net.minecraft.src.MobSpawnerDesert` (`net/minecraft/src/MobSpawnerDesert.java`) | **Líneas de código**: 13
- **Declaración**: `class MobSpawnerDesert extends MobSpawnerBase`
- **Rol**: Definición ecológica para el bioma de Desierto.
- **Descripción Técnica**: Restringe la aparición de animales pasivos y ajusta los monstruos nocturnos.
- **Guía de Modificación**: Ejemplo de especialización bioclimática.

##### <a id="class-mobspawnerhell"></a>[`MobSpawnerHell`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MobSpawnerHell.java)
- **Ruta y Paquete**: `net.minecraft.src.MobSpawnerHell` (`net/minecraft/src/MobSpawnerHell.java`) | **Líneas de código**: 17
- **Declaración**: `class MobSpawnerHell extends MobSpawnerBase`
- **Rol**: Definición ecológica para el Inframundo (Nether).
- **Descripción Técnica**: Configura la aparición exclusiva de Ghasts y PigZombies en el Inframundo.
- **Guía de Modificación**: Añadir aquí nuevos mobs del Nether.

##### <a id="class-mobspawnerswamp"></a>[`MobSpawnerSwamp`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MobSpawnerSwamp.java)
- **Ruta y Paquete**: `net.minecraft.src.MobSpawnerSwamp` (`net/minecraft/src/MobSpawnerSwamp.java`) | **Líneas de código**: 13
- **Declaración**: `class MobSpawnerSwamp extends MobSpawnerBase`
- **Rol**: Definición ecológica para el bioma de Pantano.
- **Descripción Técnica**: Configura las criaturas que pueden aparecer en ciénagas y humedales.
- **Guía de Modificación**: Ejemplo de especialización bioclimática.

##### <a id="class-path"></a>[`Path`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Path.java)
- **Ruta y Paquete**: `net.minecraft.src.Path` (`net/minecraft/src/Path.java`) | **Líneas de código**: 144
- **Declaración**: `class Path`
- **Rol**: Encapsulador de una ruta de navegación calculada para criaturas.
- **Descripción Técnica**: Almacena la secuencia de nodos PathPoint que un mob debe recorrer sucesivamente para alcanzar su destino.
- **Guía de Modificación**: Utilizado por Pathfinder y EntityCreature.
- **Métodos Clave Destacados**:
  - `PathPoint addPoint(PathPoint pathpoint)`
  - `void clearPath()`
  - `PathPoint dequeue()`
  - `void changeDistance(PathPoint pathpoint, float f)`
  - `boolean isPathEmpty()`

##### <a id="class-pathentity"></a>[`PathEntity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PathEntity.java)
- **Ruta y Paquete**: `net.minecraft.src.PathEntity` (`net/minecraft/src/PathEntity.java`) | **Líneas de código**: 37
- **Declaración**: `class PathEntity`
- **Rol**: Secuencia ordenada de puntos de ruta (PathPoint) asignada a una entidad.
- **Descripción Técnica**: Permite que un mob avance nodo a nodo comprobando si ha alcanzado el siguiente punto del camino.
- **Guía de Modificación**: Utilizado en la navegación cinemática de criaturas vivas.
- **Métodos Clave Destacados**:
  - `void incrementPathIndex()`
  - `boolean isFinished()`
  - `Vec3D getPosition(Entity entity)`
- **Campos Clave Destacados**:
  - `int pathLength`

##### <a id="class-pathpoint"></a>[`PathPoint`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PathPoint.java)
- **Ruta y Paquete**: `net.minecraft.src.PathPoint` (`net/minecraft/src/PathPoint.java`) | **Líneas de código**: 58
- **Declaración**: `class PathPoint`
- **Rol**: Nodo espacial tridimensional con coste de recorrido para el algoritmo A*.
- **Descripción Técnica**: Almacena coordenadas enteras (x, y, z), índice en el heap binario, coste acumulado g y distancia estimada h al objetivo.
- **Guía de Modificación**: El elemento básico de la búsqueda de caminos de Pathfinder.
- **Métodos Clave Destacados**:
  - `float distanceTo(PathPoint pathpoint)`
  - `boolean equals(Object obj)`
  - `int hashCode()`
  - `boolean isAssigned()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `int xCoord`
  - `int yCoord`
  - `int zCoord`
  - `int hash`
  - `boolean isFirst`

##### <a id="class-pathfinder"></a>[`Pathfinder`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Pathfinder.java)
- **Ruta y Paquete**: `net.minecraft.src.Pathfinder` (`net/minecraft/src/Pathfinder.java`) | **Líneas de código**: 218
- **Declaración**: `class Pathfinder`
- **Rol**: Motor de búsqueda de caminos sobre la malla de bloques navegables mediante algoritmo A*.
- **Descripción Técnica**: Calcula la ruta óptima sorteando obstáculos, caídas y desniveles en el terreno para que los mobs alcancen al jugador.
- **Guía de Modificación**: Fundamental para optimizar la inteligencia artificial o permitir que los mobs superen obstáculos complejos.
- **Métodos Clave Destacados**:
  - `PathEntity createEntityPathTo(Entity entity, Entity entity1, float f)`
  - `PathEntity createEntityPathTo(Entity entity, int i, int j, int k, float f)`

##### <a id="class-spawneranimals"></a>[`SpawnerAnimals`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SpawnerAnimals.java)
- **Ruta y Paquete**: `net.minecraft.src.SpawnerAnimals` (`net/minecraft/src/SpawnerAnimals.java`) | **Líneas de código**: 125
- **Declaración**: `class SpawnerAnimals`
- **Rol**: Motor biológico de spawn periódico de criaturas.
- **Descripción Técnica**: Evalúa chunks en un radio alrededor del jugador en cada tick, comprueba el límite de población de bioma e instancia nuevos animales o monstruos.
- **Guía de Modificación**: Punto clave para modificar la tasa de aparición o densidad ecológica del mundo.
- **Métodos Clave Destacados**:
  - `ChunkPosition getRandomSpawningPointInChunk(World world, int i, int j)`
  - `int performSpawning(World var0)`


### 05. Generación de Mundo y Biomas (World Generation & Biomes)
> **Resumen del subsistema**: Algoritmos procedurales de generación de terreno mediante ruido Perlin 3D. Abarca el tallado de cuevas gusano (MapGenCaves), distribución de biomas (temperatura y humedad), y generadores de árboles, minerales, lagos, mazmorras y el Inframundo (Nether).

**Total de clases**: 19

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`MapGenBase`](#class-mapgenbase) | `class` | `-` | 41 | Clase base para generadores de estructuras geológicas del mapa. |
| [`MapGenCaves`](#class-mapgencaves) | `class` | `ext MapGenBase` | 224 | Algoritmo procedural de tallado de redes de cuevas subterráneas (... |
| [`MapGenCavesHell`](#class-mapgencaveshell) | `class` | `ext MapGenBase` | 209 | Generador de cavernas y bolsas de lava específico del Inframundo ... |
| [`WorldGenBigTree`](#class-worldgenbigtree) | `class` | `ext WorldGenerator` | 450 | Generador de estructuras o recursos: 'WorldGenBigTree'. |
| [`WorldGenCactus`](#class-worldgencactus) | `class` | `ext WorldGenerator` | 39 | Generador superficial de columnas de cactus en biomas desérticos. |
| [`WorldGenClay`](#class-worldgenclay) | `class` | `ext WorldGenerator` | 69 | Generador de depósitos submarinos de arcilla. |
| [`WorldGenDungeons`](#class-worldgendungeons) | `class` | `ext WorldGenerator` | 214 | Generador de estructuras o recursos: 'WorldGenDungeons'. |
| [`WorldGenFire`](#class-worldgenfire) | `class` | `ext WorldGenerator` | 30 | Generador de fuegos eternos en el Inframundo (Nether). |
| [`WorldGenFlowers`](#class-worldgenflowers) | `class` | `ext WorldGenerator` | 33 | Generador de parches de flores amarillas y rosas rojas. |
| [`WorldGenHellLava`](#class-worldgenhelllava) | `class` | `ext WorldGenerator` | 79 | Generador de manantiales y cascadas de lava en los muros del Neth... |
| [`WorldGenLakes`](#class-worldgenlakes) | `class` | `ext WorldGenerator` | 115 | Generador de lagos superficiales y subterráneos de agua o lava. |
| [`WorldGenLightStone1`](#class-worldgenlightstone1) | `class` | `ext WorldGenerator` | 77 | Generador de estalactitas colgantes de piedra luminosa (Glowstone... |
| [`WorldGenLightStone2`](#class-worldgenlightstone2) | `class` | `ext WorldGenerator` | 77 | Segunda variante del generador de Glowstone en paredes y recoveco... |
| [`WorldGenLiquids`](#class-worldgenliquids) | `class` | `ext WorldGenerator` | 75 | Generador de pequeños manantiales de agua o lava que brotan de la... |
| [`WorldGenMinable`](#class-worldgenminable) | `class` | `ext WorldGenerator` | 60 | Generador de estructuras o recursos: 'WorldGenMinable'. |
| [`WorldGenPumpkin`](#class-worldgenpumpkin) | `class` | `ext WorldGenerator` | 30 | Generador de calabazas silvestres en la superficie del Overworld. |
| [`WorldGenReed`](#class-worldgenreed) | `class` | `ext WorldGenerator` | 39 | Generador de cañas de azúcar en orillas de ríos y costas. |
| [`WorldGenTrees`](#class-worldgentrees) | `class` | `ext WorldGenerator` | 96 | Generador de estructuras o recursos: 'WorldGenTrees'. |
| [`WorldGenerator`](#class-worldgenerator) | `class` | `-` | 20 | Generador de estructuras o recursos: 'WorldGenerator'. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-mapgenbase"></a>[`MapGenBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MapGenBase.java)
- **Ruta y Paquete**: `net.minecraft.src.MapGenBase` (`net/minecraft/src/MapGenBase.java`) | **Líneas de código**: 41
- **Declaración**: `class MapGenBase`
- **Rol**: Clase base para generadores de estructuras geológicas del mapa.
- **Descripción Técnica**: Procesa el terreno chunk a chunk aplicando transformaciones continuas (ej. tallado de cuevas).
- **Guía de Modificación**: Extender para crear generadores de barrancos, fallas o túneles subterráneos.
- **Métodos Clave Destacados**:
  - `void func_867_a(IChunkProvider ichunkprovider, World world, int i, int j, byte abyte0[])`
  - `void func_868_a(World world, int i, int j, int k, int l, byte abyte0[])`
- **Campos Clave Destacados**:
  - `int field_1306_a`
  - `Random field_1305_b`

##### <a id="class-mapgencaves"></a>[`MapGenCaves`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MapGenCaves.java)
- **Ruta y Paquete**: `net.minecraft.src.MapGenCaves` (`net/minecraft/src/MapGenCaves.java`) | **Líneas de código**: 224
- **Declaración**: `class MapGenCaves extends MapGenBase`
- **Rol**: Algoritmo procedural de tallado de redes de cuevas subterráneas (worm caves).
- **Descripción Técnica**: Genera túneles sinuosos que serpentean a través de la piedra creando cavernas, intersecciones y pozos usando ruido y trigonometría.
- **Guía de Modificación**: Ajusta la densidad y amplitud de las cuevas para crear mundos más o menos cavernosos.
- **Métodos Clave Destacados**:
  - `void func_870_a(int i, int j, byte abyte0[], double d, double d1, double d2)`
  - `void func_869_a(int i, int j, byte abyte0[], double d, double d1, double d2, float f, float f1, float f2, int k, int l, double d3)`
  - `void func_868_a(World world, int i, int j, int k, int l, byte abyte0[])`

##### <a id="class-mapgencaveshell"></a>[`MapGenCavesHell`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MapGenCavesHell.java)
- **Ruta y Paquete**: `net.minecraft.src.MapGenCavesHell` (`net/minecraft/src/MapGenCavesHell.java`) | **Líneas de código**: 209
- **Declaración**: `class MapGenCavesHell extends MapGenBase`
- **Rol**: Generador de cavernas y bolsas de lava específico del Inframundo (Nether).
- **Descripción Técnica**: Talla cavidades redondeadas a gran escala dentro de la masa sólida de Netherrack.
- **Guía de Modificación**: Modificar para alterar la morfología de las cavernas del Nether.
- **Métodos Clave Destacados**:
  - `void func_4129_a(int i, int j, byte abyte0[], double d, double d1, double d2)`
  - `void func_4128_a(int i, int j, byte abyte0[], double d, double d1, double d2, float f, float f1, float f2, int k, int l, double d3)`
  - `void func_868_a(World world, int i, int j, int k, int l, byte abyte0[])`

##### <a id="class-worldgenbigtree"></a>[`WorldGenBigTree`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenBigTree.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenBigTree` (`net/minecraft/src/WorldGenBigTree.java`) | **Líneas de código**: 450
- **Declaración**: `class WorldGenBigTree extends WorldGenerator`
- **Rol**: Generador de estructuras o recursos: 'WorldGenBigTree'.
- **Descripción Técnica**: Inserta proceduralmente bloques en el terreno durante la fase de población de chunks para 'WorldGenBigTree'.
- **Guía de Modificación**: Modifica las condiciones de colocación o el tipo de bloques generados.
- **Métodos Clave Destacados**:
  - `void func_517_a(double d, double d1, double d2)`
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgencactus"></a>[`WorldGenCactus`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenCactus.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenCactus` (`net/minecraft/src/WorldGenCactus.java`) | **Líneas de código**: 39
- **Declaración**: `class WorldGenCactus extends WorldGenerator`
- **Rol**: Generador superficial de columnas de cactus en biomas desérticos.
- **Descripción Técnica**: Coloca cactus de 1 a 3 bloques de altura sobre bloques de arena que no tengan vecinos sólidos.
- **Guía de Modificación**: Ajustar la frecuencia o altura de los cactus en desiertos.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenclay"></a>[`WorldGenClay`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenClay.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenClay` (`net/minecraft/src/WorldGenClay.java`) | **Líneas de código**: 69
- **Declaración**: `class WorldGenClay extends WorldGenerator`
- **Rol**: Generador de depósitos submarinos de arcilla.
- **Descripción Técnica**: Genera vetas de arcilla en el lecho de lagos y playas poco profundas.
- **Guía de Modificación**: Modificar para aumentar la disponibilidad de arcilla en el juego.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgendungeons"></a>[`WorldGenDungeons`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenDungeons.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenDungeons` (`net/minecraft/src/WorldGenDungeons.java`) | **Líneas de código**: 214
- **Declaración**: `class WorldGenDungeons extends WorldGenerator`
- **Rol**: Generador de estructuras o recursos: 'WorldGenDungeons'.
- **Descripción Técnica**: Inserta proceduralmente bloques en el terreno durante la fase de población de chunks para 'WorldGenDungeons'.
- **Guía de Modificación**: Modifica las condiciones de colocación o el tipo de bloques generados.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenfire"></a>[`WorldGenFire`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenFire.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenFire` (`net/minecraft/src/WorldGenFire.java`) | **Líneas de código**: 30
- **Declaración**: `class WorldGenFire extends WorldGenerator`
- **Rol**: Generador de fuegos eternos en el Inframundo (Nether).
- **Descripción Técnica**: Coloca parches de fuego sobre superficies de Netherrack durante la generación del terreno.
- **Guía de Modificación**: Controla la ambientación pirotécnica del Nether.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenflowers"></a>[`WorldGenFlowers`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenFlowers.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenFlowers` (`net/minecraft/src/WorldGenFlowers.java`) | **Líneas de código**: 33
- **Declaración**: `class WorldGenFlowers extends WorldGenerator`
- **Rol**: Generador de parches de flores amarillas y rosas rojas.
- **Descripción Técnica**: Genera agrupaciones de flores sobre bloques de hierba con exposición a la luz solar.
- **Guía de Modificación**: Modificar para introducir nuevas especies botánicas en el paisaje.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenhelllava"></a>[`WorldGenHellLava`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenHellLava.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenHellLava` (`net/minecraft/src/WorldGenHellLava.java`) | **Líneas de código**: 79
- **Declaración**: `class WorldGenHellLava extends WorldGenerator`
- **Rol**: Generador de manantiales y cascadas de lava en los muros del Nether.
- **Descripción Técnica**: Perfora cavidades en los acantilados de Netherrack colocando bloques de lava que caen en cascada hacia el suelo.
- **Guía de Modificación**: Crea el paisaje característico de cataratas de lava del Inframundo.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenlakes"></a>[`WorldGenLakes`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenLakes.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenLakes` (`net/minecraft/src/WorldGenLakes.java`) | **Líneas de código**: 115
- **Declaración**: `class WorldGenLakes extends WorldGenerator`
- **Rol**: Generador de lagos superficiales y subterráneos de agua o lava.
- **Descripción Técnica**: Talla una cuenca redondeada en el terreno y la llena con el líquido seleccionado.
- **Guía de Modificación**: Ajustar la probabilidad o líquido de los lagos generados.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenlightstone1"></a>[`WorldGenLightStone1`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenLightStone1.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenLightStone1` (`net/minecraft/src/WorldGenLightStone1.java`) | **Líneas de código**: 77
- **Declaración**: `class WorldGenLightStone1 extends WorldGenerator`
- **Rol**: Generador de estalactitas colgantes de piedra luminosa (Glowstone) en el techo del Nether.
- **Descripción Técnica**: Genera racimos suspendidos de Glowstone que iluminan las cavernas superiores del Inframundo.
- **Guía de Modificación**: Modificar para alterar la forma o frecuencia de la Glowstone en el Nether.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenlightstone2"></a>[`WorldGenLightStone2`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenLightStone2.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenLightStone2` (`net/minecraft/src/WorldGenLightStone2.java`) | **Líneas de código**: 77
- **Declaración**: `class WorldGenLightStone2 extends WorldGenerator`
- **Rol**: Segunda variante del generador de Glowstone en paredes y recovecos del Nether.
- **Descripción Técnica**: Produce acumulaciones de piedra brillante incrustadas en salientes rocosos.
- **Guía de Modificación**: Complementa WorldGenLightStone1.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenliquids"></a>[`WorldGenLiquids`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenLiquids.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenLiquids` (`net/minecraft/src/WorldGenLiquids.java`) | **Líneas de código**: 75
- **Declaración**: `class WorldGenLiquids extends WorldGenerator`
- **Rol**: Generador de pequeños manantiales de agua o lava que brotan de las paredes de piedra.
- **Descripción Técnica**: Perfora un bloque en una pared rocosa y coloca una fuente que fluye hacia el exterior.
- **Guía de Modificación**: Crea manantiales naturales dentro de cuevas.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenminable"></a>[`WorldGenMinable`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenMinable.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenMinable` (`net/minecraft/src/WorldGenMinable.java`) | **Líneas de código**: 60
- **Declaración**: `class WorldGenMinable extends WorldGenerator`
- **Rol**: Generador de estructuras o recursos: 'WorldGenMinable'.
- **Descripción Técnica**: Inserta proceduralmente bloques en el terreno durante la fase de población de chunks para 'WorldGenMinable'.
- **Guía de Modificación**: Modifica las condiciones de colocación o el tipo de bloques generados.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenpumpkin"></a>[`WorldGenPumpkin`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenPumpkin.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenPumpkin` (`net/minecraft/src/WorldGenPumpkin.java`) | **Líneas de código**: 30
- **Declaración**: `class WorldGenPumpkin extends WorldGenerator`
- **Rol**: Generador de calabazas silvestres en la superficie del Overworld.
- **Descripción Técnica**: Genera grupos dispersos de calabazas talladas en zonas de hierba.
- **Guía de Modificación**: Modificar para variar la rareza de las calabazas.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenreed"></a>[`WorldGenReed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenReed.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenReed` (`net/minecraft/src/WorldGenReed.java`) | **Líneas de código**: 39
- **Declaración**: `class WorldGenReed extends WorldGenerator`
- **Rol**: Generador de cañas de azúcar en orillas de ríos y costas.
- **Descripción Técnica**: Coloca cañas de azúcar de 1 a 3 bloques de altura sobre tierra o arena adyacente a un bloque de agua.
- **Guía de Modificación**: Ajustar para cambiar la frecuencia de cañas en el mundo.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgentrees"></a>[`WorldGenTrees`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenTrees.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenTrees` (`net/minecraft/src/WorldGenTrees.java`) | **Líneas de código**: 96
- **Declaración**: `class WorldGenTrees extends WorldGenerator`
- **Rol**: Generador de estructuras o recursos: 'WorldGenTrees'.
- **Descripción Técnica**: Inserta proceduralmente bloques en el terreno durante la fase de población de chunks para 'WorldGenTrees'.
- **Guía de Modificación**: Modifica las condiciones de colocación o el tipo de bloques generados.
- **Métodos Clave Destacados**:
  - `boolean generate(World world, Random random, int i, int j, int k)`

##### <a id="class-worldgenerator"></a>[`WorldGenerator`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldGenerator.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldGenerator` (`net/minecraft/src/WorldGenerator.java`) | **Líneas de código**: 20
- **Declaración**: `class WorldGenerator`
- **Rol**: Generador de estructuras o recursos: 'WorldGenerator'.
- **Descripción Técnica**: Inserta proceduralmente bloques en el terreno durante la fase de población de chunks para 'WorldGenerator'.
- **Guía de Modificación**: Modifica las condiciones de colocación o el tipo de bloques generados.
- **Métodos Clave Destacados**:
  - `void func_517_a(double d, double d1, double d2)`


### 06. Mundo, Chunks, Iluminación y Guardado (World, Chunks, Lighting & Storage)
> **Resumen del subsistema**: Estructura espacial y de persistencia. El mundo dividido en Chunks de 16x16x128 bloques, arrays de bloques y metadatos de 4 bits (NibbleArray), cálculo y propagación de iluminación solar y de bloques, cola de ticks programados y serialización en disco.

**Total de clases**: 30

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`Chunk`](#class-chunk) | `class` | `-` | 691 | Segmento vertical del terreno de 16x16x128 bloques. |
| [`ChunkCache`](#class-chunkcache) | `class` | `impl IBlockAccess
` | 169 | Caché ligera de chunks adyacentes para operaciones rápidas de ren... |
| [`ChunkCoordIntPair`](#class-chunkcoordintpair) | `class` | `-` | 29 | Par inmutable de coordenadas enteras (chunkX, chunkZ) para indexa... |
| [`ChunkCoordinates`](#class-chunkcoordinates) | `class` | `-` | 35 | Tupla de coordenadas espaciales tridimensionales de bloque (x, y,... |
| [`ChunkLoader`](#class-chunkloader) | `class` | `impl IChunkLoader
` | 243 | Implementación de almacenamiento de chunks en disco en formato cl... |
| [`ChunkPosition`](#class-chunkposition) | `class` | `-` | 37 | Contenedor ligero de posición tridimensional entera (x, y, z). |
| [`ChunkProviderClient`](#class-chunkproviderclient) | `class` | `impl IChunkProvider
` | 86 | Proveedor de chunks ficticio en el cliente multijugador. |
| [`ChunkProviderGenerate`](#class-chunkprovidergenerate) | `class` | `impl IChunkProvider
` | 609 | Generador procedural principal de terreno para el Overworld. |
| [`ChunkProviderHell`](#class-chunkproviderhell) | `class` | `impl IChunkProvider
` | 419 | Generador procedural del Inframundo (Nether). |
| [`ChunkProviderIso`](#class-chunkprovideriso) | `class` | `impl IChunkProvider
` | 87 | Proveedor de chunks optimizado para la vista isométrica del mapa. |
| [`ChunkProviderLoadOrGenerate`](#class-chunkproviderloadorgenerate) | `class` | `impl IChunkProvider
` | 240 | Orquestador de carga y generación de chunks para un jugador y ser... |
| [`EnumSkyBlock`](#class-enumskyblock) | `enum` | `-` | 43 | Diferenciación de tipos de iluminación (Sky = Solar, Block = Bloq... |
| [`Explosion`](#class-explosion) | `class` | `-` | 186 | Física de detonaciones con destrucción de bloques y vectores de e... |
| [`IBlockAccess`](#class-iblockaccess) | `interface` | `-` | 23 | Interfaz de solo lectura para consulta de bloques, metadatos y lu... |
| [`IChunkLoader`](#class-ichunkloader) | `interface` | `-` | 20 | Interfaz para la persistencia de chunks en almacenamiento. |
| [`IChunkProvider`](#class-ichunkprovider) | `interface` | `-` | 21 | Interfaz unificada para proveedores de chunks del mundo. |
| [`IWorldAccess`](#class-iworldaccess) | `interface` | `-` | 29 | Interfaz de escucha para eventos que alteran el entorno del mundo... |
| [`MetadataChunkBlock`](#class-metadatachunkblock) | `class` | `-` | 202 | Estructura auxiliar que empaqueta cambios de bloques en chunks pa... |
| [`NextTickListEntry`](#class-nextticklistentry) | `class` | `impl Comparable
` | 73 | Entrada individual en la cola de ticks programados de World. |
| [`NibbleArray`](#class-nibblearray) | `class` | `-` | 54 | Array de memoria comprimida que almacena 2 valores de 4 bits en c... |
| [`Teleporter`](#class-teleporter) | `class` | `-` | 302 | Gestor cinemático de portales y translación de coordenadas entre ... |
| [`World`](#class-world) | `class` | `impl IBlockAccess
` | 2448 | El gestor integral del estado del mundo, físicas, iluminación y e... |
| [`WorldBlockPositionType`](#class-worldblockpositiontype) | `class` | `-` | 28 | Tupla que asocia una posición tridimensional con el tipo de bloqu... |
| [`WorldChunkManager`](#class-worldchunkmanager) | `class` | `-` | 134 | Gestor climático y de biomas para el mundo Overworld. |
| [`WorldChunkManagerHell`](#class-worldchunkmanagerhell) | `class` | `ext WorldChunkManager` | 66 | Gestor de bioma único para el Inframundo (Nether). |
| [`WorldClient`](#class-worldclient) | `class` | `ext World` | 262 | Implementación de World especializada para el cliente multijugado... |
| [`WorldIso`](#class-worldiso) | `class` | `ext World` | 23 | Mundo adaptado para la representación isométrica en el visor de m... |
| [`WorldProvider`](#class-worldprovider) | `class` | `-` | 147 | Abstracción ambiental de dimensiones (Overworld vs Nether). |
| [`WorldProviderHell`](#class-worldproviderhell) | `class` | `ext WorldProvider` | 75 | Implementación ambiental específica del Inframundo (Nether). |
| [`WorldRenderer`](#class-worldrenderer) | `class` | `-` | 282 | Compilador de fragmentos de terreno en listas de visualización Op... |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-chunk"></a>[`Chunk`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Chunk.java)
- **Ruta y Paquete**: `net.minecraft.src.Chunk` (`net/minecraft/src/Chunk.java`) | **Líneas de código**: 691
- **Declaración**: `class Chunk`
- **Rol**: Segmento vertical del terreno de 16x16x128 bloques.
- **Descripción Técnica**: Almacena arrays planos para IDs de bloques (32768 bytes), metadatos (NibbleArray), luz solar y luz de bloque. Gestiona el mapa de alturas (heightMap) y entidades del área.
- **Guía de Modificación**: Optimiza o inspecciona esta clase para manipulaciones masivas de terreno de alto rendimiento.
- **Métodos Clave Destacados**:
  - `boolean isAtLocation(int i, int j)`
  - `int getHeightValue(int i, int j)`
  - `void func_1014_a()`
  - `void generateHeightMap()`
  - `void func_1024_c()`
  - `void func_4143_d()`
- **Campos Clave Destacados**:
  - `boolean field_1540_a`
  - `boolean isChunkLoaded`
  - `World worldObj`
  - `NibbleArray data`
  - `NibbleArray skylightMap`
  - `NibbleArray blocklightMap`

##### <a id="class-chunkcache"></a>[`ChunkCache`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkCache.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkCache` (`net/minecraft/src/ChunkCache.java`) | **Líneas de código**: 169
- **Declaración**: `class ChunkCache implements IBlockAccess
`
- **Rol**: Caché ligera de chunks adyacentes para operaciones rápidas de renderizado.
- **Descripción Técnica**: Implementa IBlockAccess sobre un rango acotado de chunks, evitando sincronizaciones pesadas con World.
- **Guía de Modificación**: Utilizada intensamente por RenderBlocks para consultar iluminación y vecinos sin penalización.
- **Métodos Clave Destacados**:
  - `int getBlockId(int i, int j, int k)`
  - `TileEntity getBlockTileEntity(int i, int j, int k)`
  - `float getLightBrightness(int i, int j, int k)`
  - `int func_4086_d(int i, int j, int k)`
  - `int func_716_a(int i, int j, int k, boolean flag)`
  - `int getBlockMetadata(int i, int j, int k)`

##### <a id="class-chunkcoordintpair"></a>[`ChunkCoordIntPair`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkCoordIntPair.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkCoordIntPair` (`net/minecraft/src/ChunkCoordIntPair.java`) | **Líneas de código**: 29
- **Declaración**: `class ChunkCoordIntPair`
- **Rol**: Par inmutable de coordenadas enteras (chunkX, chunkZ) para indexación de chunks.
- **Descripción Técnica**: Implementa hashCode() y equals() para servir como clave en tablas hash de chunks cargados.
- **Guía de Modificación**: Clase fundamental para referenciar chunks en mapas y conjuntos.
- **Métodos Clave Destacados**:
  - `int hashCode()`
  - `boolean equals(Object obj)`
- **Campos Clave Destacados**:
  - `int chunkXPos`
  - `int chunkZPos`

##### <a id="class-chunkcoordinates"></a>[`ChunkCoordinates`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkCoordinates.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkCoordinates` (`net/minecraft/src/ChunkCoordinates.java`) | **Líneas de código**: 35
- **Declaración**: `class ChunkCoordinates`
- **Rol**: Tupla de coordenadas espaciales tridimensionales de bloque (x, y, z).
- **Descripción Técnica**: Almacena puntos en el espacio para puntos de spawn, camas o marcadores de brújula.
- **Guía de Modificación**: Útil para guardar y comparar posiciones tridimensionales.
- **Métodos Clave Destacados**:
  - `boolean equals(Object obj)`
  - `int hashCode()`
- **Campos Clave Destacados**:
  - `int field_1518_a`
  - `int field_1517_b`

##### <a id="class-chunkloader"></a>[`ChunkLoader`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkLoader.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkLoader` (`net/minecraft/src/ChunkLoader.java`) | **Líneas de código**: 243
- **Declaración**: `class ChunkLoader implements IChunkLoader
`
- **Rol**: Implementación de almacenamiento de chunks en disco en formato clásico de Alpha.
- **Descripción Técnica**: Serializa cada chunk en archivos binarios individuales organizados en subdirectorios calculados por base 64.
- **Guía de Modificación**: Relevante si se diseñan conversores de mapas o guardado alternativo.
- **Métodos Clave Destacados**:
  - `Chunk loadChunk(World world, int i, int j)`
  - `void saveChunk(World world, Chunk chunk)`
  - `void storeChunkInCompound(Chunk chunk, World world, NBTTagCompound nbttagcompound)`
  - `Chunk loadChunkIntoWorldFromCompound(World world, NBTTagCompound nbttagcompound)`
  - `void func_814_a()`
  - `void saveExtraData()`

##### <a id="class-chunkposition"></a>[`ChunkPosition`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkPosition.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkPosition` (`net/minecraft/src/ChunkPosition.java`) | **Líneas de código**: 37
- **Declaración**: `class ChunkPosition`
- **Rol**: Contenedor ligero de posición tridimensional entera (x, y, z).
- **Descripción Técnica**: Utilizado en cálculos de propagación de explosiones y algoritmos de búsqueda de caminos.
- **Guía de Modificación**: Útil como estructura ligera de coordenadas inmutables.
- **Métodos Clave Destacados**:
  - `boolean equals(Object obj)`
  - `int hashCode()`
- **Campos Clave Destacados**:
  - `int x`
  - `int y`
  - `int z`

##### <a id="class-chunkproviderclient"></a>[`ChunkProviderClient`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderClient.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkProviderClient` (`net/minecraft/src/ChunkProviderClient.java`) | **Líneas de código**: 86
- **Declaración**: `class ChunkProviderClient implements IChunkProvider
`
- **Rol**: Proveedor de chunks ficticio en el cliente multijugador.
- **Descripción Técnica**: Almacena en memoria únicamente los chunks recibidos desde el servidor a través de paquetes de red.
- **Guía de Modificación**: Punto clave para gestionar los chunks visibles en el cliente multijugador.
- **Métodos Clave Destacados**:
  - `boolean chunkExists(int i, int j)`
  - `void func_539_c(int i, int j)`
  - `Chunk func_538_d(int i, int j)`
  - `Chunk provideChunk(int i, int j)`
  - `boolean saveChunks(boolean flag, IProgressUpdate iprogressupdate)`
  - `boolean func_532_a()`

##### <a id="class-chunkprovidergenerate"></a>[`ChunkProviderGenerate`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderGenerate.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkProviderGenerate` (`net/minecraft/src/ChunkProviderGenerate.java`) | **Líneas de código**: 609
- **Declaración**: `class ChunkProviderGenerate implements IChunkProvider
`
- **Rol**: Generador procedural principal de terreno para el Overworld.
- **Descripción Técnica**: Calcula la topografía mediante matrices de ruido Perlin 3D, genera capas de piedra, tierra, agua, y llama a 'populate' para árboles y minerales.
- **Guía de Modificación**: El punto central para modificar la forma de las montañas, islas flotantes o continentes del mundo.
- **Métodos Clave Destacados**:
  - `void generateTerrain(int i, int j, byte abyte0[], MobSpawnerBase amobspawnerbase[], double ad[])`
  - `void replaceBlocksForBiome(int i, int j, byte abyte0[], MobSpawnerBase amobspawnerbase[])`
  - `Chunk provideChunk(int i, int j)`
  - `boolean chunkExists(int i, int j)`
  - `void populate(IChunkProvider ichunkprovider, int i, int j)`
  - `boolean saveChunks(boolean flag, IProgressUpdate iprogressupdate)`
- **Campos Clave Destacados**:
  - `NoiseGeneratorOctaves field_922_a`
  - `NoiseGeneratorOctaves field_921_b`
  - `NoiseGeneratorOctaves field_920_c`

##### <a id="class-chunkproviderhell"></a>[`ChunkProviderHell`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderHell.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkProviderHell` (`net/minecraft/src/ChunkProviderHell.java`) | **Líneas de código**: 419
- **Declaración**: `class ChunkProviderHell implements IChunkProvider
`
- **Rol**: Generador procedural del Inframundo (Nether).
- **Descripción Técnica**: Genera una caverna infinita con techo y suelo de bedrock, lagos masivos de lava a Y=32, y pilares de netherrack.
- **Guía de Modificación**: Modificar para alterar la generación geológica del Nether o crear dimensiones subterráneas.
- **Métodos Clave Destacados**:
  - `void func_4059_a(int i, int j, byte abyte0[])`
  - `void func_4058_b(int i, int j, byte abyte0[])`
  - `Chunk provideChunk(int i, int j)`
  - `boolean chunkExists(int i, int j)`
  - `void populate(IChunkProvider ichunkprovider, int i, int j)`
  - `boolean saveChunks(boolean flag, IProgressUpdate iprogressupdate)`
- **Campos Clave Destacados**:
  - `NoiseGeneratorOctaves field_4177_a`
  - `NoiseGeneratorOctaves field_4176_b`

##### <a id="class-chunkprovideriso"></a>[`ChunkProviderIso`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderIso.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkProviderIso` (`net/minecraft/src/ChunkProviderIso.java`) | **Líneas de código**: 87
- **Declaración**: `class ChunkProviderIso implements IChunkProvider
`
- **Rol**: Proveedor de chunks optimizado para la vista isométrica del mapa.
- **Descripción Técnica**: Carga únicamente los chunks necesarios para la proyección actual.
- **Guía de Modificación**: Mejora la eficiencia al explorar mapas de gran escala.
- **Métodos Clave Destacados**:
  - `boolean chunkExists(int i, int j)`
  - `Chunk provideChunk(int i, int j)`
  - `void populate(IChunkProvider ichunkprovider, int i, int j)`
  - `boolean saveChunks(boolean flag, IProgressUpdate iprogressupdate)`
  - `boolean func_532_a()`
  - `boolean func_536_b()`

##### <a id="class-chunkproviderloadorgenerate"></a>[`ChunkProviderLoadOrGenerate`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChunkProviderLoadOrGenerate.java)
- **Ruta y Paquete**: `net.minecraft.src.ChunkProviderLoadOrGenerate` (`net/minecraft/src/ChunkProviderLoadOrGenerate.java`) | **Líneas de código**: 240
- **Declaración**: `class ChunkProviderLoadOrGenerate implements IChunkProvider
`
- **Rol**: Orquestador de carga y generación de chunks para un jugador y servidores.
- **Descripción Técnica**: Si el chunk existe en disco, lo carga con ChunkLoader; si no, lo genera con ChunkProviderGenerate y lo guarda.
- **Guía de Modificación**: Controla la persistencia de terreno y la descarga de chunks inactivos de la memoria RAM.
- **Métodos Clave Destacados**:
  - `boolean chunkExists(int i, int j)`
  - `Chunk provideChunk(int i, int j)`
  - `void populate(IChunkProvider ichunkprovider, int i, int j)`
  - `boolean saveChunks(boolean flag, IProgressUpdate iprogressupdate)`
  - `boolean func_532_a()`
  - `boolean func_536_b()`

##### <a id="class-enumskyblock"></a>[`EnumSkyBlock`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EnumSkyBlock.java)
- **Ruta y Paquete**: `net.minecraft.src.EnumSkyBlock` (`net/minecraft/src/EnumSkyBlock.java`) | **Líneas de código**: 43
- **Declaración**: `enum EnumSkyBlock`
- **Rol**: Diferenciación de tipos de iluminación (Sky = Solar, Block = Bloques luminosos).
- **Descripción Técnica**: Permite que el motor gestione dos canales de luz independientes con valores de 0 a 15 por cada bloque.
- **Guía de Modificación**: Esencial en el algoritmo de cálculo de sombras e iluminación dinámica.
- **Métodos Clave Destacados**:
  - `EnumSkyBlock[] func_4151_values()`
  - `EnumSkyBlock valueOf(String s)`
- **Campos Clave Destacados**:
  - `EnumSkyBlock Sky`
  - `EnumSkyBlock Block`
  - `int field_1722_c`

##### <a id="class-explosion"></a>[`Explosion`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Explosion.java)
- **Ruta y Paquete**: `net.minecraft.src.Explosion` (`net/minecraft/src/Explosion.java`) | **Líneas de código**: 186
- **Declaración**: `class Explosion`
- **Rol**: Física de detonaciones con destrucción de bloques y vectores de empuje.
- **Descripción Técnica**: Calcula el radio de daño mediante rayos vectoriales tridimensionales, destruye bloques según su resistencia e impulsa y daña entidades.
- **Guía de Modificación**: Ajusta la atenuación o añade efectos secundarios (fuego, radiación, congelación).
- **Métodos Clave Destacados**:
  - `void func_12248_a()`
  - `void func_12247_b()`
- **Campos Clave Destacados**:
  - `boolean field_12257_a`
  - `double field_12256_b`
  - `double field_12255_c`
  - `double field_12254_d`
  - `Entity field_12253_e`
  - `float field_12252_f`

##### <a id="class-iblockaccess"></a>[`IBlockAccess`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IBlockAccess.java)
- **Ruta y Paquete**: `net.minecraft.src.IBlockAccess` (`net/minecraft/src/IBlockAccess.java`) | **Líneas de código**: 23
- **Declaración**: `interface IBlockAccess`
- **Rol**: Interfaz de solo lectura para consulta de bloques, metadatos y luz.
- **Descripción Técnica**: Proporciona acceso a getBlockId, getBlockMetadata y getLightValue sin permitir modificaciones sobre el mundo.
- **Guía de Modificación**: Permite que clases de renderizado lean el entorno de forma segura.

##### <a id="class-ichunkloader"></a>[`IChunkLoader`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IChunkLoader.java)
- **Ruta y Paquete**: `net.minecraft.src.IChunkLoader` (`net/minecraft/src/IChunkLoader.java`) | **Líneas de código**: 20
- **Declaración**: `interface IChunkLoader`
- **Rol**: Interfaz para la persistencia de chunks en almacenamiento.
- **Descripción Técnica**: Declara métodos de carga, guardado y descarga de datos de chunks.
- **Guía de Modificación**: Implementar para crear sistemas de guardado en base de datos o formatos de región.

##### <a id="class-ichunkprovider"></a>[`IChunkProvider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IChunkProvider.java)
- **Ruta y Paquete**: `net.minecraft.src.IChunkProvider` (`net/minecraft/src/IChunkProvider.java`) | **Líneas de código**: 21
- **Declaración**: `interface IChunkProvider`
- **Rol**: Interfaz unificada para proveedores de chunks del mundo.
- **Descripción Técnica**: Declara métodos: loadChunk, provideChunk, populate, saveChunks y unload100OldestChunks.
- **Guía de Modificación**: Implementar para crear generadores de mundos completamente personalizados.

##### <a id="class-iworldaccess"></a>[`IWorldAccess`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IWorldAccess.java)
- **Ruta y Paquete**: `net.minecraft.src.IWorldAccess` (`net/minecraft/src/IWorldAccess.java`) | **Líneas de código**: 29
- **Declaración**: `interface IWorldAccess`
- **Rol**: Interfaz de escucha para eventos que alteran el entorno del mundo.
- **Descripción Técnica**: Recibe notificaciones de: markBlockAndNeighborsNeedsUpdate, playSound, spawnParticle y releaseEntitySkin.
- **Guía de Modificación**: Implementar para crear sistemas de monitoreo, mapas en tiempo real o grabadores de eventos.

##### <a id="class-metadatachunkblock"></a>[`MetadataChunkBlock`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MetadataChunkBlock.java)
- **Ruta y Paquete**: `net.minecraft.src.MetadataChunkBlock` (`net/minecraft/src/MetadataChunkBlock.java`) | **Líneas de código**: 202
- **Declaración**: `class MetadataChunkBlock`
- **Rol**: Estructura auxiliar que empaqueta cambios de bloques en chunks para red.
- **Descripción Técnica**: Almacena coordenadas locales y nuevos valores para transmisión masiva en paquetes de red.
- **Guía de Modificación**: Parte del protocolo de sincronización multijugador.
- **Métodos Clave Destacados**:
  - `void func_4127_a(World world)`
  - `boolean func_866_a(int i, int j, int k, int l, int i1, int j1)`
- **Campos Clave Destacados**:
  - `EnumSkyBlock field_1299_a`
  - `int field_1298_b`
  - `int field_1304_c`
  - `int field_1303_d`
  - `int field_1302_e`
  - `int field_1301_f`

##### <a id="class-nextticklistentry"></a>[`NextTickListEntry`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NextTickListEntry.java)
- **Ruta y Paquete**: `net.minecraft.src.NextTickListEntry` (`net/minecraft/src/NextTickListEntry.java`) | **Líneas de código**: 73
- **Declaración**: `class NextTickListEntry implements Comparable
`
- **Rol**: Entrada individual en la cola de ticks programados de World.
- **Descripción Técnica**: Almacena coordenadas (x, y, z), ID del bloque y tiempo límite programado para su actualización.
- **Guía de Modificación**: Permite programar acciones diferidas en bloques (ej. repetidores de redstone, caída de arena).
- **Métodos Clave Destacados**:
  - `boolean equals(Object obj)`
  - `int hashCode()`
  - `NextTickListEntry setScheduledTime(long l)`
  - `int comparer(NextTickListEntry nextticklistentry)`
  - `int compareTo(Object obj)`
- **Campos Clave Destacados**:
  - `int xCoord`
  - `int yCoord`
  - `int zCoord`
  - `int blockID`
  - `long scheduledTime`

##### <a id="class-nibblearray"></a>[`NibbleArray`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NibbleArray.java)
- **Ruta y Paquete**: `net.minecraft.src.NibbleArray` (`net/minecraft/src/NibbleArray.java`) | **Líneas de código**: 54
- **Declaración**: `class NibbleArray`
- **Rol**: Array de memoria comprimida que almacena 2 valores de 4 bits en cada byte.
- **Descripción Técnica**: Reduce el consumo de memoria a la mitad para datos de metadatos (0-15) y niveles de luz de chunks (0-15).
- **Guía de Modificación**: Estructura indispensable en la arquitectura de memoria de chunks de Minecraft.
- **Métodos Clave Destacados**:
  - `int getNibble(int i, int j, int k)`
  - `void setNibble(int i, int j, int k, int l)`
  - `boolean isValid()`

##### <a id="class-teleporter"></a>[`Teleporter`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Teleporter.java)
- **Ruta y Paquete**: `net.minecraft.src.Teleporter` (`net/minecraft/src/Teleporter.java`) | **Líneas de código**: 302
- **Declaración**: `class Teleporter`
- **Rol**: Gestor cinemático de portales y translación de coordenadas entre dimensiones.
- **Descripción Técnica**: Traduce coordenadas con escala 8:1 entre Overworld y Nether, localiza portales existentes o genera nuevos marcos de obsidiana.
- **Guía de Modificación**: Modificar para implementar sistemas de teletransporte personalizados entre dimensiones.
- **Métodos Clave Destacados**:
  - `void func_4107_a(World world, Entity entity)`
  - `boolean func_4106_b(World world, Entity entity)`
  - `boolean func_4108_c(World world, Entity entity)`

##### <a id="class-world"></a>[`World`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/World.java)
- **Ruta y Paquete**: `net.minecraft.src.World` (`net/minecraft/src/World.java`) | **Líneas de código**: 2448
- **Declaración**: `class World implements IBlockAccess
`
- **Rol**: El gestor integral del estado del mundo, físicas, iluminación y entidades.
- **Descripción Técnica**: La clase más extensa del juego (~2500 líneas). Mantiene listas de entidades vivas, TileEntities, chunks cargados, ciclo día/noche, clima, propagación de luz y cola de ticks.
- **Guía de Modificación**: Casi cualquier mod que altere el entorno, bloques o gravedad interactúa intensamente con World.
- **Métodos Clave Destacados**:
  - `NBTTagCompound func_629_a(File file, String s)`
  - `void deleteWorld(File file, String s)`
  - `WorldChunkManager func_4075_a()`
  - `IChunkProvider func_4081_a(File file)`
  - `void func_4076_b()`
  - `int func_614_g(int i, int j)`
- **Campos Clave Destacados**:
  - `boolean field_4214_a`
  - `List loadedEntityList`
  - `List loadedTileEntityList`
  - `List playerEntities`
  - `long worldTime`
  - `int skylightSubtracted`

##### <a id="class-worldblockpositiontype"></a>[`WorldBlockPositionType`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldBlockPositionType.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldBlockPositionType` (`net/minecraft/src/WorldBlockPositionType.java`) | **Líneas de código**: 28
- **Declaración**: `class WorldBlockPositionType`
- **Rol**: Tupla que asocia una posición tridimensional con el tipo de bloque.
- **Descripción Técnica**: Utilizada en algoritmos de análisis ambiental.
- **Guía de Modificación**: Estructura auxiliar de consulta.

##### <a id="class-worldchunkmanager"></a>[`WorldChunkManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldChunkManager.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldChunkManager` (`net/minecraft/src/WorldChunkManager.java`) | **Líneas de código**: 134
- **Declaración**: `class WorldChunkManager`
- **Rol**: Gestor climático y de biomas para el mundo Overworld.
- **Descripción Técnica**: Genera capas de temperatura y humedad para clasificar cada coordenada en bosques, llanuras, desiertos, taigas o selvas.
- **Guía de Modificación**: Punto de enganche para añadir nuevos biomas y variar su frecuencia o distribución geográfica.
- **Métodos Clave Destacados**:
  - `MobSpawnerBase func_4074_a(ChunkCoordIntPair chunkcoordintpair)`
  - `MobSpawnerBase func_4073_a(int i, int j)`
  - `double func_4072_b(int i, int j)`
  - `MobSpawnerBase[] func_4069_a(int i, int j, int k, int l)`
  - `double[] getTemperatures(double ad[], int i, int j, int k, int l)`
  - `MobSpawnerBase[] loadBlockGeneratorData(MobSpawnerBase amobspawnerbase[], int i, int j, int k, int l)`

##### <a id="class-worldchunkmanagerhell"></a>[`WorldChunkManagerHell`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldChunkManagerHell.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldChunkManagerHell` (`net/minecraft/src/WorldChunkManagerHell.java`) | **Líneas de código**: 66
- **Declaración**: `class WorldChunkManagerHell extends WorldChunkManager`
- **Rol**: Gestor de bioma único para el Inframundo (Nether).
- **Descripción Técnica**: Devuelve condiciones homogéneas extremas de calor y aridez en todas las coordenadas.
- **Guía de Modificación**: Define el clima perpetuo del Nether.
- **Métodos Clave Destacados**:
  - `MobSpawnerBase func_4074_a(ChunkCoordIntPair chunkcoordintpair)`
  - `MobSpawnerBase func_4073_a(int i, int j)`
  - `double func_4072_b(int i, int j)`
  - `MobSpawnerBase[] func_4069_a(int i, int j, int k, int l)`
  - `double[] getTemperatures(double ad[], int i, int j, int k, int l)`
  - `MobSpawnerBase[] loadBlockGeneratorData(MobSpawnerBase amobspawnerbase[], int i, int j, int k, int l)`

##### <a id="class-worldclient"></a>[`WorldClient`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldClient.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldClient` (`net/minecraft/src/WorldClient.java`) | **Líneas de código**: 262
- **Declaración**: `class WorldClient extends World`
- **Rol**: Implementación de World especializada para el cliente multijugador.
- **Descripción Técnica**: En lugar de simular físicas de bloques y spawn de mobs, actualiza el mundo exclusivamente a partir de los paquetes recibidos del servidor.
- **Guía de Modificación**: Diferencia fundamental entre la simulación local y el cliente SMP.
- **Métodos Clave Destacados**:
  - `void tick()`
  - `void func_711_c(int i, int j, int k, int l, int i1, int j1)`
  - `IChunkProvider func_4081_a(File file)`
  - `void func_4076_b()`
  - `void func_4080_j()`
  - `void scheduleBlockUpdate(int i, int j, int k, int l)`

##### <a id="class-worldiso"></a>[`WorldIso`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldIso.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldIso` (`net/minecraft/src/WorldIso.java`) | **Líneas de código**: 23
- **Declaración**: `class WorldIso extends World`
- **Rol**: Mundo adaptado para la representación isométrica en el visor de mapas.
- **Descripción Técnica**: Optimiza la consulta de bloques y colores para generar la proyección ortogonal.
- **Guía de Modificación**: Ajustar para añadir capas o filtros al mapa isométrico.
- **Métodos Clave Destacados**:
  - `IChunkProvider func_4081_a(File file)`

##### <a id="class-worldprovider"></a>[`WorldProvider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldProvider.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldProvider` (`net/minecraft/src/WorldProvider.java`) | **Líneas de código**: 147
- **Declaración**: `class WorldProvider`
- **Rol**: Abstracción ambiental de dimensiones (Overworld vs Nether).
- **Descripción Técnica**: Controla la escala de coordenadas, ciclo día/noche, niebla, color del cielo y generador de terreno aplicable.
- **Guía de Modificación**: Extender para crear dimensiones completamente nuevas (ej. Aether, Luna).
- **Métodos Clave Destacados**:
  - `void registerWorld(World world)`
  - `void generateLightBrightnessTable()`
  - `void registerWorldChunkManager()`
  - `IChunkProvider getChunkProvider()`
  - `IChunkLoader getChunkLoader(File file)`
  - `boolean canCoordinateBeSpawn(int i, int j)`
- **Campos Clave Destacados**:
  - `World worldObj`
  - `WorldChunkManager worldChunkMgr`
  - `boolean field_4220_c`
  - `boolean field_6479_d`
  - `boolean field_6478_e`
  - `int field_4218_e`

##### <a id="class-worldproviderhell"></a>[`WorldProviderHell`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldProviderHell.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldProviderHell` (`net/minecraft/src/WorldProviderHell.java`) | **Líneas de código**: 75
- **Declaración**: `class WorldProviderHell extends WorldProvider`
- **Rol**: Implementación ambiental específica del Inframundo (Nether).
- **Descripción Técnica**: Deshabilita el ciclo solar, fija niebla roja densa y escala las distancias en proporción 8:1.
- **Guía de Modificación**: Ejemplo clave de cómo personalizar una dimensión alternativa.
- **Métodos Clave Destacados**:
  - `void registerWorldChunkManager()`
  - `Vec3D func_4096_a(float f, float f1)`
  - `void generateLightBrightnessTable()`
  - `IChunkProvider getChunkProvider()`
  - `IChunkLoader getChunkLoader(File file)`
  - `boolean canCoordinateBeSpawn(int i, int j)`

##### <a id="class-worldrenderer"></a>[`WorldRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/WorldRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.WorldRenderer` (`net/minecraft/src/WorldRenderer.java`) | **Líneas de código**: 282
- **Declaración**: `class WorldRenderer`
- **Rol**: Compilador de fragmentos de terreno en listas de visualización OpenGL (Display Lists).
- **Descripción Técnica**: Divide el mundo en bloques de 16x16x16 y compila su geometría estática en listas de visualización de OpenGL para que la tarjeta gráfica los dibuje a más de 60 FPS sin recalcular geometría cada fotograma.
- **Guía de Modificación**: Modificar esta clase para optimizaciones de renderizado o shaders personalizados.
- **Métodos Clave Destacados**:
  - `void func_1197_a(int i, int j, int k)`
  - `void func_1198_a()`
  - `float func_1202_a(Entity entity)`
  - `void func_1195_b()`
  - `void func_1204_c()`
  - `int func_1200_a(int i)`
- **Campos Clave Destacados**:
  - `World worldObj`
  - `int field_1762_b`
  - `int field_1761_c`
  - `int field_1760_d`
  - `int field_1759_e`
  - `int field_1758_f`


### 07. TileEntities y Contenedores (TileEntities & Containers)
> **Resumen del subsistema**: Entidades asociadas a bloques específicos para almacenar datos complejos que superan los 4 bits de metadato. Incluye hornos, cofres, carteles de texto, generadores de monstruos y tocadiscos, junto con la sincronización de inventarios (Containers y Slots).

**Total de clases**: 13

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`IInventory`](#class-iinventory) | `interface` | `-` | 23 | Interfaz estándar para cualquier contenedor con casillas de inven... |
| [`Slot`](#class-slot) | `class` | `-` | 54 | Representación lógica y visual de una casilla individual en un co... |
| [`SlotArmor`](#class-slotarmor) | `class` | `ext SlotInventory` | 46 | Casilla especializada de inventario para piezas de armadura. |
| [`SlotInventory`](#class-slotinventory) | `class` | `ext Slot` | 30 | Casilla de inventario estándar sin restricciones de contenido. |
| [`TileEntity`](#class-tileentity) | `class` | `-` | 131 | Clase base abstracta para bloques con datos persistentes complejo... |
| [`TileEntityChest`](#class-tileentitychest) | `class` | `ext TileEntity, impl IInventory
` | 106 | Inventario de almacenamiento persistente para cofres (27 casillas... |
| [`TileEntityFurnace`](#class-tileentityfurnace) | `class` | `ext TileEntity, impl IInventory
` | 293 | Lógica interna de cocinado y quemado de combustible del horno. |
| [`TileEntityMobSpawner`](#class-tileentitymobspawner) | `class` | `ext TileEntity` | 118 | Generador automático de monstruos con rotación visual interna. |
| [`TileEntityMobSpawnerRenderer`](#class-tileentitymobspawnerrenderer) | `class` | `ext TileEntitySpecialRenderer` | 51 | Renderizador 3D especial para el generador de monstruos (Mob Spaw... |
| [`TileEntityRenderer`](#class-tileentityrenderer) | `class` | `-` | 105 | Despachador central de renderizado para TileEntities. |
| [`TileEntitySign`](#class-tileentitysign) | `class` | `ext TileEntity` | 42 | Almacenamiento de las 4 líneas de texto del cartel. |
| [`TileEntitySignRenderer`](#class-tileentitysignrenderer) | `class` | `ext TileEntitySpecialRenderer` | 86 | Renderizador 3D para carteles y su texto. |
| [`TileEntitySpecialRenderer`](#class-tileentityspecialrenderer) | `class` | `-` | 34 | Clase base abstracta para renderizado 3D de TileEntities fuera de... |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-iinventory"></a>[`IInventory`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IInventory.java)
- **Ruta y Paquete**: `net.minecraft.src.IInventory` (`net/minecraft/src/IInventory.java`) | **Líneas de código**: 23
- **Declaración**: `interface IInventory`
- **Rol**: Interfaz estándar para cualquier contenedor con casillas de inventario.
- **Descripción Técnica**: Declara getSizeInventory, getStackInSlot, decrStackSize, setInventorySlotContents y getInventoryStackLimit.
- **Guía de Modificación**: Implementar obligatoriamente en cualquier bloque, mob o vehículo que almacene ítems.

##### <a id="class-slot"></a>[`Slot`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Slot.java)
- **Ruta y Paquete**: `net.minecraft.src.Slot` (`net/minecraft/src/Slot.java`) | **Líneas de código**: 54
- **Declaración**: `class Slot`
- **Rol**: Representación lógica y visual de una casilla individual en un contenedor.
- **Descripción Técnica**: Define coordenadas en pantalla (xDisplayPosition, yDisplayPosition), el inventario asociado y restricciones de inserción.
- **Guía de Modificación**: Extender para crear casillas con filtros de ítems (ej. solo armaduras o combustibles).
- **Métodos Clave Destacados**:
  - `void onPickupFromSlot()`
  - `boolean isItemValid(ItemStack itemstack)`
  - `ItemStack getStack()`
  - `void putStack(ItemStack itemstack)`
  - `int func_775_c()`
  - `void onSlotChanged()`
- **Campos Clave Destacados**:
  - `int slotIndex`
  - `IInventory inventory`

##### <a id="class-slotarmor"></a>[`SlotArmor`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SlotArmor.java)
- **Ruta y Paquete**: `net.minecraft.src.SlotArmor` (`net/minecraft/src/SlotArmor.java`) | **Líneas de código**: 46
- **Declaración**: `class SlotArmor extends SlotInventory`
- **Rol**: Casilla especializada de inventario para piezas de armadura.
- **Descripción Técnica**: Restringe la colocación exclusivamente al tipo de armadura correspondiente (casco, peto, pantalones o botas).
- **Guía de Modificación**: Referencia para crear casillas con restricciones de equipamiento.
- **Métodos Clave Destacados**:
  - `int getSlotStackLimit()`
  - `boolean isItemValid(ItemStack itemstack)`
  - `int func_775_c()`

##### <a id="class-slotinventory"></a>[`SlotInventory`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SlotInventory.java)
- **Ruta y Paquete**: `net.minecraft.src.SlotInventory` (`net/minecraft/src/SlotInventory.java`) | **Líneas de código**: 30
- **Declaración**: `class SlotInventory extends Slot`
- **Rol**: Casilla de inventario estándar sin restricciones de contenido.
- **Descripción Técnica**: Permite colocar y retirar cualquier tipo de ítem en contenedores genéricos.
- **Guía de Modificación**: Utilizada en cofres, hornos y almacenamiento.
- **Métodos Clave Destacados**:
  - `boolean isAtCursorPos(int i, int j)`
- **Campos Clave Destacados**:
  - `int xPos`
  - `int yPos`

##### <a id="class-tileentity"></a>[`TileEntity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntity.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntity` (`net/minecraft/src/TileEntity.java`) | **Líneas de código**: 131
- **Declaración**: `class TileEntity`
- **Rol**: Clase base abstracta para bloques con datos persistentes complejos y lógica por tick.
- **Descripción Técnica**: Se vincula a una posición fija (xCoord, yCoord, zCoord). Implementa updateEntity() para ejecutarse en cada tick y serializa sus variables en etiquetas NBT.
- **Guía de Modificación**: Heredar de esta clase para cualquier bloque que requiera inventario, energía o procesamiento continuo.
- **Métodos Clave Destacados**:
  - `void readFromNBT(NBTTagCompound nbttagcompound)`
  - `void writeToNBT(NBTTagCompound nbttagcompound)`
  - `void updateEntity()`
  - `TileEntity createAndLoadEntity(NBTTagCompound nbttagcompound)`
  - `int getBlockMetadata()`
  - `void onInventoryChanged()`
- **Campos Clave Destacados**:
  - `World worldObj`
  - `int xCoord`
  - `int yCoord`
  - `int zCoord`

##### <a id="class-tileentitychest"></a>[`TileEntityChest`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntityChest.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntityChest` (`net/minecraft/src/TileEntityChest.java`) | **Líneas de código**: 106
- **Declaración**: `class TileEntityChest extends TileEntity implements IInventory
`
- **Rol**: Inventario de almacenamiento persistente para cofres (27 casillas).
- **Descripción Técnica**: Implementa IInventory gestionando el array de ItemStacks guardados, detecta aperturas y cierres para sonidos y animación de tapa.
- **Guía de Modificación**: Modificar getSizeInventory() para crear cofres con mayor capacidad.
- **Métodos Clave Destacados**:
  - `int getSizeInventory()`
  - `ItemStack getStackInSlot(int i)`
  - `ItemStack decrStackSize(int i, int j)`
  - `void setInventorySlotContents(int i, ItemStack itemstack)`
  - `String getInvName()`
  - `void readFromNBT(NBTTagCompound nbttagcompound)`

##### <a id="class-tileentityfurnace"></a>[`TileEntityFurnace`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntityFurnace.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntityFurnace` (`net/minecraft/src/TileEntityFurnace.java`) | **Líneas de código**: 293
- **Declaración**: `class TileEntityFurnace extends TileEntity implements IInventory
`
- **Rol**: Lógica interna de cocinado y quemado de combustible del horno.
- **Descripción Técnica**: Gestiona los tiempos de cocción (furnaceCookTime), combustible restante (furnaceBurnTime), itemStacks de entrada/combustible/salida e interactúa con FurnaceRecipes.
- **Guía de Modificación**: Modifica getItemBurnTime() para admitir nuevos combustibles (ej. turba, lava comprimida).
- **Métodos Clave Destacados**:
  - `int getSizeInventory()`
  - `ItemStack getStackInSlot(int i)`
  - `ItemStack decrStackSize(int i, int j)`
  - `void setInventorySlotContents(int i, ItemStack itemstack)`
  - `String getInvName()`
  - `void readFromNBT(NBTTagCompound nbttagcompound)`

##### <a id="class-tileentitymobspawner"></a>[`TileEntityMobSpawner`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntityMobSpawner.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntityMobSpawner` (`net/minecraft/src/TileEntityMobSpawner.java`) | **Líneas de código**: 118
- **Declaración**: `class TileEntityMobSpawner extends TileEntity`
- **Rol**: Generador automático de monstruos con rotación visual interna.
- **Descripción Técnica**: Spawnea monstruos del tipo configurado en 'mobID' cuando el jugador se encuentra a menos de 16 bloques, con intervalos aleatorios de retardo.
- **Guía de Modificación**: Modificar la distancia de activación o el mobID para crear spawners personalizados.
- **Métodos Clave Destacados**:
  - `boolean anyPlayerInRange()`
  - `void updateEntity()`
  - `void readFromNBT(NBTTagCompound nbttagcompound)`
  - `void writeToNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int delay`
  - `String entityID`
  - `double field_831_c`
  - `double field_830_d`

##### <a id="class-tileentitymobspawnerrenderer"></a>[`TileEntityMobSpawnerRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntityMobSpawnerRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntityMobSpawnerRenderer` (`net/minecraft/src/TileEntityMobSpawnerRenderer.java`) | **Líneas de código**: 51
- **Declaración**: `class TileEntityMobSpawnerRenderer extends TileEntitySpecialRenderer`
- **Rol**: Renderizador 3D especial para el generador de monstruos (Mob Spawner).
- **Descripción Técnica**: Dibuja el bloque con su jaula metálica y dibuja una miniatura flotante del mob girando continuamente en su interior.
- **Guía de Modificación**: Modificar para alterar la escala o animación del mob interior.
- **Métodos Clave Destacados**:
  - `void func_931_a(TileEntityMobSpawner tileentitymobspawner, double d, double d1, double d2, float f)`
  - `void renderTileEntityAt(TileEntity tileentity, double d, double d1, double d2, float f)`

##### <a id="class-tileentityrenderer"></a>[`TileEntityRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntityRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntityRenderer` (`net/minecraft/src/TileEntityRenderer.java`) | **Líneas de código**: 105
- **Declaración**: `class TileEntityRenderer`
- **Rol**: Despachador central de renderizado para TileEntities.
- **Descripción Técnica**: Mapea cada clase de TileEntity con su TileEntitySpecialRenderer correspondiente e invoca el renderizado en la posición del bloque.
- **Guía de Modificación**: Registrar aquí cualquier nuevo renderizador de TileEntity personalizado.
- **Métodos Clave Destacados**:
  - `TileEntitySpecialRenderer getSpecialRendererForClass(Class class1)`
  - `boolean hasSpecialRenderer(TileEntity tileentity)`
  - `TileEntitySpecialRenderer getSpecialRendererForEntity(TileEntity tileentity)`
  - `void setRenderingContext(World world, RenderEngine renderengine, FontRenderer fontrenderer, EntityPlayer entityplayer, float f)`
  - `void renderTileEntity(TileEntity tileentity, float f)`
  - `void renderTileEntityAt(TileEntity tileentity, double d, double d1, double d2, float f)`
- **Campos Clave Destacados**:
  - `TileEntityRenderer instance`
  - `double staticPlayerX`
  - `double staticPlayerY`
  - `double staticPlayerZ`
  - `RenderEngine renderEngine`
  - `World worldObj`

##### <a id="class-tileentitysign"></a>[`TileEntitySign`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntitySign.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntitySign` (`net/minecraft/src/TileEntitySign.java`) | **Líneas de código**: 42
- **Declaración**: `class TileEntitySign extends TileEntity`
- **Rol**: Almacenamiento de las 4 líneas de texto del cartel.
- **Descripción Técnica**: Guarda las cadenas 'signText[4]', gestiona su sincronización multijugador con Packet130UpdateSign y permite su renderizado 3D mediante TileEntitySignRenderer.
- **Guía de Modificación**: Permite ampliar el número de líneas o añadir formato de color y comandos a carteles.
- **Métodos Clave Destacados**:
  - `void writeToNBT(NBTTagCompound nbttagcompound)`
  - `void readFromNBT(NBTTagCompound nbttagcompound)`
- **Campos Clave Destacados**:
  - `int lineBeingEdited`

##### <a id="class-tileentitysignrenderer"></a>[`TileEntitySignRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntitySignRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntitySignRenderer` (`net/minecraft/src/TileEntitySignRenderer.java`) | **Líneas de código**: 86
- **Declaración**: `class TileEntitySignRenderer extends TileEntitySpecialRenderer`
- **Rol**: Renderizador 3D para carteles y su texto.
- **Descripción Técnica**: Dibuja el modelo SignModel, orienta el cartel según su rotación y renderiza las 4 líneas de texto con FontRenderer aplicando escala y centrado.
- **Guía de Modificación**: Modificar para permitir texto brillante o fuentes especiales en carteles.
- **Métodos Clave Destacados**:
  - `void renderTileEntitySignAt(TileEntitySign tileentitysign, double d, double d1, double d2, float f)`
  - `void renderTileEntityAt(TileEntity tileentity, double d, double d1, double d2, float f)`

##### <a id="class-tileentityspecialrenderer"></a>[`TileEntitySpecialRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TileEntitySpecialRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.TileEntitySpecialRenderer` (`net/minecraft/src/TileEntitySpecialRenderer.java`) | **Líneas de código**: 34
- **Declaración**: `class TileEntitySpecialRenderer`
- **Rol**: Clase base abstracta para renderizado 3D de TileEntities fuera de la malla estática de bloques.
- **Descripción Técnica**: Permite emitir comandos directos de OpenGL en la posición del bloque para geometrías complejas o animadas.
- **Guía de Modificación**: Heredar de esta clase para renderizar bloques dinámicos (tuberías, máquinas, cofres animados).
- **Métodos Clave Destacados**:
  - `void bindTextureByName(String s)`
  - `void setTileEntityRenderer(TileEntityRenderer tileentityrenderer)`
  - `FontRenderer getFontRenderer()`
- **Campos Clave Destacados**:
  - `TileEntityRenderer tileEntityRenderer`


### 08. Motor de Renderizado y Gráficos (Rendering Pipeline, Models & FX)
> **Resumen del subsistema**: La tubería gráfica basada en OpenGL 1.x/2.x y LWJGL. Destaca el Tessellator (emisión masiva de quads mediante búferes NIO nativos), listas de visualización de chunks (WorldRenderer), renderizado de bloques (RenderBlocks), modelos jerárquicos de entidades (ModelBase) y motor de partículas (EffectRenderer).

**Total de clases**: 83

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`ClippingHelper`](#class-clippinghelper) | `class` | `-` | 35 | Clase base abstracta para el cálculo del volumen de visión de la ... |
| [`ClippingHelperImplementation`](#class-clippinghelperimplementation) | `class` | `ext ClippingHelper` | 99 | Implementación concreta de ClippingHelper sobre OpenGL. |
| [`ColorizerFoliage`](#class-colorizerfoliage) | `class` | `-` | 51 | Tabla de colores 256x256 para el follaje de hojas de árboles segú... |
| [`ColorizerGrass`](#class-colorizergrass) | `class` | `-` | 51 | Tabla de colores 256x256 para el césped superficial según bioma. |
| [`EffectRenderer`](#class-effectrenderer) | `class` | `-` | 196 | Gestor central del motor de partículas del juego. |
| [`FontRenderer`](#class-fontrenderer) | `class` | `-` | 223 | Renderizador de fuentes bitmap tipográficas y códigos de formato/... |
| [`Frustrum`](#class-frustrum) | `class` | `impl ICamera
` | 37 | Verificador de inclusión espacial en el cono de visión de la cáma... |
| [`GLAllocation`](#class-glallocation) | `class` | `-` | 76 | Gestor central de asignación de memoria nativa y recursos OpenGL. |
| [`GuiTexturePacks`](#class-guitexturepacks) | `class` | `ext GuiScreen` | 240 | Pantalla o elemento de interfaz gráfica: 'GuiTexturePacks'. |
| [`ICamera`](#class-icamera) | `interface` | `-` | 13 | Interfaz para comprobaciones de visibilidad geométrica en la cáma... |
| [`ImageBuffer`](#class-imagebuffer) | `interface` | `-` | 12 | Interfaz para procesamiento de búferes de imágenes descargadas. |
| [`ImageBufferDownload`](#class-imagebufferdownload) | `class` | `impl ImageBuffer
` | 118 | Procesador de skins descargadas de internet. |
| [`IsoImageBuffer`](#class-isoimagebuffer) | `class` | `-` | 44 | Búfer de píxeles para almacenar fragmentos del mapa isométrico ge... |
| [`LoadingScreenRenderer`](#class-loadingscreenrenderer) | `class` | `impl IProgressUpdate
` | 160 | Renderizador de pantallas de carga y progreso. |
| [`LogoEffectRandomizer`](#class-logoeffectrandomizer) | `class` | `-` | 37 | Efecto de perturbación y dispersión procedural para el logo de Mo... |
| [`ModelBase`](#class-modelbase) | `class` | `-` | 25 | Modelo geométrico cúbico 3D: 'ModelBase'. |
| [`ModelBiped`](#class-modelbiped) | `class` | `ext ModelBase` | 153 | Modelo geométrico cúbico 3D: 'ModelBiped'. |
| [`ModelBoat`](#class-modelboat) | `class` | `ext ModelBase` | 52 | Modelo geométrico cúbico 3D: 'ModelBoat'. |
| [`ModelChicken`](#class-modelchicken) | `class` | `ext ModelBase` | 75 | Modelo geométrico cúbico 3D: 'ModelChicken'. |
| [`ModelCow`](#class-modelcow) | `class` | `ext ModelQuadraped` | 59 | Modelo geométrico cúbico 3D: 'ModelCow'. |
| [`ModelCreeper`](#class-modelcreeper) | `class` | `ext ModelBase` | 65 | Modelo geométrico cúbico 3D: 'ModelCreeper'. |
| [`ModelGhast`](#class-modelghast) | `class` | `ext ModelBase` | 55 | Modelo geométrico cúbico 3D: 'ModelGhast'. |
| [`ModelMinecart`](#class-modelminecart) | `class` | `ext ModelBase` | 57 | Modelo geométrico cúbico 3D: 'ModelMinecart'. |
| [`ModelPig`](#class-modelpig) | `class` | `ext ModelQuadraped` | 19 | Modelo geométrico cúbico 3D: 'ModelPig'. |
| [`ModelQuadraped`](#class-modelquadraped) | `class` | `ext ModelBase` | 60 | Modelo geométrico cúbico 3D: 'ModelQuadraped'. |
| [`ModelRenderer`](#class-modelrenderer) | `class` | `-` | 206 | Modelo geométrico cúbico 3D: 'ModelRenderer'. |
| [`ModelSheep1`](#class-modelsheep1) | `class` | `ext ModelQuadraped` | 33 | Modelo geométrico cúbico 3D: 'ModelSheep1'. |
| [`ModelSheep2`](#class-modelsheep2) | `class` | `ext ModelQuadraped` | 20 | Modelo geométrico cúbico 3D: 'ModelSheep2'. |
| [`ModelSkeleton`](#class-modelskeleton) | `class` | `ext ModelZombie` | 28 | Modelo geométrico cúbico 3D: 'ModelSkeleton'. |
| [`ModelSlime`](#class-modelslime) | `class` | `ext ModelBase` | 47 | Modelo geométrico cúbico 3D: 'ModelSlime'. |
| [`ModelSpider`](#class-modelspider) | `class` | `ext ModelBase` | 125 | Modelo geométrico cúbico 3D: 'ModelSpider'. |
| [`ModelZombie`](#class-modelzombie) | `class` | `ext ModelBiped` | 32 | Modelo geométrico cúbico 3D: 'ModelZombie'. |
| [`OpenGlCapsChecker`](#class-openglcapschecker) | `class` | `-` | 23 | Verificador de capacidades y extensiones de la tarjeta gráfica. |
| [`PositionTexureVertex`](#class-positiontexurevertex) | `class` | `-` | 37 | Vértice espacial 3D con coordenadas UV de mapeado de textura. |
| [`Render`](#class-render) | `class` | `-` | 267 | Renderizador en espacio 3D para 'Render'. |
| [`RenderArrow`](#class-renderarrow) | `class` | `ext Render` | 79 | Renderizador en espacio 3D para 'RenderArrow'. |
| [`RenderBiped`](#class-renderbiped) | `class` | `ext RenderLiving` | 56 | Renderizador en espacio 3D para 'RenderBiped'. |
| [`RenderBlocks`](#class-renderblocks) | `class` | `-` | 2078 | Renderizador en espacio 3D para 'RenderBlocks'. |
| [`RenderBoat`](#class-renderboat) | `class` | `ext Render` | 50 | Renderizador en espacio 3D para 'RenderBoat'. |
| [`RenderChicken`](#class-renderchicken) | `class` | `ext RenderLiving` | 44 | Renderizador en espacio 3D para 'RenderChicken'. |
| [`RenderCow`](#class-rendercow) | `class` | `ext RenderLiving` | 32 | Renderizador en espacio 3D para 'RenderCow'. |
| [`RenderCreeper`](#class-rendercreeper) | `class` | `ext RenderLiving` | 68 | Renderizador en espacio 3D para 'RenderCreeper'. |
| [`RenderEngine`](#class-renderengine) | `class` | `-` | 469 | Renderizador en espacio 3D para 'RenderEngine'. |
| [`RenderEntity`](#class-renderentity) | `class` | `ext Render` | 22 | Renderizador en espacio 3D para 'RenderEntity'. |
| [`RenderFallingSand`](#class-renderfallingsand) | `class` | `ext Render` | 38 | Renderizador en espacio 3D para 'RenderFallingSand'. |
| [`RenderFireball`](#class-renderfireball) | `class` | `ext Render` | 51 | Renderizador en espacio 3D para 'RenderFireball'. |
| [`RenderFish`](#class-renderfish) | `class` | `ext Render` | 92 | Renderizador en espacio 3D para 'RenderFish'. |
| [`RenderGhast`](#class-renderghast) | `class` | `ext RenderLiving` | 35 | Renderizador en espacio 3D para 'RenderGhast'. |
| [`RenderGlobal`](#class-renderglobal) | `class` | `impl IWorldAccess
` | 1286 | Renderizador en espacio 3D para 'RenderGlobal'. |
| [`RenderHelper`](#class-renderhelper) | `class` | `-` | 63 | Renderizador en espacio 3D para 'RenderHelper'. |
| [`RenderItem`](#class-renderitem) | `class` | `ext Render` | 218 | Renderizador en espacio 3D para 'RenderItem'. |
| [`RenderList`](#class-renderlist) | `class` | `-` | 87 | Renderizador en espacio 3D para 'RenderList'. |
| [`RenderLiving`](#class-renderliving) | `class` | `ext Render` | 173 | Renderizador en espacio 3D para 'RenderLiving'. |
| [`RenderManager`](#class-rendermanager) | `class` | `-` | 134 | Renderizador en espacio 3D para 'RenderManager'. |
| [`RenderMinecart`](#class-renderminecart) | `class` | `ext Render` | 95 | Renderizador en espacio 3D para 'RenderMinecart'. |
| [`RenderPainting`](#class-renderpainting) | `class` | `ext Render` | 136 | Renderizador en espacio 3D para 'RenderPainting'. |
| [`RenderPig`](#class-renderpig) | `class` | `ext RenderLiving` | 26 | Renderizador en espacio 3D para 'RenderPig'. |
| [`RenderPlayer`](#class-renderplayer) | `class` | `ext RenderLiving` | 235 | Renderizador en espacio 3D para 'RenderPlayer'. |
| [`RenderSheep`](#class-rendersheep) | `class` | `ext RenderLiving` | 26 | Renderizador en espacio 3D para 'RenderSheep'. |
| [`RenderSlime`](#class-renderslime) | `class` | `ext RenderLiving` | 54 | Renderizador en espacio 3D para 'RenderSlime'. |
| [`RenderSnowball`](#class-rendersnowball) | `class` | `ext Render` | 50 | Renderizador en espacio 3D para 'RenderSnowball'. |
| [`RenderSorter`](#class-rendersorter) | `class` | `impl Comparator
` | 50 | Renderizador en espacio 3D para 'RenderSorter'. |
| [`RenderSpider`](#class-renderspider) | `class` | `ext RenderLiving` | 52 | Renderizador en espacio 3D para 'RenderSpider'. |
| [`RenderTNTPrimed`](#class-rendertntprimed) | `class` | `ext Render` | 64 | Renderizador en espacio 3D para 'RenderTNTPrimed'. |
| [`RenderZombieSimple`](#class-renderzombiesimple) | `class` | `ext RenderLiving` | 28 | Renderizador en espacio 3D para 'RenderZombieSimple'. |
| [`ScaledResolution`](#class-scaledresolution) | `class` | `-` | 32 | Calculador dinámico del factor de escala de la interfaz de usuari... |
| [`SignModel`](#class-signmodel) | `class` | `-` | 26 | Modelo geométrico 3D del cartel (tablero y poste de soporte). |
| [`TerrainTextureManager`](#class-terraintexturemanager) | `class` | `-` | 268 | Administrador de atlas y texturas del terreno. |
| [`Tessellator`](#class-tessellator) | `class` | `-` | 355 | Motor de emisión geométrica de quads de alto rendimiento con búfe... |
| [`TextureCompassFX`](#class-texturecompassfx) | `class` | `ext TextureFX` | 137 | Animación procedural para la esfera de la brújula en el inventari... |
| [`TextureFX`](#class-texturefx) | `class` | `-` | 43 | Clase base para animaciones procedimentales de textura en memoria... |
| [`TextureFlamesFX`](#class-textureflamesfx) | `class` | `ext TextureFX` | 92 | Animación procedimental para las partículas y texturas de fuego. |
| [`TextureLavaFX`](#class-texturelavafx) | `class` | `ext TextureFX` | 93 | Animación procedimental para el bloque de lava estacionaria. |
| [`TextureLavaFlowFX`](#class-texturelavaflowfx) | `class` | `ext TextureFX` | 97 | Animación procedimental para la lava en flujo continuo. |
| [`TexturePackBase`](#class-texturepackbase) | `class` | `-` | 45 | Clase base abstracta para paquetes de texturas (Texture Packs). |
| [`TexturePackCustom`](#class-texturepackcustom) | `class` | `ext TexturePackBase` | 138 | Paquete de texturas personalizado cargado desde un archivo .zip e... |
| [`TexturePackDefault`](#class-texturepackdefault) | `class` | `ext TexturePackBase` | 55 | Paquete de texturas por defecto integrado en el cliente. |
| [`TexturePackList`](#class-texturepacklist) | `class` | `-` | 115 | Gestor del catálogo de paquetes de texturas disponibles en disco. |
| [`TexturePortalFX`](#class-textureportalfx) | `class` | `ext TextureFX` | 100 | Animación procedimental para la lámina del portal al Inframundo. |
| [`TextureWatchFX`](#class-texturewatchfx) | `class` | `ext TextureFX` | 105 | Animación procedural para la esfera del reloj en el inventario. |
| [`TextureWaterFX`](#class-texturewaterfx) | `class` | `ext TextureFX` | 99 | Animación procedimental para el bloque de agua estacionaria. |
| [`TexturedQuad`](#class-texturedquad) | `class` | `-` | 65 | Polígono cuadrilátero formado por 4 vértices PositionTexureVertex... |
| [`TexureWaterFlowFX`](#class-texurewaterflowfx) | `class` | `ext TextureFX` | 100 | Animación procedimental para el flujo horizontal del agua en movi... |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-clippinghelper"></a>[`ClippingHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ClippingHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.ClippingHelper` (`net/minecraft/src/ClippingHelper.java`) | **Líneas de código**: 35
- **Declaración**: `class ClippingHelper`
- **Rol**: Clase base abstracta para el cálculo del volumen de visión de la cámara (Frustum).
- **Descripción Técnica**: Extrae los 6 planos del frustum (izquierdo, derecho, superior, inferior, cercano, lejano) de las matrices OpenGL.
- **Guía de Modificación**: Base para algoritmos de descarte de geometría no visible (Frustum Culling).
- **Métodos Clave Destacados**:
  - `boolean func_1152_a(double d, double d1, double d2, double d3, double d4, double d5)`

##### <a id="class-clippinghelperimplementation"></a>[`ClippingHelperImplementation`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ClippingHelperImplementation.java)
- **Ruta y Paquete**: `net.minecraft.src.ClippingHelperImplementation` (`net/minecraft/src/ClippingHelperImplementation.java`) | **Líneas de código**: 99
- **Declaración**: `class ClippingHelperImplementation extends ClippingHelper`
- **Rol**: Implementación concreta de ClippingHelper sobre OpenGL.
- **Descripción Técnica**: Consulta GL_PROJECTION_MATRIX y GL_MODELVIEW_MATRIX mediante glGetFloat para actualizar los planos de corte.
- **Guía de Modificación**: Permite optimizar o depurar el sistema de culling.
- **Métodos Clave Destacados**:
  - `ClippingHelper func_1155_a()`

##### <a id="class-colorizerfoliage"></a>[`ColorizerFoliage`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ColorizerFoliage.java)
- **Ruta y Paquete**: `net.minecraft.src.ColorizerFoliage` (`net/minecraft/src/ColorizerFoliage.java`) | **Líneas de código**: 51
- **Declaración**: `class ColorizerFoliage`
- **Rol**: Tabla de colores 256x256 para el follaje de hojas de árboles según bioma.
- **Descripción Técnica**: Interpola el tono verde de las hojas en función de la temperatura y humedad calculadas.
- **Guía de Modificación**: Modificar foliage.png o esta clase para cambiar las paletas de color de los bosques.
- **Métodos Clave Destacados**:
  - `int func_4146_a(double d, double d1)`

##### <a id="class-colorizergrass"></a>[`ColorizerGrass`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ColorizerGrass.java)
- **Ruta y Paquete**: `net.minecraft.src.ColorizerGrass` (`net/minecraft/src/ColorizerGrass.java`) | **Líneas de código**: 51
- **Declaración**: `class ColorizerGrass`
- **Rol**: Tabla de colores 256x256 para el césped superficial según bioma.
- **Descripción Técnica**: Interpola el tono de verde/amarillo del bloque de césped según el clima local.
- **Guía de Modificación**: Modificar grass.png o esta clase para cambiar la paleta de color del suelo.
- **Métodos Clave Destacados**:
  - `int func_4147_a(double d, double d1)`

##### <a id="class-effectrenderer"></a>[`EffectRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/EffectRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.EffectRenderer` (`net/minecraft/src/EffectRenderer.java`) | **Líneas de código**: 196
- **Declaración**: `class EffectRenderer`
- **Rol**: Gestor central del motor de partículas del juego.
- **Descripción Técnica**: Mantiene listas de entidades EntityFX divididas en 4 capas de textura. Actualiza físicas y dibuja partículas con Tessellator.
- **Guía de Modificación**: Llama a addEffect(new EntityFX(...)) para spawnear cualquier partícula personalizada en el mundo.
- **Métodos Clave Destacados**:
  - `void func_1192_a(EntityFX entityfx)`
  - `void func_1193_a()`
  - `void func_1189_a(Entity entity, float f)`
  - `void func_1187_b(Entity entity, float f)`
  - `void func_1188_a(World world)`
  - `void func_1186_a(int i, int j, int k)`
- **Campos Clave Destacados**:
  - `World worldObj`

##### <a id="class-fontrenderer"></a>[`FontRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/FontRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.FontRenderer` (`net/minecraft/src/FontRenderer.java`) | **Líneas de código**: 223
- **Declaración**: `class FontRenderer`
- **Rol**: Renderizador de fuentes bitmap tipográficas y códigos de formato/color.
- **Descripción Técnica**: Dibuja cadenas de texto usando el atlas 'default.png'. Procesa códigos de escape '§' (0-f) para colores de texto.
- **Guía de Modificación**: Modificar para soportar fuentes de alta resolución, fuentes TrueType o códigos de formato extra.
- **Métodos Clave Destacados**:
  - `void drawStringWithShadow(String s, int i, int j, int k)`
  - `void drawString(String s, int i, int j, int k)`
  - `void renderString(String s, int i, int j, int k, boolean flag)`
  - `int getStringWidth(String s)`
- **Campos Clave Destacados**:
  - `int field_1308_a`

##### <a id="class-frustrum"></a>[`Frustrum`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Frustrum.java)
- **Ruta y Paquete**: `net.minecraft.src.Frustrum` (`net/minecraft/src/Frustrum.java`) | **Líneas de código**: 37
- **Declaración**: `class Frustrum implements ICamera
`
- **Rol**: Verificador de inclusión espacial en el cono de visión de la cámara.
- **Descripción Técnica**: Implementa ICamera. Evalúa si una caja delimitadora AxisAlignedBB se encuentra dentro de los 6 planos de visión.
- **Guía de Modificación**: Evita renderizar entidades y bloques que se encuentran a espaldas del jugador o fuera de la pantalla.
- **Métodos Clave Destacados**:
  - `void func_343_a(double d, double d1, double d2)`
  - `boolean func_344_a(double d, double d1, double d2, double d3, double d4, double d5)`
  - `boolean func_342_a(AxisAlignedBB axisalignedbb)`

##### <a id="class-glallocation"></a>[`GLAllocation`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GLAllocation.java)
- **Ruta y Paquete**: `net.minecraft.src.GLAllocation` (`net/minecraft/src/GLAllocation.java`) | **Líneas de código**: 76
- **Declaración**: `class GLAllocation`
- **Rol**: Gestor central de asignación de memoria nativa y recursos OpenGL.
- **Descripción Técnica**: Genera y elimina listas de visualización (glGenLists) y asigna ByteBuffers y IntBuffers en memoria directa.
- **Guía de Modificación**: Utilizar sus métodos para asignar búferes nativos de forma segura sin fugas de memoria.
- **Métodos Clave Destacados**:
  - `IntBuffer createDirectIntBuffer(int i)`
  - `FloatBuffer createDirectFloatBuffer(int i)`

##### <a id="class-guitexturepacks"></a>[`GuiTexturePacks`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiTexturePacks.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiTexturePacks` (`net/minecraft/src/GuiTexturePacks.java`) | **Líneas de código**: 240
- **Declaración**: `class GuiTexturePacks extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiTexturePacks'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiTexturePacks'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void mouseClicked(int i, int j, int k)`
  - `void mouseMovedOrUp(int i, int j, int k)`
  - `void drawScreen(int i, int j, float f)`
  - `void updateScreen()`
- **Campos Clave Destacados**:
  - `GuiScreen field_6461_a`

##### <a id="class-icamera"></a>[`ICamera`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ICamera.java)
- **Ruta y Paquete**: `net.minecraft.src.ICamera` (`net/minecraft/src/ICamera.java`) | **Líneas de código**: 13
- **Declaración**: `interface ICamera`
- **Rol**: Interfaz para comprobaciones de visibilidad geométrica en la cámara.
- **Descripción Técnica**: Declara métodos isBoundingBoxInFrustum() para validar si un objeto debe dibujarse.
- **Guía de Modificación**: Permite sustituir o extender el sistema de cámara.

##### <a id="class-imagebuffer"></a>[`ImageBuffer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ImageBuffer.java)
- **Ruta y Paquete**: `net.minecraft.src.ImageBuffer` (`net/minecraft/src/ImageBuffer.java`) | **Líneas de código**: 12
- **Declaración**: `interface ImageBuffer`
- **Rol**: Interfaz para procesamiento de búferes de imágenes descargadas.
- **Descripción Técnica**: Declara el método parseUserSkin para manipular los píxeles de una skin antes de subirla a la GPU.
- **Guía de Modificación**: Permite implementar filtros gráficos a las texturas de los jugadores.

##### <a id="class-imagebufferdownload"></a>[`ImageBufferDownload`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ImageBufferDownload.java)
- **Ruta y Paquete**: `net.minecraft.src.ImageBufferDownload` (`net/minecraft/src/ImageBufferDownload.java`) | **Líneas de código**: 118
- **Declaración**: `class ImageBufferDownload implements ImageBuffer
`
- **Rol**: Procesador de skins descargadas de internet.
- **Descripción Técnica**: Convierte la capa superior de la cabeza del skin en transparente si es necesario, replicando el formato de skins de Notch.
- **Guía de Modificación**: Modificar si se implementan skins en alta resolución o con capas adicionales.
- **Métodos Clave Destacados**:
  - `BufferedImage func_883_a(BufferedImage bufferedimage)`

##### <a id="class-isoimagebuffer"></a>[`IsoImageBuffer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IsoImageBuffer.java)
- **Ruta y Paquete**: `net.minecraft.src.IsoImageBuffer` (`net/minecraft/src/IsoImageBuffer.java`) | **Líneas de código**: 44
- **Declaración**: `class IsoImageBuffer`
- **Rol**: Búfer de píxeles para almacenar fragmentos del mapa isométrico generado.
- **Descripción Técnica**: Mantiene en memoria los bloques renderizados en la vista ortogonal.
- **Guía de Modificación**: Útil para exportar mapas del mundo a imágenes de alta resolución.
- **Métodos Clave Destacados**:
  - `void func_889_a(int i, int j)`
  - `void func_888_a(World world, int i, int j)`
- **Campos Clave Destacados**:
  - `BufferedImage field_1348_a`
  - `World worldObj`
  - `int field_1354_c`
  - `int field_1353_d`
  - `boolean field_1352_e`
  - `boolean field_1351_f`

##### <a id="class-loadingscreenrenderer"></a>[`LoadingScreenRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/LoadingScreenRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.LoadingScreenRenderer` (`net/minecraft/src/LoadingScreenRenderer.java`) | **Líneas de código**: 160
- **Declaración**: `class LoadingScreenRenderer implements IProgressUpdate
`
- **Rol**: Renderizador de pantallas de carga y progreso.
- **Descripción Técnica**: Dibuja el fondo de piedra, el texto de estado y la barra de porcentaje durante la generación de terreno o guardado.
- **Guía de Modificación**: Permite personalizar la pantalla de carga.
- **Métodos Clave Destacados**:
  - `void func_596_a(String s)`
  - `void func_594_b(String s)`
  - `void func_597_c(String s)`
  - `void displayLoadingString(String s)`
  - `void setLoadingProgress(int i)`

##### <a id="class-logoeffectrandomizer"></a>[`LogoEffectRandomizer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/LogoEffectRandomizer.java)
- **Ruta y Paquete**: `net.minecraft.src.LogoEffectRandomizer` (`net/minecraft/src/LogoEffectRandomizer.java`) | **Líneas de código**: 37
- **Declaración**: `class LogoEffectRandomizer`
- **Rol**: Efecto de perturbación y dispersión procedural para el logo de Mojang.
- **Descripción Técnica**: Aplica una animación de desmoronamiento de píxeles al logotipo inicial.
- **Guía de Modificación**: Permite crear efectos de introducción personalizados.
- **Métodos Clave Destacados**:
  - `void func_875_a()`
- **Campos Clave Destacados**:
  - `double field_1312_a`
  - `double field_1311_b`
  - `double field_1314_c`

##### <a id="class-modelbase"></a>[`ModelBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelBase.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelBase` (`net/minecraft/src/ModelBase.java`) | **Líneas de código**: 25
- **Declaración**: `class ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelBase'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelBase'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `float field_1244_k`
  - `boolean field_1243_l`

##### <a id="class-modelbiped"></a>[`ModelBiped`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelBiped.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelBiped` (`net/minecraft/src/ModelBiped.java`) | **Líneas de código**: 153
- **Declaración**: `class ModelBiped extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelBiped'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelBiped'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `ModelRenderer bipedHead`
  - `ModelRenderer field_1285_b`
  - `ModelRenderer field_1284_c`
  - `ModelRenderer bipedRightArm`
  - `ModelRenderer bipedLeftArm`
  - `ModelRenderer bipedRightLeg`

##### <a id="class-modelboat"></a>[`ModelBoat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelBoat.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelBoat` (`net/minecraft/src/ModelBoat.java`) | **Líneas de código**: 52
- **Declaración**: `class ModelBoat extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelBoat'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelBoat'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-modelchicken"></a>[`ModelChicken`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelChicken.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelChicken` (`net/minecraft/src/ModelChicken.java`) | **Líneas de código**: 75
- **Declaración**: `class ModelChicken extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelChicken'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelChicken'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `ModelRenderer field_1289_a`
  - `ModelRenderer field_1288_b`
  - `ModelRenderer field_1295_c`
  - `ModelRenderer field_1294_d`
  - `ModelRenderer field_1293_e`
  - `ModelRenderer field_1292_f`

##### <a id="class-modelcow"></a>[`ModelCow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelCow.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelCow` (`net/minecraft/src/ModelCow.java`) | **Líneas de código**: 59
- **Declaración**: `class ModelCow extends ModelQuadraped`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelCow'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelCow'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-modelcreeper"></a>[`ModelCreeper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelCreeper.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelCreeper` (`net/minecraft/src/ModelCreeper.java`) | **Líneas de código**: 65
- **Declaración**: `class ModelCreeper extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelCreeper'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelCreeper'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `ModelRenderer head`
  - `ModelRenderer unusedModel`
  - `ModelRenderer body`
  - `ModelRenderer leg1`
  - `ModelRenderer leg2`
  - `ModelRenderer leg3`

##### <a id="class-modelghast"></a>[`ModelGhast`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelGhast.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelGhast` (`net/minecraft/src/ModelGhast.java`) | **Líneas de código**: 55
- **Declaración**: `class ModelGhast extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelGhast'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelGhast'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-modelminecart"></a>[`ModelMinecart`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelMinecart.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelMinecart` (`net/minecraft/src/ModelMinecart.java`) | **Líneas de código**: 57
- **Declaración**: `class ModelMinecart extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelMinecart'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelMinecart'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-modelpig"></a>[`ModelPig`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelPig.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelPig` (`net/minecraft/src/ModelPig.java`) | **Líneas de código**: 19
- **Declaración**: `class ModelPig extends ModelQuadraped`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelPig'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelPig'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.

##### <a id="class-modelquadraped"></a>[`ModelQuadraped`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelQuadraped.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelQuadraped` (`net/minecraft/src/ModelQuadraped.java`) | **Líneas de código**: 60
- **Declaración**: `class ModelQuadraped extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelQuadraped'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelQuadraped'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `ModelRenderer head`
  - `ModelRenderer body`
  - `ModelRenderer leg1`
  - `ModelRenderer leg2`
  - `ModelRenderer leg3`
  - `ModelRenderer leg4`

##### <a id="class-modelrenderer"></a>[`ModelRenderer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelRenderer.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelRenderer` (`net/minecraft/src/ModelRenderer.java`) | **Líneas de código**: 206
- **Declaración**: `class ModelRenderer`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelRenderer'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelRenderer'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void func_921_a(float f, float f1, float f2, int i, int j, int k)`
  - `void addBox(float f, float f1, float f2, int i, int j, int k, float f3)`
  - `void setPosition(float f, float f1, float f2)`
  - `void render(float f)`
  - `void func_926_b(float f)`
- **Campos Clave Destacados**:
  - `float offsetX`
  - `float offsetY`
  - `float offsetZ`
  - `float rotateAngleX`
  - `float rotateAngleY`
  - `float rotateAngleZ`

##### <a id="class-modelsheep1"></a>[`ModelSheep1`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelSheep1.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelSheep1` (`net/minecraft/src/ModelSheep1.java`) | **Líneas de código**: 33
- **Declaración**: `class ModelSheep1 extends ModelQuadraped`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelSheep1'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelSheep1'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.

##### <a id="class-modelsheep2"></a>[`ModelSheep2`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelSheep2.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelSheep2` (`net/minecraft/src/ModelSheep2.java`) | **Líneas de código**: 20
- **Declaración**: `class ModelSheep2 extends ModelQuadraped`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelSheep2'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelSheep2'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.

##### <a id="class-modelskeleton"></a>[`ModelSkeleton`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelSkeleton.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelSkeleton` (`net/minecraft/src/ModelSkeleton.java`) | **Líneas de código**: 28
- **Declaración**: `class ModelSkeleton extends ModelZombie`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelSkeleton'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelSkeleton'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.

##### <a id="class-modelslime"></a>[`ModelSlime`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelSlime.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelSlime` (`net/minecraft/src/ModelSlime.java`) | **Líneas de código**: 47
- **Declaración**: `class ModelSlime extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelSlime'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelSlime'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-modelspider"></a>[`ModelSpider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelSpider.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelSpider` (`net/minecraft/src/ModelSpider.java`) | **Líneas de código**: 125
- **Declaración**: `class ModelSpider extends ModelBase`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelSpider'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelSpider'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void render(float f, float f1, float f2, float f3, float f4, float f5)`
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`
- **Campos Clave Destacados**:
  - `ModelRenderer field_1255_a`
  - `ModelRenderer field_1254_b`
  - `ModelRenderer field_1253_c`
  - `ModelRenderer field_1252_d`
  - `ModelRenderer field_1251_e`
  - `ModelRenderer field_1250_f`

##### <a id="class-modelzombie"></a>[`ModelZombie`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ModelZombie.java)
- **Ruta y Paquete**: `net.minecraft.src.ModelZombie` (`net/minecraft/src/ModelZombie.java`) | **Líneas de código**: 32
- **Declaración**: `class ModelZombie extends ModelBiped`
- **Rol**: Modelo geométrico cúbico 3D: 'ModelZombie'.
- **Descripción Técnica**: Define las partes corporales, cajas cúbicas y ángulos de rotación de animación para 'ModelZombie'.
- **Guía de Modificación**: Modifica setRotationAngles() para alterar animaciones de movimiento o posturas.
- **Métodos Clave Destacados**:
  - `void setRotationAngles(float f, float f1, float f2, float f3, float f4, float f5)`

##### <a id="class-openglcapschecker"></a>[`OpenGlCapsChecker`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/OpenGlCapsChecker.java)
- **Ruta y Paquete**: `net.minecraft.src.OpenGlCapsChecker` (`net/minecraft/src/OpenGlCapsChecker.java`) | **Líneas de código**: 23
- **Declaración**: `class OpenGlCapsChecker`
- **Rol**: Verificador de capacidades y extensiones de la tarjeta gráfica.
- **Descripción Técnica**: Comprueba compatibilidad con OpenGL 1.2+, soporte de multitextura y capacidades de sombreado.
- **Guía de Modificación**: Permite habilitar o deshabilitar efectos según la GPU del usuario.
- **Métodos Clave Destacados**:
  - `boolean checkARBOcclusion()`

##### <a id="class-positiontexurevertex"></a>[`PositionTexureVertex`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/PositionTexureVertex.java)
- **Ruta y Paquete**: `net.minecraft.src.PositionTexureVertex` (`net/minecraft/src/PositionTexureVertex.java`) | **Líneas de código**: 37
- **Declaración**: `class PositionTexureVertex`
- **Rol**: Vértice espacial 3D con coordenadas UV de mapeado de textura.
- **Descripción Técnica**: Almacena una posición vectorial Vec3D y coordenadas (texturePositionX, texturePositionY) para renderizado de modelos.
- **Guía de Modificación**: Utilizado en la construcción de piezas de modelos cúbicos en ModelRenderer.
- **Métodos Clave Destacados**:
  - `PositionTexureVertex setTexturePosition(float f, float f1)`
- **Campos Clave Destacados**:
  - `Vec3D vector3D`
  - `float texturePositionX`
  - `float texturePositionY`

##### <a id="class-render"></a>[`Render`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Render.java)
- **Ruta y Paquete**: `net.minecraft.src.Render` (`net/minecraft/src/Render.java`) | **Líneas de código**: 267
- **Declaración**: `class Render`
- **Rol**: Renderizador en espacio 3D para 'Render'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void loadTexture(String s)`
  - `void func_140_a(String s, String s1)`
  - `void renderOffsetAABB(AxisAlignedBB axisalignedbb, double d, double d1, double d2)`
  - `void renderAABB(AxisAlignedBB axisalignedbb)`
  - `void setRenderManager(RenderManager rendermanager)`
  - `void doRenderShadowAndFire(Entity entity, double d, double d1, double d2, float f, float f1)`
- **Campos Clave Destacados**:
  - `RenderManager renderManager`
  - `float field_9246_c`
  - `float field_194_c`

##### <a id="class-renderarrow"></a>[`RenderArrow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderArrow.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderArrow` (`net/minecraft/src/RenderArrow.java`) | **Líneas de código**: 79
- **Declaración**: `class RenderArrow extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderArrow'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_154_a(EntityArrow entityarrow, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderbiped"></a>[`RenderBiped`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderBiped.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderBiped` (`net/minecraft/src/RenderBiped.java`) | **Líneas de código**: 56
- **Declaración**: `class RenderBiped extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderBiped'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_6331_b(EntityLiving entityliving, float f)`
- **Campos Clave Destacados**:
  - `ModelBiped field_4013_a`

##### <a id="class-renderblocks"></a>[`RenderBlocks`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderBlocks.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderBlocks` (`net/minecraft/src/RenderBlocks.java`) | **Líneas de código**: 2078
- **Declaración**: `class RenderBlocks`
- **Rol**: Renderizador en espacio 3D para 'RenderBlocks'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void renderBlockUsingTexture(Block block, int i, int j, int k, int l)`
  - `boolean renderBlockByRenderType(Block block, int i, int j, int k)`
  - `boolean renderBlockTorch(Block block, int i, int j, int k)`
  - `boolean renderBlockLever(Block block, int i, int j, int k)`
  - `boolean renderBlockFire(Block block, int i, int j, int k)`
  - `boolean renderBlockRedstoneWire(Block block, int i, int j, int k)`

##### <a id="class-renderboat"></a>[`RenderBoat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderBoat.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderBoat` (`net/minecraft/src/RenderBoat.java`) | **Líneas de código**: 50
- **Declaración**: `class RenderBoat extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderBoat'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_157_a(EntityBoat entityboat, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`
- **Campos Clave Destacados**:
  - `ModelBase field_198_d`

##### <a id="class-renderchicken"></a>[`RenderChicken`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderChicken.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderChicken` (`net/minecraft/src/RenderChicken.java`) | **Líneas de código**: 44
- **Declaración**: `class RenderChicken extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderChicken'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_181_a(EntityChicken entitychicken, double d, double d1, double d2, float f, float f1)`
  - `float func_182_a(EntityChicken entitychicken, float f)`
  - `float func_170_d(EntityLiving entityliving, float f)`
  - `void func_171_a(EntityLiving entityliving, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-rendercow"></a>[`RenderCow`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderCow.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderCow` (`net/minecraft/src/RenderCow.java`) | **Líneas de código**: 32
- **Declaración**: `class RenderCow extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderCow'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_177_a(EntityCow entitycow, double d, double d1, double d2, float f, float f1)`
  - `void func_171_a(EntityLiving entityliving, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-rendercreeper"></a>[`RenderCreeper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderCreeper.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderCreeper` (`net/minecraft/src/RenderCreeper.java`) | **Líneas de código**: 68
- **Declaración**: `class RenderCreeper extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderCreeper'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_184_a(EntityCreeper entitycreeper, float f)`
  - `int func_183_a(EntityCreeper entitycreeper, float f, float f1)`
  - `void func_6330_a(EntityLiving entityliving, float f)`
  - `int func_173_a(EntityLiving entityliving, float f, float f1)`

##### <a id="class-renderengine"></a>[`RenderEngine`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderEngine.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderEngine` (`net/minecraft/src/RenderEngine.java`) | **Líneas de código**: 469
- **Declaración**: `class RenderEngine`
- **Rol**: Renderizador en espacio 3D para 'RenderEngine'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `int getTexture(String s)`
  - `int allocateAndSetupTexture(BufferedImage bufferedimage)`
  - `void setupTexture(BufferedImage bufferedimage, int i)`
  - `void deleteTexture(int i)`
  - `int getTextureForDownloadableImage(String s, String s1)`
  - `ThreadDownloadImageData obtainImageData(String s, ImageBuffer imagebuffer)`
- **Campos Clave Destacados**:
  - `boolean useMipmaps`

##### <a id="class-renderentity"></a>[`RenderEntity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderEntity.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderEntity` (`net/minecraft/src/RenderEntity.java`) | **Líneas de código**: 22
- **Declaración**: `class RenderEntity extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderEntity'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderfallingsand"></a>[`RenderFallingSand`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderFallingSand.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderFallingSand` (`net/minecraft/src/RenderFallingSand.java`) | **Líneas de código**: 38
- **Declaración**: `class RenderFallingSand extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderFallingSand'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_156_a(EntityFallingSand entityfallingsand, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderfireball"></a>[`RenderFireball`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderFireball.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderFireball` (`net/minecraft/src/RenderFireball.java`) | **Líneas de código**: 51
- **Declaración**: `class RenderFireball extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderFireball'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_4012_a(EntityFireball entityfireball, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderfish"></a>[`RenderFish`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderFish.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderFish` (`net/minecraft/src/RenderFish.java`) | **Líneas de código**: 92
- **Declaración**: `class RenderFish extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderFish'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_4011_a(EntityFish entityfish, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderghast"></a>[`RenderGhast`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderGhast.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderGhast` (`net/minecraft/src/RenderGhast.java`) | **Líneas de código**: 35
- **Declaración**: `class RenderGhast extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderGhast'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_4014_a(EntityGhast entityghast, float f)`
  - `void func_6330_a(EntityLiving entityliving, float f)`

##### <a id="class-renderglobal"></a>[`RenderGlobal`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderGlobal.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderGlobal` (`net/minecraft/src/RenderGlobal.java`) | **Líneas de código**: 1286
- **Declaración**: `class RenderGlobal implements IWorldAccess
`
- **Rol**: Renderizador en espacio 3D para 'RenderGlobal'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_946_a(World world)`
  - `void func_958_a()`
  - `void func_951_a(Vec3D vec3d, ICamera icamera, float f)`
  - `String func_953_b()`
  - `String func_957_c()`
  - `int func_943_a(EntityPlayer entityplayer, int i, double d)`
- **Campos Clave Destacados**:
  - `List field_1458_a`
  - `float field_1450_i`

##### <a id="class-renderhelper"></a>[`RenderHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderHelper` (`net/minecraft/src/RenderHelper.java`) | **Líneas de código**: 63
- **Declaración**: `class RenderHelper`
- **Rol**: Renderizador en espacio 3D para 'RenderHelper'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void disableStandardItemLighting()`
  - `void enableStandardItemLighting()`

##### <a id="class-renderitem"></a>[`RenderItem`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderItem.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderItem` (`net/minecraft/src/RenderItem.java`) | **Líneas de código**: 218
- **Declaración**: `class RenderItem extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderItem'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_165_a(EntityItem entityitem, double d, double d1, double d2, float f, float f1)`
  - `void renderItemIntoGUI(FontRenderer fontrenderer, RenderEngine renderengine, ItemStack itemstack, int i, int j)`
  - `void renderItemOverlayIntoGUI(FontRenderer fontrenderer, RenderEngine renderengine, ItemStack itemstack, int i, int j)`
  - `void renderTexturedQuad(int i, int j, int k, int l, int i1, int j1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderlist"></a>[`RenderList`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderList.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderList` (`net/minecraft/src/RenderList.java`) | **Líneas de código**: 87
- **Declaración**: `class RenderList`
- **Rol**: Renderizador en espacio 3D para 'RenderList'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_861_a(int i, int j, int k, double d, double d1, double d2)`
  - `boolean func_862_a(int i, int j, int k)`
  - `void func_858_a(int i)`
  - `void func_860_a()`
  - `void func_859_b()`

##### <a id="class-renderliving"></a>[`RenderLiving`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderLiving.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderLiving` (`net/minecraft/src/RenderLiving.java`) | **Líneas de código**: 173
- **Declaración**: `class RenderLiving extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderLiving'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_4013_a(ModelBase modelbase)`
  - `void func_171_a(EntityLiving entityliving, double d, double d1, double d2, float f, float f1)`
  - `float func_167_c(EntityLiving entityliving, float f)`
  - `float func_170_d(EntityLiving entityliving, float f)`
  - `void func_6331_b(EntityLiving entityliving, float f)`
  - `boolean func_166_a(EntityLiving entityliving, int i)`
- **Campos Clave Destacados**:
  - `ModelBase unusedRenderBlocks`
  - `ModelBase field_6332_f`

##### <a id="class-rendermanager"></a>[`RenderManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderManager.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderManager` (`net/minecraft/src/RenderManager.java`) | **Líneas de código**: 134
- **Declaración**: `class RenderManager`
- **Rol**: Renderizador en espacio 3D para 'RenderManager'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `Render func_4117_a(Class class1)`
  - `Render func_855_a(Entity entity)`
  - `void func_857_a(World world, RenderEngine renderengine, FontRenderer fontrenderer, EntityPlayer entityplayer, GameSettings gamesettings, float f)`
  - `void func_854_a(Entity entity, float f)`
  - `void func_853_a(Entity entity, double d, double d1, double d2, float f, float f1)`
  - `void func_852_a(World world)`
- **Campos Clave Destacados**:
  - `RenderManager instance`
  - `double field_1232_b`
  - `double field_1231_c`
  - `double field_1230_d`
  - `RenderEngine renderEngine`
  - `ItemRenderer field_4236_f`

##### <a id="class-renderminecart"></a>[`RenderMinecart`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderMinecart.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderMinecart` (`net/minecraft/src/RenderMinecart.java`) | **Líneas de código**: 95
- **Declaración**: `class RenderMinecart extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderMinecart'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_152_a(EntityMinecart entityminecart, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`
- **Campos Clave Destacados**:
  - `ModelBase unusedModelBiped`

##### <a id="class-renderpainting"></a>[`RenderPainting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderPainting.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderPainting` (`net/minecraft/src/RenderPainting.java`) | **Líneas de código**: 136
- **Declaración**: `class RenderPainting extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderPainting'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_158_a(EntityPainting entitypainting, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderpig"></a>[`RenderPig`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderPig.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderPig` (`net/minecraft/src/RenderPig.java`) | **Líneas de código**: 26
- **Declaración**: `class RenderPig extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderPig'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `boolean func_180_a(EntityPig entitypig, int i)`
  - `boolean func_166_a(EntityLiving entityliving, int i)`

##### <a id="class-renderplayer"></a>[`RenderPlayer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderPlayer.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderPlayer` (`net/minecraft/src/RenderPlayer.java`) | **Líneas de código**: 235
- **Declaración**: `class RenderPlayer extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderPlayer'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `boolean func_187_a(EntityPlayer entityplayer, int i)`
  - `void func_188_a(EntityPlayer entityplayer, double d, double d1, double d2, float f, float f1)`
  - `void func_4015_a(EntityPlayer entityplayer, float f)`
  - `void func_186_b(EntityPlayer entityplayer, float f)`
  - `void drawFirstPersonHand()`
  - `void func_6330_a(EntityLiving entityliving, float f)`

##### <a id="class-rendersheep"></a>[`RenderSheep`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderSheep.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderSheep` (`net/minecraft/src/RenderSheep.java`) | **Líneas de código**: 26
- **Declaración**: `class RenderSheep extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderSheep'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `boolean func_176_a(EntitySheep entitysheep, int i)`
  - `boolean func_166_a(EntityLiving entityliving, int i)`

##### <a id="class-renderslime"></a>[`RenderSlime`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderSlime.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderSlime` (`net/minecraft/src/RenderSlime.java`) | **Líneas de código**: 54
- **Declaración**: `class RenderSlime extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderSlime'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `boolean func_179_a(EntitySlime entityslime, int i)`
  - `void func_178_a(EntitySlime entityslime, float f)`
  - `void func_6330_a(EntityLiving entityliving, float f)`
  - `boolean func_166_a(EntityLiving entityliving, int i)`

##### <a id="class-rendersnowball"></a>[`RenderSnowball`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderSnowball.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderSnowball` (`net/minecraft/src/RenderSnowball.java`) | **Líneas de código**: 50
- **Declaración**: `class RenderSnowball extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderSnowball'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_155_a(EntitySnowball entitysnowball, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-rendersorter"></a>[`RenderSorter`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderSorter.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderSorter` (`net/minecraft/src/RenderSorter.java`) | **Líneas de código**: 50
- **Declaración**: `class RenderSorter implements Comparator
`
- **Rol**: Renderizador en espacio 3D para 'RenderSorter'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `int func_993_a(WorldRenderer worldrenderer, WorldRenderer worldrenderer1)`
  - `int compare(Object obj, Object obj1)`

##### <a id="class-renderspider"></a>[`RenderSpider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderSpider.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderSpider` (`net/minecraft/src/RenderSpider.java`) | **Líneas de código**: 52
- **Declaración**: `class RenderSpider extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderSpider'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `float func_191_a(EntitySpider entityspider)`
  - `boolean func_190_a(EntitySpider entityspider, int i)`
  - `float func_172_a(EntityLiving entityliving)`
  - `boolean func_166_a(EntityLiving entityliving, int i)`

##### <a id="class-rendertntprimed"></a>[`RenderTNTPrimed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderTNTPrimed.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderTNTPrimed` (`net/minecraft/src/RenderTNTPrimed.java`) | **Líneas de código**: 64
- **Declaración**: `class RenderTNTPrimed extends Render`
- **Rol**: Renderizador en espacio 3D para 'RenderTNTPrimed'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_153_a(EntityTNTPrimed entitytntprimed, double d, double d1, double d2, float f, float f1)`
  - `void doRender(Entity entity, double d, double d1, double d2, float f, float f1)`

##### <a id="class-renderzombiesimple"></a>[`RenderZombieSimple`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/RenderZombieSimple.java)
- **Ruta y Paquete**: `net.minecraft.src.RenderZombieSimple` (`net/minecraft/src/RenderZombieSimple.java`) | **Líneas de código**: 28
- **Declaración**: `class RenderZombieSimple extends RenderLiving`
- **Rol**: Renderizador en espacio 3D para 'RenderZombieSimple'.
- **Descripción Técnica**: Dibuja la entidad o elemento en pantalla conectando su textura con el modelo 3D correspondiente.
- **Guía de Modificación**: Sobrescribe doRender() para añadir efectos visuales o cambiar la textura.
- **Métodos Clave Destacados**:
  - `void func_175_a(EntityZombieSimple entityzombiesimple, float f)`
  - `void func_6330_a(EntityLiving entityliving, float f)`

##### <a id="class-scaledresolution"></a>[`ScaledResolution`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ScaledResolution.java)
- **Ruta y Paquete**: `net.minecraft.src.ScaledResolution` (`net/minecraft/src/ScaledResolution.java`) | **Líneas de código**: 32
- **Declaración**: `class ScaledResolution`
- **Rol**: Calculador dinámico del factor de escala de la interfaz de usuario (GUI Scale).
- **Descripción Técnica**: Calcula la resolución lógica (scaledWidth, scaledHeight) y el factor de escala según el tamaño de la ventana.
- **Guía de Modificación**: Imprescindible para que los menús y HUD se adapten correctamente a cualquier resolución de pantalla.
- **Métodos Clave Destacados**:
  - `int getScaledWidth()`
  - `int getScaledHeight()`
- **Campos Clave Destacados**:
  - `int scaleFactor`

##### <a id="class-signmodel"></a>[`SignModel`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SignModel.java)
- **Ruta y Paquete**: `net.minecraft.src.SignModel` (`net/minecraft/src/SignModel.java`) | **Líneas de código**: 26
- **Declaración**: `class SignModel`
- **Rol**: Modelo geométrico 3D del cartel (tablero y poste de soporte).
- **Descripción Técnica**: Renderizado por TileEntitySignRenderer para dar volumen al cartel en el espacio.
- **Guía de Modificación**: Permite alterar la forma física del cartel.
- **Métodos Clave Destacados**:
  - `void func_887_a()`
- **Campos Clave Destacados**:
  - `ModelRenderer field_1346_a`
  - `ModelRenderer field_1345_b`

##### <a id="class-terraintexturemanager"></a>[`TerrainTextureManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TerrainTextureManager.java)
- **Ruta y Paquete**: `net.minecraft.src.TerrainTextureManager` (`net/minecraft/src/TerrainTextureManager.java`) | **Líneas de código**: 268
- **Declaración**: `class TerrainTextureManager`
- **Rol**: Administrador de atlas y texturas del terreno.
- **Descripción Técnica**: Supervisa la asignación y carga de mosaicos en terrain.png.
- **Guía de Modificación**: Utilizado en la configuración del atlas de bloques.
- **Métodos Clave Destacados**:
  - `void func_799_a(IsoImageBuffer isoimagebuffer)`

##### <a id="class-tessellator"></a>[`Tessellator`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Tessellator.java)
- **Ruta y Paquete**: `net.minecraft.src.Tessellator` (`net/minecraft/src/Tessellator.java`) | **Líneas de código**: 355
- **Declaración**: `class Tessellator`
- **Rol**: Motor de emisión geométrica de quads de alto rendimiento con búferes nativos.
- **Descripción Técnica**: La clase gráfica más crítica del cliente. Acumula vértices, normales, UVs y colores en un ByteBuffer nativo, volcando miles de polígonos a la GPU con una sola llamada glDrawArrays.
- **Guía de Modificación**: Cualquier dibujo de geometría personalizada en el mundo debe utilizar Tessellator.instance para máximo rendimiento.
- **Métodos Clave Destacados**:
  - `void draw()`
  - `void startDrawingQuads()`
  - `void startDrawing(int i)`
  - `void setTextureUV(double d, double d1)`
  - `void setColorOpaque_F(float f, float f1, float f2)`
  - `void setColorRGBA_F(float f, float f1, float f2, float f3)`
- **Campos Clave Destacados**:
  - `Tessellator instance`

##### <a id="class-texturecompassfx"></a>[`TextureCompassFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureCompassFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureCompassFX` (`net/minecraft/src/TextureCompassFX.java`) | **Líneas de código**: 137
- **Declaración**: `class TextureCompassFX extends TextureFX`
- **Rol**: Animación procedural para la esfera de la brújula en el inventario.
- **Descripción Técnica**: Calcula el ángulo hacia el punto de spawn del mundo respecto a la orientación del jugador y rota la aguja suavemente con inercia.
- **Guía de Modificación**: Modificar para que la brújula apunte a coordenadas personalizadas o bases de jugadores.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturefx"></a>[`TextureFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureFX` (`net/minecraft/src/TextureFX.java`) | **Líneas de código**: 43
- **Declaración**: `class TextureFX`
- **Rol**: Clase base para animaciones procedimentales de textura en memoria.
- **Descripción Técnica**: Almacena un búfer de píxeles (imageData) que se actualiza en cada tick antes de enviarse a OpenGL mediante glTexSubImage2D.
- **Guía de Modificación**: Heredar de esta clase para crear texturas animadas personalizadas (fluidos, energía, pantallas).
- **Métodos Clave Destacados**:
  - `void func_783_a()`
  - `void func_782_a(RenderEngine renderengine)`
- **Campos Clave Destacados**:
  - `int field_1126_b`
  - `boolean field_1131_c`
  - `int field_1130_d`
  - `int field_1129_e`
  - `int field_1128_f`

##### <a id="class-textureflamesfx"></a>[`TextureFlamesFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureFlamesFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureFlamesFX` (`net/minecraft/src/TextureFlamesFX.java`) | **Líneas de código**: 92
- **Declaración**: `class TextureFlamesFX extends TextureFX`
- **Rol**: Animación procedimental para las partículas y texturas de fuego.
- **Descripción Técnica**: Simula llamas ascendentes con degradados térmicos dinámicos.
- **Guía de Modificación**: Modificar para crear fuego azul, verde o mágico.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturelavafx"></a>[`TextureLavaFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureLavaFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureLavaFX` (`net/minecraft/src/TextureLavaFX.java`) | **Líneas de código**: 93
- **Declaración**: `class TextureLavaFX extends TextureFX`
- **Rol**: Animación procedimental para el bloque de lava estacionaria.
- **Descripción Técnica**: Simula burbujeo y corrientes incandescentes con transiciones de color naranja y rojo.
- **Guía de Modificación**: Modificar para personalizar la apariencia de fluidos ígneos.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturelavaflowfx"></a>[`TextureLavaFlowFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureLavaFlowFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureLavaFlowFX` (`net/minecraft/src/TextureLavaFlowFX.java`) | **Líneas de código**: 97
- **Declaración**: `class TextureLavaFlowFX extends TextureFX`
- **Rol**: Animación procedimental para la lava en flujo continuo.
- **Descripción Técnica**: Calcula el desplazamiento visual de la corriente de lava en movimiento horizontal y vertical.
- **Guía de Modificación**: Complementa TextureLavaFX.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturepackbase"></a>[`TexturePackBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturePackBase.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturePackBase` (`net/minecraft/src/TexturePackBase.java`) | **Líneas de código**: 45
- **Declaración**: `class TexturePackBase`
- **Rol**: Clase base abstracta para paquetes de texturas (Texture Packs).
- **Descripción Técnica**: Declara métodos para abrir streams de recursos gráficos, leer la descripción del paquete y el icono pack.png.
- **Guía de Modificación**: Base del sistema de personalización gráfica del juego.
- **Métodos Clave Destacados**:
  - `void func_6482_a()`
  - `void closeTexturePackFile()`
  - `void func_6485_a(Minecraft minecraft)`
  - `void func_6484_b(Minecraft minecraft)`
  - `void func_6483_c(Minecraft minecraft)`
  - `InputStream func_6481_a(String s)`
- **Campos Clave Destacados**:
  - `String texturePackFileName`
  - `String firstDescriptionLine`
  - `String secondDescriptionLine`
  - `String field_6488_d`

##### <a id="class-texturepackcustom"></a>[`TexturePackCustom`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturePackCustom.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturePackCustom` (`net/minecraft/src/TexturePackCustom.java`) | **Líneas de código**: 138
- **Declaración**: `class TexturePackCustom extends TexturePackBase`
- **Rol**: Paquete de texturas personalizado cargado desde un archivo .zip en la carpeta texturepacks.
- **Descripción Técnica**: Extrae imágenes, sonidos y fuentes desde el archivo comprimido sin descomprimirlo en disco.
- **Guía de Modificación**: Permite a los usuarios sustituir los gráficos del juego.
- **Métodos Clave Destacados**:
  - `void func_6485_a(Minecraft minecraft)`
  - `void func_6484_b(Minecraft minecraft)`
  - `void func_6483_c(Minecraft minecraft)`
  - `void func_6482_a()`
  - `void closeTexturePackFile()`
  - `InputStream func_6481_a(String s)`

##### <a id="class-texturepackdefault"></a>[`TexturePackDefault`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturePackDefault.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturePackDefault` (`net/minecraft/src/TexturePackDefault.java`) | **Líneas de código**: 55
- **Declaración**: `class TexturePackDefault extends TexturePackBase`
- **Rol**: Paquete de texturas por defecto integrado en el cliente.
- **Descripción Técnica**: Carga los recursos originales directamente desde el classpath del juego.
- **Guía de Modificación**: Paquete de respaldo permanente ante archivos faltantes.
- **Métodos Clave Destacados**:
  - `void func_6484_b(Minecraft minecraft)`
  - `void func_6483_c(Minecraft minecraft)`

##### <a id="class-texturepacklist"></a>[`TexturePackList`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturePackList.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturePackList` (`net/minecraft/src/TexturePackList.java`) | **Líneas de código**: 115
- **Declaración**: `class TexturePackList`
- **Rol**: Gestor del catálogo de paquetes de texturas disponibles en disco.
- **Descripción Técnica**: Escanea la carpeta texturepacks, detecta archivos ZIP añadidos por el usuario y permite alternar el paquete activo.
- **Guía de Modificación**: Modificar si se requiere forzar un paquete de recursos en un modpack.
- **Métodos Clave Destacados**:
  - `boolean setTexturePack(TexturePackBase texturepackbase)`
  - `void func_6532_a()`
  - `List availableTexturePacks()`
- **Campos Clave Destacados**:
  - `TexturePackBase selectedTexturePack`

##### <a id="class-textureportalfx"></a>[`TexturePortalFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturePortalFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturePortalFX` (`net/minecraft/src/TexturePortalFX.java`) | **Líneas de código**: 100
- **Declaración**: `class TexturePortalFX extends TextureFX`
- **Rol**: Animación procedimental para la lámina del portal al Inframundo.
- **Descripción Técnica**: Genera remolinos de ruido púrpura hipnótico dentro del portal de obsidiana.
- **Guía de Modificación**: Personalizar para crear portales con efectos visuales únicos.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturewatchfx"></a>[`TextureWatchFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureWatchFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureWatchFX` (`net/minecraft/src/TextureWatchFX.java`) | **Líneas de código**: 105
- **Declaración**: `class TextureWatchFX extends TextureFX`
- **Rol**: Animación procedural para la esfera del reloj en el inventario.
- **Descripción Técnica**: Calcula la posición del sol y la luna a partir de World.worldTime y rota el dial celeste en tiempo real.
- **Guía de Modificación**: Referencia para crear diales temporales o calendarios en ítems.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturewaterfx"></a>[`TextureWaterFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TextureWaterFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TextureWaterFX` (`net/minecraft/src/TextureWaterFX.java`) | **Líneas de código**: 99
- **Declaración**: `class TextureWaterFX extends TextureFX`
- **Rol**: Animación procedimental para el bloque de agua estacionaria.
- **Descripción Técnica**: Simula ondas y perturbaciones continuas en la superficie del agua calculando fluctuaciones físicas de píxeles.
- **Guía de Modificación**: Modificar para cambiar el estilo o color del agua animada.
- **Métodos Clave Destacados**:
  - `void func_783_a()`

##### <a id="class-texturedquad"></a>[`TexturedQuad`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexturedQuad.java)
- **Ruta y Paquete**: `net.minecraft.src.TexturedQuad` (`net/minecraft/src/TexturedQuad.java`) | **Líneas de código**: 65
- **Declaración**: `class TexturedQuad`
- **Rol**: Polígono cuadrilátero formado por 4 vértices PositionTexureVertex.
- **Descripción Técnica**: Dibuja una cara plana con textura mapeada en el espacio tridimensional calculando el vector normal para iluminación.
- **Guía de Modificación**: El elemento constructivo básico de las cajas en ModelRenderer.
- **Métodos Clave Destacados**:
  - `void func_809_a()`
  - `void func_808_a(Tessellator tessellator, float f)`
- **Campos Clave Destacados**:
  - `int field_1194_b`

##### <a id="class-texurewaterflowfx"></a>[`TexureWaterFlowFX`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/TexureWaterFlowFX.java)
- **Ruta y Paquete**: `net.minecraft.src.TexureWaterFlowFX` (`net/minecraft/src/TexureWaterFlowFX.java`) | **Líneas de código**: 100
- **Declaración**: `class TexureWaterFlowFX extends TextureFX`
- **Rol**: Animación procedimental para el flujo horizontal del agua en movimiento.
- **Descripción Técnica**: Modifica píxel a píxel el búfer de imagen para simular olas y corrientes de agua animadas.
- **Guía de Modificación**: Permite personalizar la apariencia de fluidos en movimiento.
- **Métodos Clave Destacados**:
  - `void func_783_a()`


### 09. Interfaz de Usuario y Menús (GUI, HUD & Screens)
> **Resumen del subsistema**: El sistema de interfaz 2D. Maneja el HUD del juego (vida, oxígeno, hotbar, mira), menús principales, menús de inventario, cofres y hornos, pantallas de opciones y controles, pantalla de muerte, selección de mundos y chat.

**Total de clases**: 30

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`ChatLine`](#class-chatline) | `class` | `-` | 18 | Representación de una línea individual de texto en el historial d... |
| [`Gui`](#class-gui) | `class` | `-` | 91 | Pantalla o elemento de interfaz gráfica: 'Gui'. |
| [`GuiButton`](#class-guibutton) | `class` | `ext Gui` | 93 | Pantalla o elemento de interfaz gráfica: 'GuiButton'. |
| [`GuiChat`](#class-guichat) | `class` | `ext GuiScreen` | 87 | Pantalla o elemento de interfaz gráfica: 'GuiChat'. |
| [`GuiChest`](#class-guichest) | `class` | `ext GuiContainer` | 69 | Pantalla del inventario de cofre simple (27 slots) o doble (54 sl... |
| [`GuiConflictWarning`](#class-guiconflictwarning) | `class` | `ext GuiScreen` | 52 | Pantalla o elemento de interfaz gráfica: 'GuiConflictWarning'. |
| [`GuiConnectFailed`](#class-guiconnectfailed) | `class` | `ext GuiScreen` | 50 | Pantalla o elemento de interfaz gráfica: 'GuiConnectFailed'. |
| [`GuiConnecting`](#class-guiconnecting) | `class` | `ext GuiScreen` | 82 | Pantalla o elemento de interfaz gráfica: 'GuiConnecting'. |
| [`GuiContainer`](#class-guicontainer) | `class` | `ext GuiScreen` | 278 | Pantalla o elemento de interfaz gráfica: 'GuiContainer'. |
| [`GuiControls`](#class-guicontrols) | `class` | `ext GuiScreen` | 71 | Pantalla o elemento de interfaz gráfica: 'GuiControls'. |
| [`GuiDeleteWorld`](#class-guideleteworld) | `class` | `ext GuiSelectWorld` | 41 | Pantalla o elemento de interfaz gráfica: 'GuiDeleteWorld'. |
| [`GuiDownloadTerrain`](#class-guidownloadterrain) | `class` | `ext GuiScreen` | 52 | Pantalla o elemento de interfaz gráfica: 'GuiDownloadTerrain'. |
| [`GuiEditSign`](#class-guieditsign) | `class` | `ext GuiScreen` | 119 | Pantalla o elemento de interfaz gráfica: 'GuiEditSign'. |
| [`GuiFurnace`](#class-guifurnace) | `class` | `ext GuiContainer` | 59 | Pantalla interactiva del horno de fundición. |
| [`GuiGameOver`](#class-guigameover) | `class` | `ext GuiScreen` | 62 | Pantalla o elemento de interfaz gráfica: 'GuiGameOver'. |
| [`GuiIngame`](#class-guiingame) | `class` | `ext Gui` | 394 | Pantalla o elemento de interfaz gráfica: 'GuiIngame'. |
| [`GuiIngameMenu`](#class-guiingamemenu) | `class` | `ext GuiScreen` | 76 | Pantalla o elemento de interfaz gráfica: 'GuiIngameMenu'. |
| [`GuiInventory`](#class-guiinventory) | `class` | `ext GuiContainer` | 101 | Pantalla interactiva del inventario personal del jugador (tecla E... |
| [`GuiMainMenu`](#class-guimainmenu) | `class` | `ext GuiScreen` | 251 | Pantalla del menú principal del juego. |
| [`GuiMultiplayer`](#class-guimultiplayer) | `class` | `ext GuiScreen` | 120 | Pantalla o elemento de interfaz gráfica: 'GuiMultiplayer'. |
| [`GuiOptions`](#class-guioptions) | `class` | `ext GuiScreen` | 70 | Pantalla o elemento de interfaz gráfica: 'GuiOptions'. |
| [`GuiScreen`](#class-guiscreen) | `class` | `ext Gui` | 196 | Clase base para todas las pantallas interactivas de menús y diálo... |
| [`GuiSelectWorld`](#class-guiselectworld) | `class` | `ext GuiScreen` | 98 | Pantalla o elemento de interfaz gráfica: 'GuiSelectWorld'. |
| [`GuiSlider`](#class-guislider) | `class` | `ext GuiButton` | 83 | Pantalla o elemento de interfaz gráfica: 'GuiSlider'. |
| [`GuiSmallButton`](#class-guismallbutton) | `class` | `ext GuiButton` | 14 | Pantalla o elemento de interfaz gráfica: 'GuiSmallButton'. |
| [`GuiUnused`](#class-guiunused) | `class` | `ext GuiScreen` | 28 | Pantalla o elemento de interfaz gráfica: 'GuiUnused'. |
| [`GuiYesNo`](#class-guiyesno) | `class` | `ext GuiScreen` | 42 | Pantalla o elemento de interfaz gráfica: 'GuiYesNo'. |
| [`IProgressUpdate`](#class-iprogressupdate) | `interface` | `-` | 15 | Interfaz para receptores de progreso de tareas largas. |
| [`Packet3Chat`](#class-packet3chat) | `class` | `ext Packet` | 41 | Paquete de datos de red: 'Packet3Chat' en el protocolo SMP. |
| [`ScreenShotHelper`](#class-screenshothelper) | `class` | `-` | 75 | Utilidad para capturar pantallas de juego y guardarlas en disco. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-chatline"></a>[`ChatLine`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ChatLine.java)
- **Ruta y Paquete**: `net.minecraft.src.ChatLine` (`net/minecraft/src/ChatLine.java`) | **Líneas de código**: 18
- **Declaración**: `class ChatLine`
- **Rol**: Representación de una línea individual de texto en el historial del chat.
- **Descripción Técnica**: Almacena el texto del mensaje y un contador de tiempo de vida (updateCounter) para su desvanecimiento gradual.
- **Guía de Modificación**: Permite modificar la duración de visibilidad de los mensajes o añadir marcas de tiempo (timestamps).
- **Campos Clave Destacados**:
  - `String message`
  - `int updateCounter`

##### <a id="class-gui"></a>[`Gui`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Gui.java)
- **Ruta y Paquete**: `net.minecraft.src.Gui` (`net/minecraft/src/Gui.java`) | **Líneas de código**: 91
- **Declaración**: `class Gui`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'Gui'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'Gui'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void drawRect(int i, int j, int k, int l, int i1)`
  - `void drawGradientRect(int i, int j, int k, int l, int i1, int j1)`
  - `void drawCenteredString(FontRenderer fontrenderer, String s, int i, int j, int k)`
  - `void drawString(FontRenderer fontrenderer, String s, int i, int j, int k)`
  - `void drawTexturedModalRect(int i, int j, int k, int l, int i1, int j1)`
- **Campos Clave Destacados**:
  - `float zLevel`

##### <a id="class-guibutton"></a>[`GuiButton`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiButton.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiButton` (`net/minecraft/src/GuiButton.java`) | **Líneas de código**: 93
- **Declaración**: `class GuiButton extends Gui`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiButton'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiButton'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `int getHoverState(boolean flag)`
  - `void drawButton(Minecraft minecraft, int i, int j)`
  - `void mouseDragged(Minecraft minecraft, int i, int j)`
  - `void mouseReleased(int i, int j)`
  - `boolean mousePressed(Minecraft minecraft, int i, int j)`
- **Campos Clave Destacados**:
  - `int width`
  - `int height`
  - `int xPosition`
  - `int yPosition`
  - `String displayString`
  - `int id`

##### <a id="class-guichat"></a>[`GuiChat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiChat.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiChat` (`net/minecraft/src/GuiChat.java`) | **Líneas de código**: 87
- **Declaración**: `class GuiChat extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiChat'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiChat'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void onGuiClosed()`
  - `void updateScreen()`
  - `void keyTyped(char c, int i)`
  - `void drawScreen(int i, int j, float f)`
  - `void mouseClicked(int i, int j, int k)`

##### <a id="class-guichest"></a>[`GuiChest`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiChest.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiChest` (`net/minecraft/src/GuiChest.java`) | **Líneas de código**: 69
- **Declaración**: `class GuiChest extends GuiContainer`
- **Rol**: Pantalla del inventario de cofre simple (27 slots) o doble (54 slots).
- **Descripción Técnica**: Muestra las casillas del cofre en la mitad superior y el inventario del jugador en la mitad inferior para transferencia de ítems.
- **Guía de Modificación**: Permite añadir botones de ordenación automática de cofres.
- **Métodos Clave Destacados**:
  - `void drawGuiContainerForegroundLayer()`
  - `void drawGuiContainerBackgroundLayer(float f)`

##### <a id="class-guiconflictwarning"></a>[`GuiConflictWarning`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiConflictWarning.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiConflictWarning` (`net/minecraft/src/GuiConflictWarning.java`) | **Líneas de código**: 52
- **Declaración**: `class GuiConflictWarning extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiConflictWarning'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiConflictWarning'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void updateScreen()`
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guiconnectfailed"></a>[`GuiConnectFailed`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiConnectFailed.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiConnectFailed` (`net/minecraft/src/GuiConnectFailed.java`) | **Líneas de código**: 50
- **Declaración**: `class GuiConnectFailed extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiConnectFailed'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiConnectFailed'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void updateScreen()`
  - `void keyTyped(char c, int i)`
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guiconnecting"></a>[`GuiConnecting`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiConnecting.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiConnecting` (`net/minecraft/src/GuiConnecting.java`) | **Líneas de código**: 82
- **Declaración**: `class GuiConnecting extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiConnecting'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiConnecting'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void updateScreen()`
  - `void keyTyped(char c, int i)`
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guicontainer"></a>[`GuiContainer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiContainer.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiContainer` (`net/minecraft/src/GuiContainer.java`) | **Líneas de código**: 278
- **Declaración**: `class GuiContainer extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiContainer'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiContainer'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void drawScreen(int i, int j, float f)`
  - `void drawGuiContainerForegroundLayer()`
  - `void mouseClicked(int i, int j, int k)`
  - `void mouseMovedOrUp(int i, int j, int k)`
  - `void keyTyped(char c, int i)`
  - `void onGuiClosed()`
- **Campos Clave Destacados**:
  - `int xSize`
  - `int ySize`
  - `List inventorySlots`

##### <a id="class-guicontrols"></a>[`GuiControls`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiControls.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiControls` (`net/minecraft/src/GuiControls.java`) | **Líneas de código**: 71
- **Declaración**: `class GuiControls extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiControls'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiControls'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void keyTyped(char c, int i)`
  - `void drawScreen(int i, int j, float f)`
- **Campos Clave Destacados**:
  - `String screenTitle`

##### <a id="class-guideleteworld"></a>[`GuiDeleteWorld`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiDeleteWorld.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiDeleteWorld` (`net/minecraft/src/GuiDeleteWorld.java`) | **Líneas de código**: 41
- **Declaración**: `class GuiDeleteWorld extends GuiSelectWorld`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiDeleteWorld'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiDeleteWorld'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui2()`
  - `void selectWorld(int i)`
  - `void deleteWorld(boolean flag, int i)`

##### <a id="class-guidownloadterrain"></a>[`GuiDownloadTerrain`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiDownloadTerrain.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiDownloadTerrain` (`net/minecraft/src/GuiDownloadTerrain.java`) | **Líneas de código**: 52
- **Declaración**: `class GuiDownloadTerrain extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiDownloadTerrain'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiDownloadTerrain'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void keyTyped(char c, int i)`
  - `void initGui()`
  - `void updateScreen()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guieditsign"></a>[`GuiEditSign`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiEditSign.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiEditSign` (`net/minecraft/src/GuiEditSign.java`) | **Líneas de código**: 119
- **Declaración**: `class GuiEditSign extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiEditSign'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiEditSign'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void onGuiClosed()`
  - `void updateScreen()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void keyTyped(char c, int i)`
  - `void drawScreen(int i, int j, float f)`
- **Campos Clave Destacados**:
  - `String screenTitle`

##### <a id="class-guifurnace"></a>[`GuiFurnace`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiFurnace.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiFurnace` (`net/minecraft/src/GuiFurnace.java`) | **Líneas de código**: 59
- **Declaración**: `class GuiFurnace extends GuiContainer`
- **Rol**: Pantalla interactiva del horno de fundición.
- **Descripción Técnica**: Dibuja la animación de la llama de fuego consumiéndose y la flecha de progreso del cocinado del material.
- **Guía de Modificación**: Modificar si se añaden ranuras adicionales de combustible o mejoras (upgrades).
- **Métodos Clave Destacados**:
  - `void drawGuiContainerForegroundLayer()`
  - `void drawGuiContainerBackgroundLayer(float f)`

##### <a id="class-guigameover"></a>[`GuiGameOver`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiGameOver.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiGameOver` (`net/minecraft/src/GuiGameOver.java`) | **Líneas de código**: 62
- **Declaración**: `class GuiGameOver extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiGameOver'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiGameOver'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void keyTyped(char c, int i)`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`
  - `boolean doesGuiPauseGame()`

##### <a id="class-guiingame"></a>[`GuiIngame`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiIngame.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiIngame` (`net/minecraft/src/GuiIngame.java`) | **Líneas de código**: 394
- **Declaración**: `class GuiIngame extends Gui`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiIngame'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiIngame'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void renderGameOverlay(float f, boolean flag, int i, int j)`
  - `void func_555_a()`
  - `void addChatMessage(String s)`
  - `void func_553_b(String s)`
- **Campos Clave Destacados**:
  - `String field_933_a`
  - `float field_6446_b`

##### <a id="class-guiingamemenu"></a>[`GuiIngameMenu`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiIngameMenu.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiIngameMenu` (`net/minecraft/src/GuiIngameMenu.java`) | **Líneas de código**: 76
- **Declaración**: `class GuiIngameMenu extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiIngameMenu'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiIngameMenu'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void updateScreen()`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guiinventory"></a>[`GuiInventory`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiInventory.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiInventory` (`net/minecraft/src/GuiInventory.java`) | **Líneas de código**: 101
- **Declaración**: `class GuiInventory extends GuiContainer`
- **Rol**: Pantalla interactiva del inventario personal del jugador (tecla E).
- **Descripción Técnica**: Muestra la vista previa 3D del modelo del jugador girando, las 4 casillas de armadura, la cuadrícula de crafteo 2x2, la casilla de resultado y el inventario general.
- **Guía de Modificación**: Permite añadir slots especiales de accesorios (anillos, capas) o botones adicionales.
- **Métodos Clave Destacados**:
  - `void drawGuiContainerForegroundLayer()`
  - `void drawScreen(int i, int j, float f)`
  - `void drawGuiContainerBackgroundLayer(float f)`

##### <a id="class-guimainmenu"></a>[`GuiMainMenu`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiMainMenu.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiMainMenu` (`net/minecraft/src/GuiMainMenu.java`) | **Líneas de código**: 251
- **Declaración**: `class GuiMainMenu extends GuiScreen`
- **Rol**: Pantalla del menú principal del juego.
- **Descripción Técnica**: Renderiza el logotipo tridimensional de Minecraft, los textos amarillos oscilantes ('Splash text') y los botones de 'Singleplayer', 'Multiplayer', 'Mods and Texture Packs' y 'Options'.
- **Guía de Modificación**: Modifica los botones o el fondo para personalizar la pantalla de bienvenida de un modpack.
- **Métodos Clave Destacados**:
  - `void updateScreen()`
  - `void keyTyped(char c, int i)`
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guimultiplayer"></a>[`GuiMultiplayer`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiMultiplayer.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiMultiplayer` (`net/minecraft/src/GuiMultiplayer.java`) | **Líneas de código**: 120
- **Declaración**: `class GuiMultiplayer extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiMultiplayer'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiMultiplayer'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void updateScreen()`
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void keyTyped(char c, int i)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-guioptions"></a>[`GuiOptions`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiOptions.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiOptions` (`net/minecraft/src/GuiOptions.java`) | **Líneas de código**: 70
- **Declaración**: `class GuiOptions extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiOptions'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiOptions'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`
- **Campos Clave Destacados**:
  - `String screenTitle`

##### <a id="class-guiscreen"></a>[`GuiScreen`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiScreen.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiScreen` (`net/minecraft/src/GuiScreen.java`) | **Líneas de código**: 196
- **Declaración**: `class GuiScreen extends Gui`
- **Rol**: Clase base para todas las pantallas interactivas de menús y diálogos.
- **Descripción Técnica**: Maneja el ciclo de vida de una pantalla: initGui() (configuración de botones), actionPerformed() (manejo de clics), keyTyped() (teclado), y drawScreen() (dibujo de elementos 2D).
- **Guía de Modificación**: Cualquier nuevo menú o panel interactivo debe extender GuiScreen.
- **Métodos Clave Destacados**:
  - `void drawScreen(int i, int j, float f)`
  - `void keyTyped(char c, int i)`
  - `String getClipboardString()`
  - `void mouseClicked(int i, int j, int k)`
  - `void mouseMovedOrUp(int i, int j, int k)`
  - `void actionPerformed(GuiButton guibutton)`
- **Campos Clave Destacados**:
  - `Minecraft mc`
  - `int width`
  - `int height`
  - `boolean field_948_f`
  - `FontRenderer fontRenderer`

##### <a id="class-guiselectworld"></a>[`GuiSelectWorld`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiSelectWorld.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiSelectWorld` (`net/minecraft/src/GuiSelectWorld.java`) | **Líneas de código**: 98
- **Declaración**: `class GuiSelectWorld extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiSelectWorld'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiSelectWorld'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `String getWorldName(int i)`
  - `void initGui2()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void selectWorld(int i)`
  - `void drawScreen(int i, int j, float f)`
- **Campos Clave Destacados**:
  - `GuiScreen parentScreen`
  - `String screenTitle`

##### <a id="class-guislider"></a>[`GuiSlider`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiSlider.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiSlider` (`net/minecraft/src/GuiSlider.java`) | **Líneas de código**: 83
- **Declaración**: `class GuiSlider extends GuiButton`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiSlider'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiSlider'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `int getHoverState(boolean flag)`
  - `void mouseDragged(Minecraft minecraft, int i, int j)`
  - `boolean mousePressed(Minecraft minecraft, int i, int j)`
  - `void mouseReleased(int i, int j)`
- **Campos Clave Destacados**:
  - `float sliderValue`
  - `boolean dragging`

##### <a id="class-guismallbutton"></a>[`GuiSmallButton`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiSmallButton.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiSmallButton` (`net/minecraft/src/GuiSmallButton.java`) | **Líneas de código**: 14
- **Declaración**: `class GuiSmallButton extends GuiButton`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiSmallButton'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiSmallButton'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.

##### <a id="class-guiunused"></a>[`GuiUnused`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiUnused.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiUnused` (`net/minecraft/src/GuiUnused.java`) | **Líneas de código**: 28
- **Declaración**: `class GuiUnused extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiUnused'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiUnused'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void drawScreen(int i, int j, float f)`
  - `void keyTyped(char c, int i)`

##### <a id="class-guiyesno"></a>[`GuiYesNo`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/GuiYesNo.java)
- **Ruta y Paquete**: `net.minecraft.src.GuiYesNo` (`net/minecraft/src/GuiYesNo.java`) | **Líneas de código**: 42
- **Declaración**: `class GuiYesNo extends GuiScreen`
- **Rol**: Pantalla o elemento de interfaz gráfica: 'GuiYesNo'.
- **Descripción Técnica**: Controla los botones, textos, fondos y eventos de ratón/teclado para la interfaz 'GuiYesNo'.
- **Guía de Modificación**: Sobrescribe initGui, actionPerformed o drawScreen para personalizar la interfaz.
- **Métodos Clave Destacados**:
  - `void initGui()`
  - `void actionPerformed(GuiButton guibutton)`
  - `void drawScreen(int i, int j, float f)`

##### <a id="class-iprogressupdate"></a>[`IProgressUpdate`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/IProgressUpdate.java)
- **Ruta y Paquete**: `net.minecraft.src.IProgressUpdate` (`net/minecraft/src/IProgressUpdate.java`) | **Líneas de código**: 15
- **Declaración**: `interface IProgressUpdate`
- **Rol**: Interfaz para receptores de progreso de tareas largas.
- **Descripción Técnica**: Permite que World actualice el texto y porcentaje en LoadingScreenRenderer durante la carga o guardado.
- **Guía de Modificación**: Usar para mostrar barras de progreso en procesos extensos de mods.

##### <a id="class-packet3chat"></a>[`Packet3Chat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet3Chat.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet3Chat` (`net/minecraft/src/Packet3Chat.java`) | **Líneas de código**: 41
- **Declaración**: `class Packet3Chat extends Packet`
- **Rol**: Paquete de datos de red: 'Packet3Chat' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet3Chat'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `String message`

##### <a id="class-screenshothelper"></a>[`ScreenShotHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/ScreenShotHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.ScreenShotHelper` (`net/minecraft/src/ScreenShotHelper.java`) | **Líneas de código**: 75
- **Declaración**: `class ScreenShotHelper`
- **Rol**: Utilidad para capturar pantallas de juego y guardarlas en disco.
- **Descripción Técnica**: Lee el framebuffer mediante glReadPixels, convierte el formato de color BGR a RGB y escribe un archivo PNG en la carpeta screenshots.
- **Guía de Modificación**: Modificar si se desea añadir marcas de agua o formatos de compresión alternativos a las capturas.
- **Métodos Clave Destacados**:
  - `String func_4148_a(File file, int i, int j)`


### 10. Redes y Protocolo Multijugador (Networking & Packets)
> **Resumen del subsistema**: La arquitectura cliente-servidor de Alpha 1.2.6 (SMP). Protocolo basado en sockets TCP con hilos asíncronos de lectura y escritura (NetworkManager), y el catálogo completo de paquetes binarios (0x00 a 0xFF) para sincronización de entidades, bloques, inventarios y chat.

**Total de clases**: 46

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`NetClientHandler`](#class-netclienthandler) | `class` | `ext NetHandler` | 557 | Despachador central de paquetes de red en el cliente multijugador... |
| [`NetHandler`](#class-nethandler) | `class` | `-` | 185 | Manejador base de eventos del protocolo de paquetes de red. |
| [`NetworkManager`](#class-networkmanager) | `class` | `-` | 239 | Gestor multihilo de sockets TCP para comunicaciones de red SMP. |
| [`NetworkMasterThread`](#class-networkmasterthread) | `class` | `ext Thread` | 44 | Hilo maestro supervisor de la conexión de red TCP. |
| [`NetworkReaderThread`](#class-networkreaderthread) | `class` | `ext Thread` | 36 | Hilo secundario para la recepción continua de paquetes de red des... |
| [`NetworkWriterThread`](#class-networkwriterthread) | `class` | `ext Thread` | 36 | Hilo secundario para la transmisión asíncrona de paquetes hacia e... |
| [`Packet`](#class-packet) | `class` | `-` | 152 | Paquete de datos de red: 'Packet' en el protocolo SMP. |
| [`Packet0KeepAlive`](#class-packet0keepalive) | `class` | `ext Packet` | 31 | Paquete de datos de red: 'Packet0KeepAlive' en el protocolo SMP. |
| [`Packet10Flying`](#class-packet10flying) | `class` | `ext Packet` | 49 | Paquete de datos de red: 'Packet10Flying' en el protocolo SMP. |
| [`Packet11PlayerPosition`](#class-packet11playerposition) | `class` | `ext Packet10Flying` | 48 | Paquete de datos de red: 'Packet11PlayerPosition' en el protocolo... |
| [`Packet12PlayerLook`](#class-packet12playerlook) | `class` | `ext Packet10Flying` | 42 | Paquete de datos de red: 'Packet12PlayerLook' en el protocolo SMP... |
| [`Packet13PlayerLookMove`](#class-packet13playerlookmove) | `class` | `ext Packet10Flying` | 56 | Paquete de datos de red: 'Packet13PlayerLookMove' en el protocolo... |
| [`Packet14BlockDig`](#class-packet14blockdig) | `class` | `ext Packet` | 57 | Paquete de datos de red: 'Packet14BlockDig' en el protocolo SMP. |
| [`Packet15Place`](#class-packet15place) | `class` | `ext Packet` | 57 | Paquete de datos de red: 'Packet15Place' en el protocolo SMP. |
| [`Packet16BlockItemSwitch`](#class-packet16blockitemswitch) | `class` | `ext Packet` | 45 | Paquete de datos de red: 'Packet16BlockItemSwitch' en el protocol... |
| [`Packet17AddToInventory`](#class-packet17addtoinventory) | `class` | `ext Packet` | 42 | Paquete de datos de red: 'Packet17AddToInventory' en el protocolo... |
| [`Packet18ArmAnimation`](#class-packet18armanimation) | `class` | `ext Packet` | 45 | Paquete de datos de red: 'Packet18ArmAnimation' en el protocolo S... |
| [`Packet1Login`](#class-packet1login) | `class` | `ext Packet` | 55 | Paquete de datos de red: 'Packet1Login' en el protocolo SMP. |
| [`Packet20NamedEntitySpawn`](#class-packet20namedentityspawn) | `class` | `ext Packet` | 70 | Paquete de datos de red: 'Packet20NamedEntitySpawn' en el protoco... |
| [`Packet21PickupSpawn`](#class-packet21pickupspawn) | `class` | `ext Packet` | 73 | Paquete de datos de red: 'Packet21PickupSpawn' en el protocolo SM... |
| [`Packet22Collect`](#class-packet22collect) | `class` | `ext Packet` | 39 | Paquete de datos de red: 'Packet22Collect' en el protocolo SMP. |
| [`Packet23VehicleSpawn`](#class-packet23vehiclespawn) | `class` | `ext Packet` | 48 | Paquete de datos de red: 'Packet23VehicleSpawn' en el protocolo S... |
| [`Packet24MobSpawn`](#class-packet24mobspawn) | `class` | `ext Packet` | 65 | Paquete de datos de red: 'Packet24MobSpawn' en el protocolo SMP. |
| [`Packet255KickDisconnect`](#class-packet255kickdisconnect) | `class` | `ext Packet` | 41 | Paquete de datos de red: 'Packet255KickDisconnect' en el protocol... |
| [`Packet28`](#class-packet28) | `class` | `ext Packet` | 83 | Paquete de datos de red: 'Packet28' en el protocolo SMP. |
| [`Packet29DestroyEntity`](#class-packet29destroyentity) | `class` | `ext Packet` | 36 | Paquete de datos de red: 'Packet29DestroyEntity' en el protocolo ... |
| [`Packet2Handshake`](#class-packet2handshake) | `class` | `ext Packet` | 41 | Paquete de datos de red: 'Packet2Handshake' en el protocolo SMP. |
| [`Packet30Entity`](#class-packet30entity) | `class` | `ext Packet` | 43 | Paquete de datos de red: 'Packet30Entity' en el protocolo SMP. |
| [`Packet31RelEntityMove`](#class-packet31relentitymove) | `class` | `ext Packet30Entity` | 35 | Paquete de datos de red: 'Packet31RelEntityMove' en el protocolo ... |
| [`Packet32EntityLook`](#class-packet32entitylook) | `class` | `ext Packet30Entity` | 34 | Paquete de datos de red: 'Packet32EntityLook' en el protocolo SMP... |
| [`Packet33RelEntityMoveLook`](#class-packet33relentitymovelook) | `class` | `ext Packet30Entity` | 40 | Paquete de datos de red: 'Packet33RelEntityMoveLook' en el protoc... |
| [`Packet34EntityTeleport`](#class-packet34entityteleport) | `class` | `ext Packet` | 61 | Paquete de datos de red: 'Packet34EntityTeleport' en el protocolo... |
| [`Packet38`](#class-packet38) | `class` | `ext Packet` | 39 | Paquete de datos de red: 'Packet38' en el protocolo SMP. |
| [`Packet39`](#class-packet39) | `class` | `ext Packet` | 39 | Paquete de datos de red: 'Packet39' en el protocolo SMP. |
| [`Packet4UpdateTime`](#class-packet4updatetime) | `class` | `ext Packet` | 36 | Paquete de datos de red: 'Packet4UpdateTime' en el protocolo SMP. |
| [`Packet50PreChunk`](#class-packet50prechunk) | `class` | `ext Packet` | 42 | Paquete de datos de red: 'Packet50PreChunk' en el protocolo SMP. |
| [`Packet51MapChunk`](#class-packet51mapchunk) | `class` | `ext Packet` | 76 | Paquete de datos de red: 'Packet51MapChunk' en el protocolo SMP. |
| [`Packet52MultiBlockChange`](#class-packet52multiblockchange) | `class` | `ext Packet` | 63 | Paquete de datos de red: 'Packet52MultiBlockChange' en el protoco... |
| [`Packet53BlockChange`](#class-packet53blockchange) | `class` | `ext Packet` | 49 | Paquete de datos de red: 'Packet53BlockChange' en el protocolo SM... |
| [`Packet59ComplexEntity`](#class-packet59complexentity) | `class` | `ext Packet` | 69 | Paquete de datos de red: 'Packet59ComplexEntity' en el protocolo ... |
| [`Packet5PlayerInventory`](#class-packet5playerinventory) | `class` | `ext Packet` | 75 | Paquete de datos de red: 'Packet5PlayerInventory' en el protocolo... |
| [`Packet60`](#class-packet60) | `class` | `ext Packet` | 75 | Paquete de datos de red: 'Packet60' en el protocolo SMP. |
| [`Packet6SpawnPosition`](#class-packet6spawnposition) | `class` | `ext Packet` | 42 | Paquete de datos de red: 'Packet6SpawnPosition' en el protocolo S... |
| [`Packet7`](#class-packet7) | `class` | `ext Packet` | 49 | Paquete de datos de red: 'Packet7' en el protocolo SMP. |
| [`Packet8`](#class-packet8) | `class` | `ext Packet` | 36 | Paquete de datos de red: 'Packet8' en el protocolo SMP. |
| [`Packet9`](#class-packet9) | `class` | `ext Packet` | 32 | Paquete de datos de red: 'Packet9' en el protocolo SMP. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-netclienthandler"></a>[`NetClientHandler`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetClientHandler.java)
- **Ruta y Paquete**: `net.minecraft.src.NetClientHandler` (`net/minecraft/src/NetClientHandler.java`) | **Líneas de código**: 557
- **Declaración**: `class NetClientHandler extends NetHandler`
- **Rol**: Despachador central de paquetes de red en el cliente multijugador.
- **Descripción Técnica**: Recibe los paquetes del servidor y los traduce en modificaciones concretas del mundo local: creación de entidades, bloques, salud y chat.
- **Guía de Modificación**: Punto clave para interceptar o añadir nuevos tipos de mensajes entre cliente y servidor.
- **Métodos Clave Destacados**:
  - `void processReadPackets()`
  - `void handleLogin(Packet1Login packet1login)`
  - `void handlePickupSpawn(Packet21PickupSpawn packet21pickupspawn)`
  - `void handleVehicleSpawn(Packet23VehicleSpawn packet23vehiclespawn)`
  - `void func_6498_a(Packet28 packet28)`
  - `void handleNamedEntitySpawn(Packet20NamedEntitySpawn packet20namedentityspawn)`
- **Campos Clave Destacados**:
  - `String field_1209_a`

##### <a id="class-nethandler"></a>[`NetHandler`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetHandler.java)
- **Ruta y Paquete**: `net.minecraft.src.NetHandler` (`net/minecraft/src/NetHandler.java`) | **Líneas de código**: 185
- **Declaración**: `class NetHandler`
- **Rol**: Manejador base de eventos del protocolo de paquetes de red.
- **Descripción Técnica**: Declara métodos vacíos para todos los paquetes (handleLogin, handleChat, handleBlockDig, etc.).
- **Guía de Modificación**: Base abstracta que implementan NetClientHandler y NetServerHandler.
- **Métodos Clave Destacados**:
  - `void handleMapChunk(Packet51MapChunk packet51mapchunk)`
  - `void func_4114_b(Packet packet)`
  - `void handleErrorMessage(String s)`
  - `void handleKickDisconnect(Packet255KickDisconnect packet255kickdisconnect)`
  - `void handleLogin(Packet1Login packet1login)`
  - `void handleFlying(Packet10Flying packet10flying)`

##### <a id="class-networkmanager"></a>[`NetworkManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetworkManager.java)
- **Ruta y Paquete**: `net.minecraft.src.NetworkManager` (`net/minecraft/src/NetworkManager.java`) | **Líneas de código**: 239
- **Declaración**: `class NetworkManager`
- **Rol**: Gestor multihilo de sockets TCP para comunicaciones de red SMP.
- **Descripción Técnica**: Mantiene hilos dedicados para lectura y escritura asíncrona sobre sockets TCP, administrando colas de paquetes entrantes y salientes.
- **Guía de Modificación**: Modificar esta clase para cifrado de red, compresión de paquetes o proxies.
- **Métodos Clave Destacados**:
  - `void addToSendQueue(Packet packet)`
  - `void networkShutdown(String s)`
  - `void processReadPackets()`
- **Campos Clave Destacados**:
  - `Object threadSyncObject`
  - `int numReadThreads`
  - `int numWriteThreads`

##### <a id="class-networkmasterthread"></a>[`NetworkMasterThread`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetworkMasterThread.java)
- **Ruta y Paquete**: `net.minecraft.src.NetworkMasterThread` (`net/minecraft/src/NetworkMasterThread.java`) | **Líneas de código**: 44
- **Declaración**: `class NetworkMasterThread extends Thread`
- **Rol**: Hilo maestro supervisor de la conexión de red TCP.
- **Descripción Técnica**: Supervisa el estado de salud de los hilos de lectura y escritura del socket.
- **Guía de Modificación**: Garantiza el cierre limpio de recursos en caso de caída de conexión.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-networkreaderthread"></a>[`NetworkReaderThread`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetworkReaderThread.java)
- **Ruta y Paquete**: `net.minecraft.src.NetworkReaderThread` (`net/minecraft/src/NetworkReaderThread.java`) | **Líneas de código**: 36
- **Declaración**: `class NetworkReaderThread extends Thread`
- **Rol**: Hilo secundario para la recepción continua de paquetes de red desde el socket.
- **Descripción Técnica**: Lee bytes del DataInputStream, deserializa el paquete y lo encola en la lista de paquetes entrantes.
- **Guía de Modificación**: Evita que la latencia de la red bloquee el hilo principal de renderizado.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-networkwriterthread"></a>[`NetworkWriterThread`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NetworkWriterThread.java)
- **Ruta y Paquete**: `net.minecraft.src.NetworkWriterThread` (`net/minecraft/src/NetworkWriterThread.java`) | **Líneas de código**: 36
- **Declaración**: `class NetworkWriterThread extends Thread`
- **Rol**: Hilo secundario para la transmisión asíncrona de paquetes hacia el socket TCP.
- **Descripción Técnica**: Toma paquetes de la cola de salida, los serializa en DataOutputStream y los vacía en el flujo de red.
- **Guía de Modificación**: Garantiza una emisión fluida de paquetes sin retrasar los ticks del juego.
- **Métodos Clave Destacados**:
  - `void run()`

##### <a id="class-packet"></a>[`Packet`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet` (`net/minecraft/src/Packet.java`) | **Líneas de código**: 152
- **Declaración**: `class Packet`
- **Rol**: Paquete de datos de red: 'Packet' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `Packet getNewPacket(int i)`
  - `int getPacketId()`
  - `Packet readPacket(DataInputStream datainputstream)`
  - `void writePacket(Packet packet, DataOutputStream dataoutputstream)`
- **Campos Clave Destacados**:
  - `boolean isChunkDataPacket`

##### <a id="class-packet0keepalive"></a>[`Packet0KeepAlive`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet0KeepAlive.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet0KeepAlive` (`net/minecraft/src/Packet0KeepAlive.java`) | **Líneas de código**: 31
- **Declaración**: `class Packet0KeepAlive extends Packet`
- **Rol**: Paquete de datos de red: 'Packet0KeepAlive' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet0KeepAlive'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void processPacket(NetHandler nethandler)`
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet10flying"></a>[`Packet10Flying`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet10Flying.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet10Flying` (`net/minecraft/src/Packet10Flying.java`) | **Líneas de código**: 49
- **Declaración**: `class Packet10Flying extends Packet`
- **Rol**: Paquete de datos de red: 'Packet10Flying' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet10Flying'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void processPacket(NetHandler nethandler)`
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `double xPosition`
  - `double yPosition`
  - `double zPosition`
  - `double stance`
  - `float yaw`
  - `float pitch`

##### <a id="class-packet11playerposition"></a>[`Packet11PlayerPosition`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet11PlayerPosition.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet11PlayerPosition` (`net/minecraft/src/Packet11PlayerPosition.java`) | **Líneas de código**: 48
- **Declaración**: `class Packet11PlayerPosition extends Packet10Flying`
- **Rol**: Paquete de datos de red: 'Packet11PlayerPosition' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet11PlayerPosition'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet12playerlook"></a>[`Packet12PlayerLook`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet12PlayerLook.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet12PlayerLook` (`net/minecraft/src/Packet12PlayerLook.java`) | **Líneas de código**: 42
- **Declaración**: `class Packet12PlayerLook extends Packet10Flying`
- **Rol**: Paquete de datos de red: 'Packet12PlayerLook' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet12PlayerLook'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet13playerlookmove"></a>[`Packet13PlayerLookMove`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet13PlayerLookMove.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet13PlayerLookMove` (`net/minecraft/src/Packet13PlayerLookMove.java`) | **Líneas de código**: 56
- **Declaración**: `class Packet13PlayerLookMove extends Packet10Flying`
- **Rol**: Paquete de datos de red: 'Packet13PlayerLookMove' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet13PlayerLookMove'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet14blockdig"></a>[`Packet14BlockDig`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet14BlockDig.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet14BlockDig` (`net/minecraft/src/Packet14BlockDig.java`) | **Líneas de código**: 57
- **Declaración**: `class Packet14BlockDig extends Packet`
- **Rol**: Paquete de datos de red: 'Packet14BlockDig' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet14BlockDig'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `int face`
  - `int status`

##### <a id="class-packet15place"></a>[`Packet15Place`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet15Place.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet15Place` (`net/minecraft/src/Packet15Place.java`) | **Líneas de código**: 57
- **Declaración**: `class Packet15Place extends Packet`
- **Rol**: Paquete de datos de red: 'Packet15Place' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet15Place'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int id`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `int direction`

##### <a id="class-packet16blockitemswitch"></a>[`Packet16BlockItemSwitch`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet16BlockItemSwitch.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet16BlockItemSwitch` (`net/minecraft/src/Packet16BlockItemSwitch.java`) | **Líneas de código**: 45
- **Declaración**: `class Packet16BlockItemSwitch extends Packet`
- **Rol**: Paquete de datos de red: 'Packet16BlockItemSwitch' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet16BlockItemSwitch'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int unused`
  - `int id`

##### <a id="class-packet17addtoinventory"></a>[`Packet17AddToInventory`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet17AddToInventory.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet17AddToInventory` (`net/minecraft/src/Packet17AddToInventory.java`) | **Líneas de código**: 42
- **Declaración**: `class Packet17AddToInventory extends Packet`
- **Rol**: Paquete de datos de red: 'Packet17AddToInventory' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet17AddToInventory'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int id`
  - `int count`
  - `int durability`

##### <a id="class-packet18armanimation"></a>[`Packet18ArmAnimation`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet18ArmAnimation.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet18ArmAnimation` (`net/minecraft/src/Packet18ArmAnimation.java`) | **Líneas de código**: 45
- **Declaración**: `class Packet18ArmAnimation extends Packet`
- **Rol**: Paquete de datos de red: 'Packet18ArmAnimation' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet18ArmAnimation'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `int animate`

##### <a id="class-packet1login"></a>[`Packet1Login`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet1Login.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet1Login` (`net/minecraft/src/Packet1Login.java`) | **Líneas de código**: 55
- **Declaración**: `class Packet1Login extends Packet`
- **Rol**: Paquete de datos de red: 'Packet1Login' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet1Login'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int protocolVersion`
  - `String username`
  - `String password`
  - `long field_4074_d`
  - `byte field_4073_e`

##### <a id="class-packet20namedentityspawn"></a>[`Packet20NamedEntitySpawn`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet20NamedEntitySpawn.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet20NamedEntitySpawn` (`net/minecraft/src/Packet20NamedEntitySpawn.java`) | **Líneas de código**: 70
- **Declaración**: `class Packet20NamedEntitySpawn extends Packet`
- **Rol**: Paquete de datos de red: 'Packet20NamedEntitySpawn' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet20NamedEntitySpawn'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `String name`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `byte rotation`

##### <a id="class-packet21pickupspawn"></a>[`Packet21PickupSpawn`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet21PickupSpawn.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet21PickupSpawn` (`net/minecraft/src/Packet21PickupSpawn.java`) | **Líneas de código**: 73
- **Declaración**: `class Packet21PickupSpawn extends Packet`
- **Rol**: Paquete de datos de red: 'Packet21PickupSpawn' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet21PickupSpawn'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `byte rotation`
  - `byte pitch`

##### <a id="class-packet22collect"></a>[`Packet22Collect`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet22Collect.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet22Collect` (`net/minecraft/src/Packet22Collect.java`) | **Líneas de código**: 39
- **Declaración**: `class Packet22Collect extends Packet`
- **Rol**: Paquete de datos de red: 'Packet22Collect' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet22Collect'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int collectedEntityId`
  - `int collectorEntityId`

##### <a id="class-packet23vehiclespawn"></a>[`Packet23VehicleSpawn`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet23VehicleSpawn.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet23VehicleSpawn` (`net/minecraft/src/Packet23VehicleSpawn.java`) | **Líneas de código**: 48
- **Declaración**: `class Packet23VehicleSpawn extends Packet`
- **Rol**: Paquete de datos de red: 'Packet23VehicleSpawn' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet23VehicleSpawn'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `int type`

##### <a id="class-packet24mobspawn"></a>[`Packet24MobSpawn`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet24MobSpawn.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet24MobSpawn` (`net/minecraft/src/Packet24MobSpawn.java`) | **Líneas de código**: 65
- **Declaración**: `class Packet24MobSpawn extends Packet`
- **Rol**: Paquete de datos de red: 'Packet24MobSpawn' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet24MobSpawn'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `byte type`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `byte yaw`

##### <a id="class-packet255kickdisconnect"></a>[`Packet255KickDisconnect`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet255KickDisconnect.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet255KickDisconnect` (`net/minecraft/src/Packet255KickDisconnect.java`) | **Líneas de código**: 41
- **Declaración**: `class Packet255KickDisconnect extends Packet`
- **Rol**: Paquete de datos de red: 'Packet255KickDisconnect' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet255KickDisconnect'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `String reason`

##### <a id="class-packet28"></a>[`Packet28`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet28.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet28` (`net/minecraft/src/Packet28.java`) | **Líneas de código**: 83
- **Declaración**: `class Packet28 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet28' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet28'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int field_6367_a`
  - `int field_6366_b`
  - `int field_6369_c`
  - `int field_6368_d`

##### <a id="class-packet29destroyentity"></a>[`Packet29DestroyEntity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet29DestroyEntity.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet29DestroyEntity` (`net/minecraft/src/Packet29DestroyEntity.java`) | **Líneas de código**: 36
- **Declaración**: `class Packet29DestroyEntity extends Packet`
- **Rol**: Paquete de datos de red: 'Packet29DestroyEntity' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet29DestroyEntity'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`

##### <a id="class-packet2handshake"></a>[`Packet2Handshake`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet2Handshake.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet2Handshake` (`net/minecraft/src/Packet2Handshake.java`) | **Líneas de código**: 41
- **Declaración**: `class Packet2Handshake extends Packet`
- **Rol**: Paquete de datos de red: 'Packet2Handshake' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet2Handshake'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `String username`

##### <a id="class-packet30entity"></a>[`Packet30Entity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet30Entity.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet30Entity` (`net/minecraft/src/Packet30Entity.java`) | **Líneas de código**: 43
- **Declaración**: `class Packet30Entity extends Packet`
- **Rol**: Paquete de datos de red: 'Packet30Entity' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet30Entity'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `byte xPosition`
  - `byte yPosition`
  - `byte zPosition`
  - `byte yaw`
  - `byte pitch`

##### <a id="class-packet31relentitymove"></a>[`Packet31RelEntityMove`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet31RelEntityMove.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet31RelEntityMove` (`net/minecraft/src/Packet31RelEntityMove.java`) | **Líneas de código**: 35
- **Declaración**: `class Packet31RelEntityMove extends Packet30Entity`
- **Rol**: Paquete de datos de red: 'Packet31RelEntityMove' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet31RelEntityMove'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet32entitylook"></a>[`Packet32EntityLook`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet32EntityLook.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet32EntityLook` (`net/minecraft/src/Packet32EntityLook.java`) | **Líneas de código**: 34
- **Declaración**: `class Packet32EntityLook extends Packet30Entity`
- **Rol**: Paquete de datos de red: 'Packet32EntityLook' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet32EntityLook'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet33relentitymovelook"></a>[`Packet33RelEntityMoveLook`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet33RelEntityMoveLook.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet33RelEntityMoveLook` (`net/minecraft/src/Packet33RelEntityMoveLook.java`) | **Líneas de código**: 40
- **Declaración**: `class Packet33RelEntityMoveLook extends Packet30Entity`
- **Rol**: Paquete de datos de red: 'Packet33RelEntityMoveLook' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet33RelEntityMoveLook'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`

##### <a id="class-packet34entityteleport"></a>[`Packet34EntityTeleport`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet34EntityTeleport.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet34EntityTeleport` (`net/minecraft/src/Packet34EntityTeleport.java`) | **Líneas de código**: 61
- **Declaración**: `class Packet34EntityTeleport extends Packet`
- **Rol**: Paquete de datos de red: 'Packet34EntityTeleport' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet34EntityTeleport'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int entityId`
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `byte yaw`
  - `byte pitch`

##### <a id="class-packet38"></a>[`Packet38`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet38.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet38` (`net/minecraft/src/Packet38.java`) | **Líneas de código**: 39
- **Declaración**: `class Packet38 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet38' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet38'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int field_9274_a`
  - `byte field_9273_b`

##### <a id="class-packet39"></a>[`Packet39`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet39.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet39` (`net/minecraft/src/Packet39.java`) | **Líneas de código**: 39
- **Declaración**: `class Packet39 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet39' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet39'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `int getPacketSize()`
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
- **Campos Clave Destacados**:
  - `int field_6365_a`
  - `int field_6364_b`

##### <a id="class-packet4updatetime"></a>[`Packet4UpdateTime`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet4UpdateTime.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet4UpdateTime` (`net/minecraft/src/Packet4UpdateTime.java`) | **Líneas de código**: 36
- **Declaración**: `class Packet4UpdateTime extends Packet`
- **Rol**: Paquete de datos de red: 'Packet4UpdateTime' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet4UpdateTime'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `long time`

##### <a id="class-packet50prechunk"></a>[`Packet50PreChunk`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet50PreChunk.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet50PreChunk` (`net/minecraft/src/Packet50PreChunk.java`) | **Líneas de código**: 42
- **Declaración**: `class Packet50PreChunk extends Packet`
- **Rol**: Paquete de datos de red: 'Packet50PreChunk' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet50PreChunk'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `boolean mode`

##### <a id="class-packet51mapchunk"></a>[`Packet51MapChunk`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet51MapChunk.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet51MapChunk` (`net/minecraft/src/Packet51MapChunk.java`) | **Líneas de código**: 76
- **Declaración**: `class Packet51MapChunk extends Packet`
- **Rol**: Paquete de datos de red: 'Packet51MapChunk' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet51MapChunk'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `int xSize`
  - `int ySize`
  - `int zSize`

##### <a id="class-packet52multiblockchange"></a>[`Packet52MultiBlockChange`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet52MultiBlockChange.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet52MultiBlockChange` (`net/minecraft/src/Packet52MultiBlockChange.java`) | **Líneas de código**: 63
- **Declaración**: `class Packet52MultiBlockChange extends Packet`
- **Rol**: Paquete de datos de red: 'Packet52MultiBlockChange' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet52MultiBlockChange'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int zPosition`
  - `int size`

##### <a id="class-packet53blockchange"></a>[`Packet53BlockChange`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet53BlockChange.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet53BlockChange` (`net/minecraft/src/Packet53BlockChange.java`) | **Líneas de código**: 49
- **Declaración**: `class Packet53BlockChange extends Packet`
- **Rol**: Paquete de datos de red: 'Packet53BlockChange' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet53BlockChange'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `int type`
  - `int metadata`

##### <a id="class-packet59complexentity"></a>[`Packet59ComplexEntity`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet59ComplexEntity.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet59ComplexEntity` (`net/minecraft/src/Packet59ComplexEntity.java`) | **Líneas de código**: 69
- **Declaración**: `class Packet59ComplexEntity extends Packet`
- **Rol**: Paquete de datos de red: 'Packet59ComplexEntity' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet59ComplexEntity'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`
  - `NBTTagCompound entityNBT`

##### <a id="class-packet5playerinventory"></a>[`Packet5PlayerInventory`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet5PlayerInventory.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet5PlayerInventory` (`net/minecraft/src/Packet5PlayerInventory.java`) | **Líneas de código**: 75
- **Declaración**: `class Packet5PlayerInventory extends Packet`
- **Rol**: Paquete de datos de red: 'Packet5PlayerInventory' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet5PlayerInventory'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int type`

##### <a id="class-packet60"></a>[`Packet60`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet60.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet60` (`net/minecraft/src/Packet60.java`) | **Líneas de código**: 75
- **Declaración**: `class Packet60 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet60' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet60'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `double field_12236_a`
  - `double field_12235_b`
  - `double field_12239_c`
  - `float field_12238_d`
  - `Set field_12237_e`

##### <a id="class-packet6spawnposition"></a>[`Packet6SpawnPosition`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet6SpawnPosition.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet6SpawnPosition` (`net/minecraft/src/Packet6SpawnPosition.java`) | **Líneas de código**: 42
- **Declaración**: `class Packet6SpawnPosition extends Packet`
- **Rol**: Paquete de datos de red: 'Packet6SpawnPosition' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet6SpawnPosition'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int xPosition`
  - `int yPosition`
  - `int zPosition`

##### <a id="class-packet7"></a>[`Packet7`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet7.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet7` (`net/minecraft/src/Packet7.java`) | **Líneas de código**: 49
- **Declaración**: `class Packet7 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet7' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet7'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int field_9277_a`
  - `int field_9276_b`
  - `int field_9278_c`

##### <a id="class-packet8"></a>[`Packet8`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet8.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet8` (`net/minecraft/src/Packet8.java`) | **Líneas de código**: 36
- **Declaración**: `class Packet8 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet8' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet8'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `void processPacket(NetHandler nethandler)`
  - `int getPacketSize()`
- **Campos Clave Destacados**:
  - `int healthMP`

##### <a id="class-packet9"></a>[`Packet9`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Packet9.java)
- **Ruta y Paquete**: `net.minecraft.src.Packet9` (`net/minecraft/src/Packet9.java`) | **Líneas de código**: 32
- **Declaración**: `class Packet9 extends Packet`
- **Rol**: Paquete de datos de red: 'Packet9' en el protocolo SMP.
- **Descripción Técnica**: Serializa y deserializa información binaria a través de sockets TCP para la acción 'Packet9'.
- **Guía de Modificación**: Añade nuevos campos o sincronizaciones modificando readPacketData y writePacketData.
- **Métodos Clave Destacados**:
  - `void processPacket(NetHandler nethandler)`
  - `void readPacketData(DataInputStream datainputstream)`
  - `void writePacketData(DataOutputStream dataoutputstream)`
  - `int getPacketSize()`


### 11. Sistema NBT y Serialización (Named Binary Tag)
> **Resumen del subsistema**: El formato de archivo binario jerárquico ideado por Notch. Estructuras de datos tipadas en árbol (Byte, Short, Int, Long, Float, Double, ByteArray, String, List, Compound) con soporte de compresión y descompresión GZIP para guardar jugadores, mundos y chunks.

**Total de clases**: 13

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`CompressedStreamTools`](#class-compressedstreamtools) | `class` | `-` | 89 | Utilidades estáticas para lectura y escritura de estructuras NBT ... |
| [`NBTBase`](#class-nbtbase) | `class` | `-` | 149 | Nodo de almacenamiento serializable NBT: 'NBTBase'. |
| [`NBTTagByte`](#class-nbttagbyte) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagByte'. |
| [`NBTTagByteArray`](#class-nbttagbytearray) | `class` | `ext NBTBase` | 44 | Nodo de almacenamiento serializable NBT: 'NBTTagByteArray'. |
| [`NBTTagCompound`](#class-nbttagcompound) | `class` | `ext NBTBase` | 221 | Nodo de almacenamiento serializable NBT: 'NBTTagCompound'. |
| [`NBTTagDouble`](#class-nbttagdouble) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagDouble'. |
| [`NBTTagEnd`](#class-nbttagend) | `class` | `ext NBTBase` | 32 | Nodo de almacenamiento serializable NBT: 'NBTTagEnd'. |
| [`NBTTagFloat`](#class-nbttagfloat) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagFloat'. |
| [`NBTTagInt`](#class-nbttagint) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagInt'. |
| [`NBTTagList`](#class-nbttaglist) | `class` | `ext NBTBase` | 78 | Nodo de almacenamiento serializable NBT: 'NBTTagList'. |
| [`NBTTagLong`](#class-nbttaglong) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagLong'. |
| [`NBTTagShort`](#class-nbttagshort) | `class` | `ext NBTBase` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagShort'. |
| [`NBTTagString`](#class-nbttagstring) | `class` | `ext NBTBase` | 48 | Nodo de almacenamiento serializable NBT: 'NBTTagString'. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-compressedstreamtools"></a>[`CompressedStreamTools`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CompressedStreamTools.java)
- **Ruta y Paquete**: `net.minecraft.src.CompressedStreamTools` (`net/minecraft/src/CompressedStreamTools.java`) | **Líneas de código**: 89
- **Declaración**: `class CompressedStreamTools`
- **Rol**: Utilidades estáticas para lectura y escritura de estructuras NBT comprimidas con GZIP.
- **Descripción Técnica**: Serializa árboles NBT hacia flujos de datos o archivos en disco con compresión transparente.
- **Guía de Modificación**: Usar para guardar y cargar configuraciones personalizadas o estructuras en disco.
- **Métodos Clave Destacados**:
  - `NBTTagCompound func_1138_a(InputStream inputstream)`
  - `void writeGzippedCompoundToOutputStream(NBTTagCompound nbttagcompound, OutputStream outputstream)`
  - `NBTTagCompound func_1140_a(byte abyte0[])`
  - `byte[] func_1142_a(NBTTagCompound nbttagcompound)`
  - `NBTTagCompound func_1141_a(DataInput datainput)`
  - `void func_1139_a(NBTTagCompound nbttagcompound, DataOutput dataoutput)`

##### <a id="class-nbtbase"></a>[`NBTBase`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTBase.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTBase` (`net/minecraft/src/NBTBase.java`) | **Líneas de código**: 149
- **Declaración**: `class NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTBase'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTBase'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `String getKey()`
  - `NBTBase setKey(String s)`
  - `NBTBase readTag(DataInput datainput)`
  - `void writeTag(NBTBase nbtbase, DataOutput dataoutput)`
  - `NBTBase createTagOfType(byte byte0)`
  - `String getTagName(byte byte0)`

##### <a id="class-nbttagbyte"></a>[`NBTTagByte`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagByte.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagByte` (`net/minecraft/src/NBTTagByte.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagByte extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagByte'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagByte'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `byte byteValue`

##### <a id="class-nbttagbytearray"></a>[`NBTTagByteArray`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagByteArray.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagByteArray` (`net/minecraft/src/NBTTagByteArray.java`) | **Líneas de código**: 44
- **Declaración**: `class NBTTagByteArray extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagByteArray'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagByteArray'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`

##### <a id="class-nbttagcompound"></a>[`NBTTagCompound`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagCompound.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagCompound` (`net/minecraft/src/NBTTagCompound.java`) | **Líneas de código**: 221
- **Declaración**: `class NBTTagCompound extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagCompound'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagCompound'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `void setTag(String s, NBTBase nbtbase)`
  - `void setByte(String s, byte byte0)`
  - `void setShort(String s, short word0)`
  - `void setInteger(String s, int i)`
  - `void setLong(String s, long l)`

##### <a id="class-nbttagdouble"></a>[`NBTTagDouble`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagDouble.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagDouble` (`net/minecraft/src/NBTTagDouble.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagDouble extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagDouble'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagDouble'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `double doubleValue`

##### <a id="class-nbttagend"></a>[`NBTTagEnd`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagEnd.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagEnd` (`net/minecraft/src/NBTTagEnd.java`) | **Líneas de código**: 32
- **Declaración**: `class NBTTagEnd extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagEnd'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagEnd'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`

##### <a id="class-nbttagfloat"></a>[`NBTTagFloat`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagFloat.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagFloat` (`net/minecraft/src/NBTTagFloat.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagFloat extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagFloat'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagFloat'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `float floatValue`

##### <a id="class-nbttagint"></a>[`NBTTagInt`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagInt.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagInt` (`net/minecraft/src/NBTTagInt.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagInt extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagInt'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagInt'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `int intValue`

##### <a id="class-nbttaglist"></a>[`NBTTagList`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagList.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagList` (`net/minecraft/src/NBTTagList.java`) | **Líneas de código**: 78
- **Declaración**: `class NBTTagList extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagList'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagList'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
  - `void setTag(NBTBase nbtbase)`
  - `NBTBase tagAt(int i)`
  - `int tagCount()`

##### <a id="class-nbttaglong"></a>[`NBTTagLong`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagLong.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagLong` (`net/minecraft/src/NBTTagLong.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagLong extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagLong'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagLong'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `long longValue`

##### <a id="class-nbttagshort"></a>[`NBTTagShort`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagShort.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagShort` (`net/minecraft/src/NBTTagShort.java`) | **Líneas de código**: 41
- **Declaración**: `class NBTTagShort extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagShort'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagShort'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `short shortValue`

##### <a id="class-nbttagstring"></a>[`NBTTagString`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NBTTagString.java)
- **Ruta y Paquete**: `net.minecraft.src.NBTTagString` (`net/minecraft/src/NBTTagString.java`) | **Líneas de código**: 48
- **Declaración**: `class NBTTagString extends NBTBase`
- **Rol**: Nodo de almacenamiento serializable NBT: 'NBTTagString'.
- **Descripción Técnica**: Almacena datos en el formato binario jerárquico Named Binary Tag para 'NBTTagString'.
- **Guía de Modificación**: Utilizado para guardar variables personalizadas en mundos, jugadores y entidades.
- **Métodos Clave Destacados**:
  - `byte getType()`
  - `String toString()`
- **Campos Clave Destacados**:
  - `String stringValue`


### 12. Motor de Audio y Sonido (Sound Engine & Music)
> **Resumen del subsistema**: Integración con la librería Paulscode SoundSystem sobre OpenAL. Carga y reproduce efectos de sonido aleatorizados en tono/volumen (SoundPool), decodifica audio Ogg Vorbis (CodecJOrbis) y gestiona la música ambiental y de discos.

**Total de clases**: 5

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`CodecMus`](#class-codecmus) | `class` | `ext CodecJOrbis` | 26 | Decodificador de pistas de audio en formato .mus cifrado de C418. |
| [`MusInputStream`](#class-musinputstream) | `class` | `ext InputStream` | 55 | Flujo de lectura con descifrado al vuelo para archivos de música ... |
| [`SoundManager`](#class-soundmanager) | `class` | `-` | 244 | Gestor del sistema de audio y enlace con Paulscode SoundSystem (O... |
| [`SoundPool`](#class-soundpool) | `class` | `-` | 79 | Colección de clips de sonido indexados por categoría y nombre. |
| [`SoundPoolEntry`](#class-soundpoolentry) | `class` | `-` | 19 | Entrada individual de un archivo de sonido en un SoundPool. |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-codecmus"></a>[`CodecMus`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/CodecMus.java)
- **Ruta y Paquete**: `net.minecraft.src.CodecMus` (`net/minecraft/src/CodecMus.java`) | **Líneas de código**: 26
- **Declaración**: `class CodecMus extends CodecJOrbis`
- **Rol**: Decodificador de pistas de audio en formato .mus cifrado de C418.
- **Descripción Técnica**: Aplica una clave de descifrado XOR para convertir el flujo de datos a audio Ogg reproducible.
- **Guía de Modificación**: Necesario para decodificar las pistas de música oficiales del juego.
- **Métodos Clave Destacados**:
  - `InputStream openInputStream()`

##### <a id="class-musinputstream"></a>[`MusInputStream`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MusInputStream.java)
- **Ruta y Paquete**: `net.minecraft.src.MusInputStream` (`net/minecraft/src/MusInputStream.java`) | **Líneas de código**: 55
- **Declaración**: `class MusInputStream extends InputStream`
- **Rol**: Flujo de lectura con descifrado al vuelo para archivos de música .mus.
- **Descripción Técnica**: Aplica una máscara de clave pseudoaleatoria para descifrar el audio OGG de C418 en tiempo de ejecución.
- **Guía de Modificación**: Permite comprender el formato de empaquetado de música clásico de Mojang.
- **Métodos Clave Destacados**:
  - `int read()`
  - `int read(byte abyte0[], int i, int j)`

##### <a id="class-soundmanager"></a>[`SoundManager`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SoundManager.java)
- **Ruta y Paquete**: `net.minecraft.src.SoundManager` (`net/minecraft/src/SoundManager.java`) | **Líneas de código**: 244
- **Declaración**: `class SoundManager`
- **Rol**: Gestor del sistema de audio y enlace con Paulscode SoundSystem (OpenAL).
- **Descripción Técnica**: Inicializa OpenAL, reproduce música ambiental y emite efectos de sonido 3D con atenuación y paneo según la posición de la cámara.
- **Guía de Modificación**: Llama a playSoundFX(...) para emitir cualquier efecto de audio personalizado.
- **Métodos Clave Destacados**:
  - `void func_340_a(GameSettings gamesettings)`
  - `void onSoundOptionsChanged()`
  - `void closeMinecraft()`
  - `void func_6372_a(String s, File file)`
  - `void addStreaming(String s, File file)`
  - `void addMusic(String s, File file)`

##### <a id="class-soundpool"></a>[`SoundPool`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SoundPool.java)
- **Ruta y Paquete**: `net.minecraft.src.SoundPool` (`net/minecraft/src/SoundPool.java`) | **Líneas de código**: 79
- **Declaración**: `class SoundPool`
- **Rol**: Colección de clips de sonido indexados por categoría y nombre.
- **Descripción Técnica**: Mantiene listas de archivos de audio para cada acción (pasos, explosiones, golpes) y selecciona variaciones aleatorias de tono y volumen.
- **Guía de Modificación**: Permite añadir nuevos efectos de audio al repertorio del juego.
- **Métodos Clave Destacados**:
  - `SoundPoolEntry addSound(String s, File file)`
  - `SoundPoolEntry getRandomSoundFromSoundPool(String s)`
  - `SoundPoolEntry getRandomSound()`
- **Campos Clave Destacados**:
  - `int numberOfSoundPoolEntries`
  - `boolean field_1657_b`

##### <a id="class-soundpoolentry"></a>[`SoundPoolEntry`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/SoundPoolEntry.java)
- **Ruta y Paquete**: `net.minecraft.src.SoundPoolEntry` (`net/minecraft/src/SoundPoolEntry.java`) | **Líneas de código**: 19
- **Declaración**: `class SoundPoolEntry`
- **Rol**: Entrada individual de un archivo de sonido en un SoundPool.
- **Descripción Técnica**: Almacena el nombre del sonido y la URL del archivo de audio en disco.
- **Guía de Modificación**: Estructura interna del catálogo sonoro.
- **Campos Clave Destacados**:
  - `String soundName`
  - `URL soundUrl`


### 13. Matemáticas, Física y Estructuras de Datos (Math & Fast Collections)
> **Resumen del subsistema**: Utilidades matemáticas y físicas de alto rendimiento. Cajas delimitadoras alineadas con los ejes (AxisAlignedBB) con piscina de reciclaje de memoria, vectores 3D (Vec3D), tablas trigonométricas precalculadas (MathHelper), algoritmos de ruido Perlin y tablas hash primitivas (MCHashTable).

**Total de clases**: 11

| Clase | Tipo | Herencia / Interfaces | Líneas | Resumen Funcional |
|---|---|---|---|---|
| [`AxisAlignedBB`](#class-axisalignedbb) | `class` | `-` | 383 | Caja delimitadora rectangular alineada con los ejes (AABB) con pi... |
| [`HashEntry`](#class-hashentry) | `class` | `-` | 63 | Nodo interno para la tabla hash optimizada MCHashTable. |
| [`MCHashTable`](#class-mchashtable) | `class` | `-` | 166 | Tabla hash personalizada optimizada para claves de tipo entero pr... |
| [`MathHelper`](#class-mathhelper) | `class` | `-` | 86 | Tablas trigonométricas precalculadas y optimizaciones matemáticas... |
| [`MovingObjectPosition`](#class-movingobjectposition) | `class` | `-` | 34 | Contenedor del resultado de una intersección por raycast (trazado... |
| [`NoiseGenerator`](#class-noisegenerator) | `class` | `-` | 13 | Clase base abstracta para generadores de ruido procedural. |
| [`NoiseGenerator2`](#class-noisegenerator2) | `class` | `-` | 157 | Variante de generador de ruido procedural. |
| [`NoiseGeneratorOctaves`](#class-noisegeneratoroctaves) | `class` | `ext NoiseGenerator` | 68 | Generador de ruido fractal mediante suma de octavas de ruido Perl... |
| [`NoiseGeneratorOctaves2`](#class-noisegeneratoroctaves2) | `class` | `ext NoiseGenerator` | 58 | Generador fractal de octavas bidimensional. |
| [`NoiseGeneratorPerlin`](#class-noisegeneratorperlin) | `class` | `ext NoiseGenerator` | 228 | Implementación pura del algoritmo de Ruido Perlin tridimensional. |
| [`Vec3D`](#class-vec3d) | `class` | `-` | 206 | Vector espacial tridimensional para cálculos cinemáticos y de pos... |

#### Fichas Técnicas Detalladas del Subsistema

##### <a id="class-axisalignedbb"></a>[`AxisAlignedBB`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/AxisAlignedBB.java)
- **Ruta y Paquete**: `net.minecraft.src.AxisAlignedBB` (`net/minecraft/src/AxisAlignedBB.java`) | **Líneas de código**: 383
- **Declaración**: `class AxisAlignedBB`
- **Rol**: Caja delimitadora rectangular alineada con los ejes (AABB) con piscina de reciclaje de memoria.
- **Descripción Técnica**: Modela volúmenes 3D (minX, minY, minZ a maxX, maxY, maxZ) para colisiones y selecciones. Incluye un pool estático de instancias para evitar saturar el Garbage Collector de Java durante miles de operaciones físicas por segundo.
- **Guía de Modificación**: Usa siempre AxisAlignedBB.getBoundingBoxFromPool(...) en lugar de 'new' para preservar el rendimiento.
- **Métodos Clave Destacados**:
  - `AxisAlignedBB getBoundingBox(double d, double d1, double d2, double d3, double d4, double d5)`
  - `void clearBoundingBoxPool()`
  - `AxisAlignedBB getBoundingBoxFromPool(double d, double d1, double d2, double d3, double d4, double d5)`
  - `AxisAlignedBB setBounds(double d, double d1, double d2, double d3, double d4, double d5)`
  - `AxisAlignedBB addCoord(double d, double d1, double d2)`
  - `AxisAlignedBB expands(double d, double d1, double d2)`
- **Campos Clave Destacados**:
  - `double minX`
  - `double minY`
  - `double minZ`
  - `double maxX`
  - `double maxY`
  - `double maxZ`

##### <a id="class-hashentry"></a>[`HashEntry`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/HashEntry.java)
- **Ruta y Paquete**: `net.minecraft.src.HashEntry` (`net/minecraft/src/HashEntry.java`) | **Líneas de código**: 63
- **Declaración**: `class HashEntry`
- **Rol**: Nodo interno para la tabla hash optimizada MCHashTable.
- **Descripción Técnica**: Almacena la clave entera primitiva, el valor de referencia y el puntero al siguiente nodo colisionado.
- **Guía de Modificación**: Estructura interna de almacenamiento de alto rendimiento.
- **Métodos Clave Destacados**:
  - `int getHash()`
  - `Object getValue()`
  - `boolean equals(Object obj)`
  - `int hashCode()`
  - `String toString()`

##### <a id="class-mchashtable"></a>[`MCHashTable`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MCHashTable.java)
- **Ruta y Paquete**: `net.minecraft.src.MCHashTable` (`net/minecraft/src/MCHashTable.java`) | **Líneas de código**: 166
- **Declaración**: `class MCHashTable`
- **Rol**: Tabla hash personalizada optimizada para claves de tipo entero primitivo (int).
- **Descripción Técnica**: Evita la sobrecarga de boxing de Integer a int de las colecciones estándar de Java. Crucial en gestión de entidades y chunks.
- **Guía de Modificación**: Usar en mods para mapear IDs numéricos a objetos con mínimo coste de memoria y CPU.
- **Métodos Clave Destacados**:
  - `Object lookup(int i)`
  - `void addKey(int i, Object obj)`
  - `Object removeObject(int i)`
  - `void clearMap()`

##### <a id="class-mathhelper"></a>[`MathHelper`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MathHelper.java)
- **Ruta y Paquete**: `net.minecraft.src.MathHelper` (`net/minecraft/src/MathHelper.java`) | **Líneas de código**: 86
- **Declaración**: `class MathHelper`
- **Rol**: Tablas trigonométricas precalculadas y optimizaciones matemáticas.
- **Descripción Técnica**: Mantiene tablas estáticas de seno y coseno (65536 entradas) que sustituyen las costosas llamadas a java.lang.Math. Incluye floor_double y sqrt rápidos.
- **Guía de Modificación**: Usa MathHelper.sin() y MathHelper.cos() en cualquier cálculo de animación o física frecuente.
- **Métodos Clave Destacados**:
  - `float sin(float f)`
  - `float cos(float f)`
  - `float sqrt_float(float f)`
  - `float sqrt_double(double d)`
  - `int floor_float(float f)`
  - `int floor_double(double d)`

##### <a id="class-movingobjectposition"></a>[`MovingObjectPosition`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/MovingObjectPosition.java)
- **Ruta y Paquete**: `net.minecraft.src.MovingObjectPosition` (`net/minecraft/src/MovingObjectPosition.java`) | **Líneas de código**: 34
- **Declaración**: `class MovingObjectPosition`
- **Rol**: Contenedor del resultado de una intersección por raycast (trazado de rayos).
- **Descripción Técnica**: Almacena si la línea de visión o puntero impactó contra un bloque (coordenadas x, y, z y cara) o contra una entidad (entityHit).
- **Guía de Modificación**: Consultar este objeto para saber qué bloque o criatura tiene seleccionada el jugador.
- **Campos Clave Destacados**:
  - `int typeOfHit`
  - `int blockX`
  - `int blockY`
  - `int blockZ`
  - `int sideHit`
  - `Vec3D hitVec`

##### <a id="class-noisegenerator"></a>[`NoiseGenerator`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NoiseGenerator.java)
- **Ruta y Paquete**: `net.minecraft.src.NoiseGenerator` (`net/minecraft/src/NoiseGenerator.java`) | **Líneas de código**: 13
- **Declaración**: `class NoiseGenerator`
- **Rol**: Clase base abstracta para generadores de ruido procedural.
- **Descripción Técnica**: Base matemática para algoritmos de síntesis topográfica de terreno.
- **Guía de Modificación**: Extender para implementar nuevos algoritmos de ruido matemático.

##### <a id="class-noisegenerator2"></a>[`NoiseGenerator2`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NoiseGenerator2.java)
- **Ruta y Paquete**: `net.minecraft.src.NoiseGenerator2` (`net/minecraft/src/NoiseGenerator2.java`) | **Líneas de código**: 157
- **Declaración**: `class NoiseGenerator2`
- **Rol**: Variante de generador de ruido procedural.
- **Descripción Técnica**: Proporciona cálculos de modulación de terreno.
- **Guía de Modificación**: Utilizado en la generación de biomas.
- **Métodos Clave Destacados**:
  - `void func_4157_a(double ad[], double d, double d1, int i, int j, double d2, double d3, double d4)`
- **Campos Clave Destacados**:
  - `double field_4292_a`
  - `double field_4291_b`
  - `double field_4297_c`

##### <a id="class-noisegeneratoroctaves"></a>[`NoiseGeneratorOctaves`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NoiseGeneratorOctaves.java)
- **Ruta y Paquete**: `net.minecraft.src.NoiseGeneratorOctaves` (`net/minecraft/src/NoiseGeneratorOctaves.java`) | **Líneas de código**: 68
- **Declaración**: `class NoiseGeneratorOctaves extends NoiseGenerator`
- **Rol**: Generador de ruido fractal mediante suma de octavas de ruido Perlin.
- **Descripción Técnica**: Combina múltiples capas de diferente frecuencia y amplitud para producir orografía natural y montañas creíbles.
- **Guía de Modificación**: El componente más influyente en la forma de las cordilleras y valles de Minecraft.
- **Métodos Clave Destacados**:
  - `double func_806_a(double d, double d1)`
  - `double[] func_807_a(double ad[], double d, double d1, double d2, int i, int j, int k, double d3, double d4, double d5)`
  - `double[] func_4109_a(double ad[], int i, int j, int k, int l, double d, double d1, double d2)`

##### <a id="class-noisegeneratoroctaves2"></a>[`NoiseGeneratorOctaves2`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NoiseGeneratorOctaves2.java)
- **Ruta y Paquete**: `net.minecraft.src.NoiseGeneratorOctaves2` (`net/minecraft/src/NoiseGeneratorOctaves2.java`) | **Líneas de código**: 58
- **Declaración**: `class NoiseGeneratorOctaves2 extends NoiseGenerator`
- **Rol**: Generador fractal de octavas bidimensional.
- **Descripción Técnica**: Optimizado para cálculos de mapas de calor y humedad en el plano horizontal X-Z.
- **Guía de Modificación**: Determina la distribución geográfica de los biomas.
- **Métodos Clave Destacados**:
  - `double[] func_4112_a(double ad[], double d, double d1, int i, int j, double d2, double d3, double d4)`
  - `double[] func_4111_a(double ad[], double d, double d1, int i, int j, double d2, double d3, double d4, double d5)`

##### <a id="class-noisegeneratorperlin"></a>[`NoiseGeneratorPerlin`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/NoiseGeneratorPerlin.java)
- **Ruta y Paquete**: `net.minecraft.src.NoiseGeneratorPerlin` (`net/minecraft/src/NoiseGeneratorPerlin.java`) | **Líneas de código**: 228
- **Declaración**: `class NoiseGeneratorPerlin extends NoiseGenerator`
- **Rol**: Implementación pura del algoritmo de Ruido Perlin tridimensional.
- **Descripción Técnica**: Genera gradientes suaves de ruido a partir de una semilla numérica (seed).
- **Guía de Modificación**: El ladrillo fundamental del generador de mundos de Notch.
- **Métodos Clave Destacados**:
  - `double generateNoise(double d, double d1, double d2)`
  - `double lerp(double d, double d1, double d2)`
  - `double func_4110_a(int i, double d, double d1)`
  - `double grad(int i, double d, double d1, double d2)`
  - `double func_801_a(double d, double d1)`
  - `void func_805_a(double ad[], double d, double d1, double d2, int i, int j, int k, double d3, double d4, double d5, double d6)`
- **Campos Clave Destacados**:
  - `double xCoord`
  - `double yCoord`
  - `double zCoord`

##### <a id="class-vec3d"></a>[`Vec3D`](file:///C:/Users/ThinkPad/Documents/src_minecraft_a126/net/minecraft/src/Vec3D.java)
- **Ruta y Paquete**: `net.minecraft.src.Vec3D` (`net/minecraft/src/Vec3D.java`) | **Líneas de código**: 206
- **Declaración**: `class Vec3D`
- **Rol**: Vector espacial tridimensional para cálculos cinemáticos y de posición.
- **Descripción Técnica**: Modela posiciones y velocidades continuas (xCoord, yCoord, zCoord). Implementa suma, normalización, distancia euclidiana y producto escalar.
- **Guía de Modificación**: Utilizado masivamente en físicas, raycasting de selección y trayectorias balísticas.
- **Métodos Clave Destacados**:
  - `Vec3D createVectorHelper(double d, double d1, double d2)`
  - `void initialize()`
  - `Vec3D createVector(double d, double d1, double d2)`
  - `Vec3D subtract(Vec3D vec3d)`
  - `Vec3D normalize()`
  - `Vec3D crossProduct(Vec3D vec3d)`
- **Campos Clave Destacados**:
  - `double xCoord`
  - `double yCoord`
  - `double zCoord`


---
## 5. Índice Maestro Alfabético (Quick Lookup A-Z)
Catálogo completo de todas las 457 clases ordenadas alfabéticamente para búsqueda rápida:

| # | Clase | Subsistema | Tipo | Líneas | Resumen Arquitectónico |
|---|---|---|---|---|---|
| 1 | [`AxisAlignedBB`](#class-axisalignedbb) | Matemáticas, Física y Estructuras de Datos | `class` | 383 | Caja delimitadora rectangular alineada con los ejes (AABB) c... |
| 2 | [`Block`](#class-block) | Bloques y Materiales | `class` | 719 | Clase base abstracta de todos los bloques en el mundo voxel. |
| 3 | [`BlockBloodStone`](#class-blockbloodstone) | Bloques y Materiales | `class` | 14 | Netherrack / Piedra del Nether que mantiene el fuego perpetu... |
| 4 | [`BlockBookshelf`](#class-blockbookshelf) | Bloques y Materiales | `class` | 31 | Manejador del bloque 'BlockBookshelf' en el mundo voxel. |
| 5 | [`BlockBreakable`](#class-blockbreakable) | Bloques y Materiales | `class` | 34 | Manejador del bloque 'BlockBreakable' en el mundo voxel. |
| 6 | [`BlockButton`](#class-blockbutton) | Bloques y Materiales | `class` | 329 | Manejador del bloque 'BlockButton' en el mundo voxel. |
| 7 | [`BlockCactus`](#class-blockcactus) | Bloques y Materiales | `class` | 128 | Manejador del bloque de cactus con daño por espinas y crecim... |
| 8 | [`BlockChest`](#class-blockchest) | Bloques y Materiales | `class` | 273 | Bloque de cofre para almacenamiento persistente de objetos. |
| 9 | [`BlockClay`](#class-blockclay) | Bloques y Materiales | `class` | 25 | Manejador del bloque 'BlockClay' en el mundo voxel. |
| 10 | [`BlockContainer`](#class-blockcontainer) | Bloques y Materiales | `class` | 34 | Manejador del bloque 'BlockContainer' en el mundo voxel. |
| 11 | [`BlockCrops`](#class-blockcrops) | Bloques y Materiales | `class` | 138 | Cultivo de trigo por etapas de crecimiento (de 0 a 7). |
| 12 | [`BlockDirt`](#class-blockdirt) | Bloques y Materiales | `class` | 14 | Manejador del bloque 'BlockDirt' en el mundo voxel. |
| 13 | [`BlockDoor`](#class-blockdoor) | Bloques y Materiales | `class` | 250 | Mecánica de puertas de 2 bloques de altura con bisagras y ap... |
| 14 | [`BlockFence`](#class-blockfence) | Bloques y Materiales | `class` | 50 | Manejador del bloque 'BlockFence' en el mundo voxel. |
| 15 | [`BlockFire`](#class-blockfire) | Bloques y Materiales | `class` | 317 | Sistema de propagación de fuego, combustión y extinción. |
| 16 | [`BlockFlower`](#class-blockflower) | Bloques y Materiales | `class` | 74 | Manejador del bloque 'BlockFlower' en el mundo voxel. |
| 17 | [`BlockFlowing`](#class-blockflowing) | Bloques y Materiales | `class` | 322 | Manejador del bloque 'BlockFlowing' en el mundo voxel. |
| 18 | [`BlockFluids`](#class-blockfluids) | Bloques y Materiales | `class` | 349 | Manejador del bloque 'BlockFluids' en el mundo voxel. |
| 19 | [`BlockFurnace`](#class-blockfurnace) | Bloques y Materiales | `class` | 181 | Bloque de horno para fundición de minerales y cocinado de co... |
| 20 | [`BlockGlass`](#class-blockglass) | Bloques y Materiales | `class` | 20 | Manejador del bloque 'BlockGlass' en el mundo voxel. |
| 21 | [`BlockGrass`](#class-blockgrass) | Bloques y Materiales | `class` | 66 | Bloque de césped con propagación y degradación a tierra. |
| 22 | [`BlockGravel`](#class-blockgravel) | Bloques y Materiales | `class` | 26 | Manejador del bloque 'BlockGravel' en el mundo voxel. |
| 23 | [`BlockIce`](#class-blockice) | Bloques y Materiales | `class` | 50 | Manejador del bloque 'BlockIce' en el mundo voxel. |
| 24 | [`BlockJukeBox`](#class-blockjukebox) | Bloques y Materiales | `class` | 60 | Manejador del bloque 'BlockJukeBox' en el mundo voxel. |
| 25 | [`BlockLadder`](#class-blockladder) | Bloques y Materiales | `class` | 148 | Manejador del bloque 'BlockLadder' en el mundo voxel. |
| 26 | [`BlockLeaves`](#class-blockleaves) | Bloques y Materiales | `class` | 170 | Bloque de hojas de árbol con descomposición por distancia al... |
| 27 | [`BlockLeavesBase`](#class-blockleavesbase) | Bloques y Materiales | `class` | 34 | Manejador del bloque 'BlockLeavesBase' en el mundo voxel. |
| 28 | [`BlockLever`](#class-blocklever) | Bloques y Materiales | `class` | 289 | Manejador del bloque 'BlockLever' en el mundo voxel. |
| 29 | [`BlockLightStone`](#class-blocklightstone) | Bloques y Materiales | `class` | 20 | Glowstone / Piedra luminosa del Nether. |
| 30 | [`BlockLog`](#class-blocklog) | Bloques y Materiales | `class` | 35 | Bloque de tronco / madera natural de árbol. |
| 31 | [`BlockMinecartTrack`](#class-blockminecarttrack) | Bloques y Materiales | `class` | 135 | Manejador del bloque 'BlockMinecartTrack' en el mundo voxel. |
| 32 | [`BlockMobSpawner`](#class-blockmobspawner) | Bloques y Materiales | `class` | 35 | Manejador del bloque 'BlockMobSpawner' en el mundo voxel. |
| 33 | [`BlockMushroom`](#class-blockmushroom) | Bloques y Materiales | `class` | 26 | Manejador del bloque 'BlockMushroom' en el mundo voxel. |
| 34 | [`BlockObsidian`](#class-blockobsidian) | Bloques y Materiales | `class` | 25 | Manejador del bloque 'BlockObsidian' en el mundo voxel. |
| 35 | [`BlockOre`](#class-blockore) | Bloques y Materiales | `class` | 35 | Manejador del bloque 'BlockOre' en el mundo voxel. |
| 36 | [`BlockOreBlock`](#class-blockoreblock) | Bloques y Materiales | `class` | 20 | Manejador del bloque 'BlockOreBlock' en el mundo voxel. |
| 37 | [`BlockPortal`](#class-blockportal) | Bloques y Materiales | `class` | 207 | Bloque de portal dimensional al Inframundo (Nether). |
| 38 | [`BlockPressurePlate`](#class-blockpressureplate) | Bloques y Materiales | `class` | 194 | Manejador del bloque 'BlockPressurePlate' en el mundo voxel. |
| 39 | [`BlockPumpkin`](#class-blockpumpkin) | Bloques y Materiales | `class` | 91 | Manejador del bloque 'BlockPumpkin' en el mundo voxel. |
| 40 | [`BlockRedstoneOre`](#class-blockredstoneore) | Bloques y Materiales | `class` | 121 | Manejador del bloque 'BlockRedstoneOre' en el mundo voxel. |
| 41 | [`BlockRedstoneTorch`](#class-blockredstonetorch) | Bloques y Materiales | `class` | 224 | Antorcha de redstone que actúa como inversor lógico NOT y fu... |
| 42 | [`BlockRedstoneWire`](#class-blockredstonewire) | Bloques y Materiales | `class` | 402 | Cable conductor de energía de redstone con atenuación de 15 ... |
| 43 | [`BlockReed`](#class-blockreed) | Bloques y Materiales | `class` | 110 | Manejador del bloque 'BlockReed' en el mundo voxel. |
| 44 | [`BlockSand`](#class-blocksand) | Bloques y Materiales | `class` | 78 | Manejador del bloque 'BlockSand' en el mundo voxel. |
| 45 | [`BlockSapling`](#class-blocksapling) | Bloques y Materiales | `class` | 42 | Manejador del bloque 'BlockSapling' en el mundo voxel. |
| 46 | [`BlockSign`](#class-blocksign) | Bloques y Materiales | `class` | 136 | Manejador del bloque 'BlockSign' en el mundo voxel. |
| 47 | [`BlockSlowSand`](#class-blockslowsand) | Bloques y Materiales | `class` | 26 | Soul Sand / Arena de almas del Nether con reducción de veloc... |
| 48 | [`BlockSnow`](#class-blocksnow) | Bloques y Materiales | `class` | 110 | Manejador del bloque 'BlockSnow' en el mundo voxel. |
| 49 | [`BlockSnowBlock`](#class-blocksnowblock) | Bloques y Materiales | `class` | 35 | Manejador del bloque 'BlockSnowBlock' en el mundo voxel. |
| 50 | [`BlockSoil`](#class-blocksoil) | Bloques y Materiales | `class` | 133 | Manejador del bloque 'BlockSoil' en el mundo voxel. |
| 51 | [`BlockSponge`](#class-blocksponge) | Bloques y Materiales | `class` | 51 | Manejador del bloque 'BlockSponge' en el mundo voxel. |
| 52 | [`BlockStairs`](#class-blockstairs) | Bloques y Materiales | `class` | 233 | Manejador del bloque 'BlockStairs' en el mundo voxel. |
| 53 | [`BlockStationary`](#class-blockstationary) | Bloques y Materiales | `class` | 73 | Manejador del bloque 'BlockStationary' en el mundo voxel. |
| 54 | [`BlockStep`](#class-blockstep) | Bloques y Materiales | `class` | 91 | Manejador del bloque 'BlockStep' en el mundo voxel. |
| 55 | [`BlockStone`](#class-blockstone) | Bloques y Materiales | `class` | 20 | Manejador del bloque 'BlockStone' en el mundo voxel. |
| 56 | [`BlockTNT`](#class-blocktnt) | Bloques y Materiales | `class` | 65 | Bloque de dinamita activable por golpe o redstone. |
| 57 | [`BlockTorch`](#class-blocktorch) | Bloques y Materiales | `class` | 225 | Manejador del bloque 'BlockTorch' en el mundo voxel. |
| 58 | [`BlockWorkbench`](#class-blockworkbench) | Bloques y Materiales | `class` | 40 | Mesa de trabajo para crafteo en cuadrícula extendida de 3x3. |
| 59 | [`CanvasCrashReport`](#class-canvascrashreport) | Núcleo y Ciclo del Juego | `class` | 17 | Lienzo de dibujo para la pantalla de reporte de fallo (Crash... |
| 60 | [`CanvasIsomPreview`](#class-canvasisompreview) | Núcleo y Ciclo del Juego | `class` | 466 | Lienzo de dibujo del visor de mapas isométrico integrado en ... |
| 61 | [`CanvasMajongLogo`](#class-canvasmajonglogo) | Núcleo y Ciclo del Juego | `class` | 33 | Lienzo gráfico para dibujar el logotipo inicial de Mojang. |
| 62 | [`CanvasMinecraftApplet`](#class-canvasminecraftapplet) | Núcleo y Ciclo del Juego | `class` | 30 | Componente Canvas de AWT para el lienzo de renderizado del A... |
| 63 | [`ChatLine`](#class-chatline) | Interfaz de Usuario y Menús | `class` | 18 | Representación de una línea individual de texto en el histor... |
| 64 | [`Chunk`](#class-chunk) | Mundo, Chunks, Iluminación y Guardado | `class` | 691 | Segmento vertical del terreno de 16x16x128 bloques. |
| 65 | [`ChunkCache`](#class-chunkcache) | Mundo, Chunks, Iluminación y Guardado | `class` | 169 | Caché ligera de chunks adyacentes para operaciones rápidas d... |
| 66 | [`ChunkCoordIntPair`](#class-chunkcoordintpair) | Mundo, Chunks, Iluminación y Guardado | `class` | 29 | Par inmutable de coordenadas enteras (chunkX, chunkZ) para i... |
| 67 | [`ChunkCoordinates`](#class-chunkcoordinates) | Mundo, Chunks, Iluminación y Guardado | `class` | 35 | Tupla de coordenadas espaciales tridimensionales de bloque (... |
| 68 | [`ChunkLoader`](#class-chunkloader) | Mundo, Chunks, Iluminación y Guardado | `class` | 243 | Implementación de almacenamiento de chunks en disco en forma... |
| 69 | [`ChunkPosition`](#class-chunkposition) | Mundo, Chunks, Iluminación y Guardado | `class` | 37 | Contenedor ligero de posición tridimensional entera (x, y, z... |
| 70 | [`ChunkProviderClient`](#class-chunkproviderclient) | Mundo, Chunks, Iluminación y Guardado | `class` | 86 | Proveedor de chunks ficticio en el cliente multijugador. |
| 71 | [`ChunkProviderGenerate`](#class-chunkprovidergenerate) | Mundo, Chunks, Iluminación y Guardado | `class` | 609 | Generador procedural principal de terreno para el Overworld. |
| 72 | [`ChunkProviderHell`](#class-chunkproviderhell) | Mundo, Chunks, Iluminación y Guardado | `class` | 419 | Generador procedural del Inframundo (Nether). |
| 73 | [`ChunkProviderIso`](#class-chunkprovideriso) | Mundo, Chunks, Iluminación y Guardado | `class` | 87 | Proveedor de chunks optimizado para la vista isométrica del ... |
| 74 | [`ChunkProviderLoadOrGenerate`](#class-chunkproviderloadorgenerate) | Mundo, Chunks, Iluminación y Guardado | `class` | 240 | Orquestador de carga y generación de chunks para un jugador ... |
| 75 | [`ClippingHelper`](#class-clippinghelper) | Motor de Renderizado y Gráficos | `class` | 35 | Clase base abstracta para el cálculo del volumen de visión d... |
| 76 | [`ClippingHelperImplementation`](#class-clippinghelperimplementation) | Motor de Renderizado y Gráficos | `class` | 99 | Implementación concreta de ClippingHelper sobre OpenGL. |
| 77 | [`CodecMus`](#class-codecmus) | Motor de Audio y Sonido | `class` | 26 | Decodificador de pistas de audio en formato .mus cifrado de ... |
| 78 | [`ColorizerFoliage`](#class-colorizerfoliage) | Motor de Renderizado y Gráficos | `class` | 51 | Tabla de colores 256x256 para el follaje de hojas de árboles... |
| 79 | [`ColorizerGrass`](#class-colorizergrass) | Motor de Renderizado y Gráficos | `class` | 51 | Tabla de colores 256x256 para el césped superficial según bi... |
| 80 | [`CompressedStreamTools`](#class-compressedstreamtools) | Sistema NBT y Serialización | `class` | 89 | Utilidades estáticas para lectura y escritura de estructuras... |
| 81 | [`CraftingInventoryCB`](#class-craftinginventorycb) | Objetos, Crafteo e Inventarios | `class` | 31 | Contenedor abstracto base para interfaces con cuadrículas de... |
| 82 | [`CraftingInventoryPlayerCB`](#class-craftinginventoryplayercb) | Objetos, Crafteo e Inventarios | `class` | 57 | Contenedor para la cuadrícula de crafteo básica de 2x2 en el... |
| 83 | [`CraftingInventoryWorkbenchCB`](#class-craftinginventoryworkbenchcb) | Objetos, Crafteo e Inventarios | `class` | 55 | Contenedor para la mesa de trabajo de 3x3 (Workbench). |
| 84 | [`CraftingManager`](#class-craftingmanager) | Objetos, Crafteo e Inventarios | `class` | 229 | Gestor y registro central de recetas de crafteo del juego. |
| 85 | [`CraftingRecipe`](#class-craftingrecipe) | Objetos, Crafteo e Inventarios | `class` | 85 | Definición y validación de una receta de crafteo específica. |
| 86 | [`EffectRenderer`](#class-effectrenderer) | Motor de Renderizado y Gráficos | `class` | 196 | Gestor central del motor de partículas del juego. |
| 87 | [`Entity`](#class-entity) | Entidades, Mobs y Físicas | `class` | 1057 | Clase base abstracta de todas las entidades y objetos físico... |
| 88 | [`EntityAnimals`](#class-entityanimals) | Entidades, Mobs y Físicas | `class` | 48 | Entidad dinámica 'EntityAnimals' en el espacio 3D. |
| 89 | [`EntityArrow`](#class-entityarrow) | Entidades, Mobs y Físicas | `class` | 287 | Proyectil de flecha con física balística y anclaje en bloque... |
| 90 | [`EntityBoat`](#class-entityboat) | Entidades, Mobs y Físicas | `class` | 344 | Entidad dinámica 'EntityBoat' en el espacio 3D. |
| 91 | [`EntityBubbleFX`](#class-entitybubblefx) | Entidades, Mobs y Físicas | `class` | 46 | Entidad dinámica 'EntityBubbleFX' en el espacio 3D. |
| 92 | [`EntityChicken`](#class-entitychicken) | Entidades, Mobs y Físicas | `class` | 97 | Criatura pasiva de gallina que pone huevos y aletea al caer. |
| 93 | [`EntityClientPlayerMP`](#class-entityclientplayermp) | Entidades, Mobs y Físicas | `class` | 183 | Entidad dinámica 'EntityClientPlayerMP' en el espacio 3D. |
| 94 | [`EntityCow`](#class-entitycow) | Entidades, Mobs y Físicas | `class` | 67 | Criatura pasiva de vaca ordeñable con cubos vacíos. |
| 95 | [`EntityCreature`](#class-entitycreature) | Entidades, Mobs y Físicas | `class` | 170 | Entidad dinámica 'EntityCreature' en el espacio 3D. |
| 96 | [`EntityCreeper`](#class-entitycreeper) | Entidades, Mobs y Físicas | `class` | 153 | Mob hostil icónico: Creeper con detonación suicida por proxi... |
| 97 | [`EntityDiggingFX`](#class-entitydiggingfx) | Entidades, Mobs y Físicas | `class` | 60 | Entidad dinámica 'EntityDiggingFX' en el espacio 3D. |
| 98 | [`EntityExplodeFX`](#class-entityexplodefx) | Entidades, Mobs y Físicas | `class` | 49 | Entidad dinámica 'EntityExplodeFX' en el espacio 3D. |
| 99 | [`EntityFX`](#class-entityfx) | Entidades, Mobs y Físicas | `class` | 117 | Entidad dinámica 'EntityFX' en el espacio 3D. |
| 100 | [`EntityFallingSand`](#class-entityfallingsand) | Entidades, Mobs y Físicas | `class` | 102 | Entidad dinámica 'EntityFallingSand' en el espacio 3D. |
| 101 | [`EntityFireball`](#class-entityfireball) | Entidades, Mobs y Físicas | `class` | 237 | Entidad dinámica 'EntityFireball' en el espacio 3D. |
| 102 | [`EntityFish`](#class-entityfish) | Entidades, Mobs y Físicas | `class` | 390 | Entidad dinámica 'EntityFish' en el espacio 3D. |
| 103 | [`EntityFlameFX`](#class-entityflamefx) | Entidades, Mobs y Físicas | `class` | 71 | Entidad dinámica 'EntityFlameFX' en el espacio 3D. |
| 104 | [`EntityFlying`](#class-entityflying) | Entidades, Mobs y Físicas | `class` | 81 | Entidad dinámica 'EntityFlying' en el espacio 3D. |
| 105 | [`EntityGhast`](#class-entityghast) | Entidades, Mobs y Físicas | `class` | 173 | Mob hostil gigante volador del Inframundo (Nether). |
| 106 | [`EntityItem`](#class-entityitem) | Entidades, Mobs y Físicas | `class` | 219 | Entidad dinámica 'EntityItem' en el espacio 3D. |
| 107 | [`EntityLavaFX`](#class-entitylavafx) | Entidades, Mobs y Físicas | `class` | 65 | Entidad dinámica 'EntityLavaFX' en el espacio 3D. |
| 108 | [`EntityList`](#class-entitylist) | Entidades, Mobs y Físicas | `class` | 143 | Entidad dinámica 'EntityList' en el espacio 3D. |
| 109 | [`EntityLiving`](#class-entityliving) | Entidades, Mobs y Físicas | `class` | 867 | Clase base para criaturas vivientes (con salud, respiración ... |
| 110 | [`EntityMinecart`](#class-entityminecart) | Entidades, Mobs y Físicas | `class` | 855 | Entidad dinámica 'EntityMinecart' en el espacio 3D. |
| 111 | [`EntityMobs`](#class-entitymobs) | Entidades, Mobs y Físicas | `class` | 109 | Entidad dinámica 'EntityMobs' en el espacio 3D. |
| 112 | [`EntityOtherPlayerMP`](#class-entityotherplayermp) | Entidades, Mobs y Físicas | `class` | 112 | Entidad dinámica 'EntityOtherPlayerMP' en el espacio 3D. |
| 113 | [`EntityPainting`](#class-entitypainting) | Entidades, Mobs y Físicas | `class` | 238 | Entidad dinámica 'EntityPainting' en el espacio 3D. |
| 114 | [`EntityPickupFX`](#class-entitypickupfx) | Entidades, Mobs y Físicas | `class` | 65 | Entidad dinámica 'EntityPickupFX' en el espacio 3D. |
| 115 | [`EntityPig`](#class-entitypig) | Entidades, Mobs y Físicas | `class` | 64 | Criatura pasiva de cerdo montable con silla de montar. |
| 116 | [`EntityPigZombie`](#class-entitypigzombie) | Entidades, Mobs y Físicas | `class` | 126 | Hombre cerdo zombi neutral del Nether con agresión colectiva... |
| 117 | [`EntityPlayer`](#class-entityplayer) | Entidades, Mobs y Físicas | `class` | 342 | Representación del jugador dentro del mundo de Minecraft. |
| 118 | [`EntityPlayerSP`](#class-entityplayersp) | Entidades, Mobs y Físicas | `class` | 200 | Instancia del jugador local en el cliente (procesa entrada d... |
| 119 | [`EntityPortalFX`](#class-entityportalfx) | Entidades, Mobs y Físicas | `class` | 72 | Entidad dinámica 'EntityPortalFX' en el espacio 3D. |
| 120 | [`EntityRainFX`](#class-entityrainfx) | Entidades, Mobs y Físicas | `class` | 64 | Entidad dinámica 'EntityRainFX' en el espacio 3D. |
| 121 | [`EntityReddustFX`](#class-entityreddustfx) | Entidades, Mobs y Físicas | `class` | 74 | Entidad dinámica 'EntityReddustFX' en el espacio 3D. |
| 122 | [`EntityRenderer`](#class-entityrenderer) | Entidades, Mobs y Físicas | `class` | 641 | Entidad dinámica 'EntityRenderer' en el espacio 3D. |
| 123 | [`EntitySheep`](#class-entitysheep) | Entidades, Mobs y Físicas | `class` | 65 | Criatura pasiva de oveja con lana esquilable y tintada. |
| 124 | [`EntitySkeleton`](#class-entityskeleton) | Entidades, Mobs y Físicas | `class` | 93 | Mob hostil a distancia armado con arco y flechas. |
| 125 | [`EntitySlime`](#class-entityslime) | Entidades, Mobs y Físicas | `class` | 163 | Entidad dinámica 'EntitySlime' en el espacio 3D. |
| 126 | [`EntitySlimeFX`](#class-entityslimefx) | Entidades, Mobs y Físicas | `class` | 42 | Entidad dinámica 'EntitySlimeFX' en el espacio 3D. |
| 127 | [`EntitySmokeFX`](#class-entitysmokefx) | Entidades, Mobs y Físicas | `class` | 74 | Entidad dinámica 'EntitySmokeFX' en el espacio 3D. |
| 128 | [`EntitySnowball`](#class-entitysnowball) | Entidades, Mobs y Físicas | `class` | 270 | Entidad dinámica 'EntitySnowball' en el espacio 3D. |
| 129 | [`EntitySorter`](#class-entitysorter) | Entidades, Mobs y Físicas | `class` | 28 | Entidad dinámica 'EntitySorter' en el espacio 3D. |
| 130 | [`EntitySpider`](#class-entityspider) | Entidades, Mobs y Físicas | `class` | 91 | Mob hostil arácnido con capacidad para escalar paredes verti... |
| 131 | [`EntitySplashFX`](#class-entitysplashfx) | Entidades, Mobs y Físicas | `class` | 23 | Entidad dinámica 'EntitySplashFX' en el espacio 3D. |
| 132 | [`EntityTNTPrimed`](#class-entitytntprimed) | Entidades, Mobs y Físicas | `class` | 87 | Entidad dinámica 'EntityTNTPrimed' en el espacio 3D. |
| 133 | [`EntityZombie`](#class-entityzombie) | Entidades, Mobs y Físicas | `class` | 51 | Mob hostil cuerpo a cuerpo que arde bajo la luz del sol. |
| 134 | [`EntityZombieSimple`](#class-entityzombiesimple) | Entidades, Mobs y Físicas | `class` | 25 | Entidad dinámica 'EntityZombieSimple' en el espacio 3D. |
| 135 | [`EnumArt`](#class-enumart) | Objetos, Crafteo e Inventarios | `enum` | 119 | Enumeración de cuadros artísticos disponibles (pinturas deco... |
| 136 | [`EnumCreatureType`](#class-enumcreaturetype) | Entidades, Mobs y Físicas | `enum` | 45 | Clasificación biológica de criaturas (Monstruos, Criaturas p... |
| 137 | [`EnumMobType`](#class-enummobtype) | Entidades, Mobs y Físicas | `enum` | 43 | Clasificación del tipo de mob según comportamiento. |
| 138 | [`EnumOS1`](#class-enumos1) | Núcleo y Ciclo del Juego | `enum` | 49 | Enumeración de sistemas operativos compatibles (Windows, mac... |
| 139 | [`EnumOS2`](#class-enumos2) | Núcleo y Ciclo del Juego | `enum` | 50 | Enumeración auxiliar de variantes de sistemas operativos. |
| 140 | [`EnumOSMappingHelper`](#class-enumosmappinghelper) | Núcleo y Ciclo del Juego | `class` | 37 | Ayudante de mapeo entre cadenas de arquitectura y valores de... |
| 141 | [`EnumSkyBlock`](#class-enumskyblock) | Mundo, Chunks, Iluminación y Guardado | `enum` | 43 | Diferenciación de tipos de iluminación (Sky = Solar, Block =... |
| 142 | [`Explosion`](#class-explosion) | Mundo, Chunks, Iluminación y Guardado | `class` | 186 | Física de detonaciones con destrucción de bloques y vectores... |
| 143 | [`FontRenderer`](#class-fontrenderer) | Motor de Renderizado y Gráficos | `class` | 223 | Renderizador de fuentes bitmap tipográficas y códigos de for... |
| 144 | [`Frustrum`](#class-frustrum) | Motor de Renderizado y Gráficos | `class` | 37 | Verificador de inclusión espacial en el cono de visión de la... |
| 145 | [`GLAllocation`](#class-glallocation) | Motor de Renderizado y Gráficos | `class` | 76 | Gestor central de asignación de memoria nativa y recursos Op... |
| 146 | [`GameSettings`](#class-gamesettings) | Núcleo y Ciclo del Juego | `class` | 391 | Gestor del archivo de configuración options.txt y preferenci... |
| 147 | [`GameWindowListener`](#class-gamewindowlistener) | Núcleo y Ciclo del Juego | `class` | 35 | Oyente de eventos de la ventana gráfica de AWT/LWJGL. |
| 148 | [`Gui`](#class-gui) | Interfaz de Usuario y Menús | `class` | 91 | Pantalla o elemento de interfaz gráfica: 'Gui'. |
| 149 | [`GuiButton`](#class-guibutton) | Interfaz de Usuario y Menús | `class` | 93 | Pantalla o elemento de interfaz gráfica: 'GuiButton'. |
| 150 | [`GuiChat`](#class-guichat) | Interfaz de Usuario y Menús | `class` | 87 | Pantalla o elemento de interfaz gráfica: 'GuiChat'. |
| 151 | [`GuiChest`](#class-guichest) | Interfaz de Usuario y Menús | `class` | 69 | Pantalla del inventario de cofre simple (27 slots) o doble (... |
| 152 | [`GuiConflictWarning`](#class-guiconflictwarning) | Interfaz de Usuario y Menús | `class` | 52 | Pantalla o elemento de interfaz gráfica: 'GuiConflictWarning... |
| 153 | [`GuiConnectFailed`](#class-guiconnectfailed) | Interfaz de Usuario y Menús | `class` | 50 | Pantalla o elemento de interfaz gráfica: 'GuiConnectFailed'. |
| 154 | [`GuiConnecting`](#class-guiconnecting) | Interfaz de Usuario y Menús | `class` | 82 | Pantalla o elemento de interfaz gráfica: 'GuiConnecting'. |
| 155 | [`GuiContainer`](#class-guicontainer) | Interfaz de Usuario y Menús | `class` | 278 | Pantalla o elemento de interfaz gráfica: 'GuiContainer'. |
| 156 | [`GuiControls`](#class-guicontrols) | Interfaz de Usuario y Menús | `class` | 71 | Pantalla o elemento de interfaz gráfica: 'GuiControls'. |
| 157 | [`GuiCrafting`](#class-guicrafting) | Objetos, Crafteo e Inventarios | `class` | 65 | Pantalla o elemento de interfaz gráfica: 'GuiCrafting'. |
| 158 | [`GuiDeleteWorld`](#class-guideleteworld) | Interfaz de Usuario y Menús | `class` | 41 | Pantalla o elemento de interfaz gráfica: 'GuiDeleteWorld'. |
| 159 | [`GuiDownloadTerrain`](#class-guidownloadterrain) | Interfaz de Usuario y Menús | `class` | 52 | Pantalla o elemento de interfaz gráfica: 'GuiDownloadTerrain... |
| 160 | [`GuiEditSign`](#class-guieditsign) | Interfaz de Usuario y Menús | `class` | 119 | Pantalla o elemento de interfaz gráfica: 'GuiEditSign'. |
| 161 | [`GuiFurnace`](#class-guifurnace) | Interfaz de Usuario y Menús | `class` | 59 | Pantalla interactiva del horno de fundición. |
| 162 | [`GuiGameOver`](#class-guigameover) | Interfaz de Usuario y Menús | `class` | 62 | Pantalla o elemento de interfaz gráfica: 'GuiGameOver'. |
| 163 | [`GuiIngame`](#class-guiingame) | Interfaz de Usuario y Menús | `class` | 394 | Pantalla o elemento de interfaz gráfica: 'GuiIngame'. |
| 164 | [`GuiIngameMenu`](#class-guiingamemenu) | Interfaz de Usuario y Menús | `class` | 76 | Pantalla o elemento de interfaz gráfica: 'GuiIngameMenu'. |
| 165 | [`GuiInventory`](#class-guiinventory) | Interfaz de Usuario y Menús | `class` | 101 | Pantalla interactiva del inventario personal del jugador (te... |
| 166 | [`GuiMainMenu`](#class-guimainmenu) | Interfaz de Usuario y Menús | `class` | 251 | Pantalla del menú principal del juego. |
| 167 | [`GuiMultiplayer`](#class-guimultiplayer) | Interfaz de Usuario y Menús | `class` | 120 | Pantalla o elemento de interfaz gráfica: 'GuiMultiplayer'. |
| 168 | [`GuiOptions`](#class-guioptions) | Interfaz de Usuario y Menús | `class` | 70 | Pantalla o elemento de interfaz gráfica: 'GuiOptions'. |
| 169 | [`GuiScreen`](#class-guiscreen) | Interfaz de Usuario y Menús | `class` | 196 | Clase base para todas las pantallas interactivas de menús y ... |
| 170 | [`GuiSelectWorld`](#class-guiselectworld) | Interfaz de Usuario y Menús | `class` | 98 | Pantalla o elemento de interfaz gráfica: 'GuiSelectWorld'. |
| 171 | [`GuiSlider`](#class-guislider) | Interfaz de Usuario y Menús | `class` | 83 | Pantalla o elemento de interfaz gráfica: 'GuiSlider'. |
| 172 | [`GuiSmallButton`](#class-guismallbutton) | Interfaz de Usuario y Menús | `class` | 14 | Pantalla o elemento de interfaz gráfica: 'GuiSmallButton'. |
| 173 | [`GuiTexturePacks`](#class-guitexturepacks) | Motor de Renderizado y Gráficos | `class` | 240 | Pantalla o elemento de interfaz gráfica: 'GuiTexturePacks'. |
| 174 | [`GuiUnused`](#class-guiunused) | Interfaz de Usuario y Menús | `class` | 28 | Pantalla o elemento de interfaz gráfica: 'GuiUnused'. |
| 175 | [`GuiYesNo`](#class-guiyesno) | Interfaz de Usuario y Menús | `class` | 42 | Pantalla o elemento de interfaz gráfica: 'GuiYesNo'. |
| 176 | [`HashEntry`](#class-hashentry) | Matemáticas, Física y Estructuras de Datos | `class` | 63 | Nodo interno para la tabla hash optimizada MCHashTable. |
| 177 | [`IBlockAccess`](#class-iblockaccess) | Mundo, Chunks, Iluminación y Guardado | `interface` | 23 | Interfaz de solo lectura para consulta de bloques, metadatos... |
| 178 | [`ICamera`](#class-icamera) | Motor de Renderizado y Gráficos | `interface` | 13 | Interfaz para comprobaciones de visibilidad geométrica en la... |
| 179 | [`IChunkLoader`](#class-ichunkloader) | Mundo, Chunks, Iluminación y Guardado | `interface` | 20 | Interfaz para la persistencia de chunks en almacenamiento. |
| 180 | [`IChunkProvider`](#class-ichunkprovider) | Mundo, Chunks, Iluminación y Guardado | `interface` | 21 | Interfaz unificada para proveedores de chunks del mundo. |
| 181 | [`IInventory`](#class-iinventory) | TileEntities y Contenedores | `interface` | 23 | Interfaz estándar para cualquier contenedor con casillas de ... |
| 182 | [`IMobs`](#class-imobs) | Entidades, Mobs y Físicas | `interface` | 9 | Interfaz marcadora que identifica a una entidad como monstru... |
| 183 | [`IProgressUpdate`](#class-iprogressupdate) | Interfaz de Usuario y Menús | `interface` | 15 | Interfaz para receptores de progreso de tareas largas. |
| 184 | [`IWorldAccess`](#class-iworldaccess) | Mundo, Chunks, Iluminación y Guardado | `interface` | 29 | Interfaz de escucha para eventos que alteran el entorno del ... |
| 185 | [`ImageBuffer`](#class-imagebuffer) | Motor de Renderizado y Gráficos | `interface` | 12 | Interfaz para procesamiento de búferes de imágenes descargad... |
| 186 | [`ImageBufferDownload`](#class-imagebufferdownload) | Motor de Renderizado y Gráficos | `class` | 118 | Procesador de skins descargadas de internet. |
| 187 | [`InventoryCraftResult`](#class-inventorycraftresult) | Objetos, Crafteo e Inventarios | `class` | 59 | Inventario de 1 sola casilla que alberga el producto resulta... |
| 188 | [`InventoryCrafting`](#class-inventorycrafting) | Objetos, Crafteo e Inventarios | `class` | 82 | Inventario de entrada para la cuadrícula de fabricación (2x2... |
| 189 | [`InventoryLargeChest`](#class-inventorylargechest) | Objetos, Crafteo e Inventarios | `class` | 75 | Contenedor virtual que fusiona dos TileEntityChest en un cof... |
| 190 | [`InventoryPlayer`](#class-inventoryplayer) | Objetos, Crafteo e Inventarios | `class` | 502 | El inventario completo del jugador (44 casillas en total). |
| 191 | [`IsoImageBuffer`](#class-isoimagebuffer) | Motor de Renderizado y Gráficos | `class` | 44 | Búfer de píxeles para almacenar fragmentos del mapa isométri... |
| 192 | [`Item`](#class-item) | Objetos, Crafteo e Inventarios | `class` | 213 | Clase base de todos los objetos, herramientas y consumibles ... |
| 193 | [`ItemArmor`](#class-itemarmor) | Objetos, Crafteo e Inventarios | `class` | 32 | Piezas de armadura corporal (casco, peto, pantalones, botas)... |
| 194 | [`ItemAxe`](#class-itemaxe) | Objetos, Crafteo e Inventarios | `class` | 23 | Ítem especializado 'ItemAxe' en el sistema de inventario. |
| 195 | [`ItemBlock`](#class-itemblock) | Objetos, Crafteo e Inventarios | `class` | 68 | Ítem especializado 'ItemBlock' en el sistema de inventario. |
| 196 | [`ItemBoat`](#class-itemboat) | Objetos, Crafteo e Inventarios | `class` | 52 | Ítem especializado 'ItemBoat' en el sistema de inventario. |
| 197 | [`ItemBow`](#class-itembow) | Objetos, Crafteo e Inventarios | `class` | 29 | Arco para disparo balístico de flechas. |
| 198 | [`ItemBucket`](#class-itembucket) | Objetos, Crafteo e Inventarios | `class` | 119 | Ítem especializado 'ItemBucket' en el sistema de inventario. |
| 199 | [`ItemDoor`](#class-itemdoor) | Objetos, Crafteo e Inventarios | `class` | 83 | Ítem especializado 'ItemDoor' en el sistema de inventario. |
| 200 | [`ItemFishingRod`](#class-itemfishingrod) | Objetos, Crafteo e Inventarios | `class` | 45 | Ítem especializado 'ItemFishingRod' en el sistema de inventa... |
| 201 | [`ItemFlintAndSteel`](#class-itemflintandsteel) | Objetos, Crafteo e Inventarios | `class` | 53 | Ítem especializado 'ItemFlintAndSteel' en el sistema de inve... |
| 202 | [`ItemFood`](#class-itemfood) | Objetos, Crafteo e Inventarios | `class` | 25 | Alimento consumible que restaura salud directamente. |
| 203 | [`ItemHoe`](#class-itemhoe) | Objetos, Crafteo e Inventarios | `class` | 58 | Ítem especializado 'ItemHoe' en el sistema de inventario. |
| 204 | [`ItemMinecart`](#class-itemminecart) | Objetos, Crafteo e Inventarios | `class` | 35 | Ítem especializado 'ItemMinecart' en el sistema de inventari... |
| 205 | [`ItemPainting`](#class-itempainting) | Objetos, Crafteo e Inventarios | `class` | 47 | Ítem especializado 'ItemPainting' en el sistema de inventari... |
| 206 | [`ItemPickaxe`](#class-itempickaxe) | Objetos, Crafteo e Inventarios | `class` | 55 | Herramienta de pico para minar piedra y minerales. |
| 207 | [`ItemRecord`](#class-itemrecord) | Objetos, Crafteo e Inventarios | `class` | 32 | Ítem especializado 'ItemRecord' en el sistema de inventario. |
| 208 | [`ItemRedstone`](#class-itemredstone) | Objetos, Crafteo e Inventarios | `class` | 52 | Ítem especializado 'ItemRedstone' en el sistema de inventari... |
| 209 | [`ItemReed`](#class-itemreed) | Objetos, Crafteo e Inventarios | `class` | 66 | Ítem especializado 'ItemReed' en el sistema de inventario. |
| 210 | [`ItemRenderer`](#class-itemrenderer) | Objetos, Crafteo e Inventarios | `class` | 364 | Ítem especializado 'ItemRenderer' en el sistema de inventari... |
| 211 | [`ItemSaddle`](#class-itemsaddle) | Objetos, Crafteo e Inventarios | `class` | 34 | Ítem especializado 'ItemSaddle' en el sistema de inventario. |
| 212 | [`ItemSeeds`](#class-itemseeds) | Objetos, Crafteo e Inventarios | `class` | 35 | Ítem especializado 'ItemSeeds' en el sistema de inventario. |
| 213 | [`ItemSign`](#class-itemsign) | Objetos, Crafteo e Inventarios | `class` | 66 | Ítem especializado 'ItemSign' en el sistema de inventario. |
| 214 | [`ItemSnowball`](#class-itemsnowball) | Objetos, Crafteo e Inventarios | `class` | 27 | Ítem especializado 'ItemSnowball' en el sistema de inventari... |
| 215 | [`ItemSoup`](#class-itemsoup) | Objetos, Crafteo e Inventarios | `class` | 20 | Ítem especializado 'ItemSoup' en el sistema de inventario. |
| 216 | [`ItemSpade`](#class-itemspade) | Objetos, Crafteo e Inventarios | `class` | 32 | Ítem especializado 'ItemSpade' en el sistema de inventario. |
| 217 | [`ItemStack`](#class-itemstack) | Objetos, Crafteo e Inventarios | `class` | 164 | Instancia concreta de un ítem con cantidad y metadato/daño. |
| 218 | [`ItemSword`](#class-itemsword) | Objetos, Crafteo e Inventarios | `class` | 48 | Arma de combate cuerpo a cuerpo con bonificación de daño. |
| 219 | [`ItemTool`](#class-itemtool) | Objetos, Crafteo e Inventarios | `class` | 63 | Clase base para herramientas de trabajo (picos, hachas, pala... |
| 220 | [`KeyBinding`](#class-keybinding) | Núcleo y Ciclo del Juego | `class` | 18 | Asociación entre una acción de juego y una tecla física del ... |
| 221 | [`LoadingScreenRenderer`](#class-loadingscreenrenderer) | Motor de Renderizado y Gráficos | `class` | 160 | Renderizador de pantallas de carga y progreso. |
| 222 | [`LogoEffectRandomizer`](#class-logoeffectrandomizer) | Motor de Renderizado y Gráficos | `class` | 37 | Efecto de perturbación y dispersión procedural para el logo ... |
| 223 | [`MCHashTable`](#class-mchashtable) | Matemáticas, Física y Estructuras de Datos | `class` | 166 | Tabla hash personalizada optimizada para claves de tipo ente... |
| 224 | [`MapGenBase`](#class-mapgenbase) | Generación de Mundo y Biomas | `class` | 41 | Clase base para generadores de estructuras geológicas del ma... |
| 225 | [`MapGenCaves`](#class-mapgencaves) | Generación de Mundo y Biomas | `class` | 224 | Algoritmo procedural de tallado de redes de cuevas subterrán... |
| 226 | [`MapGenCavesHell`](#class-mapgencaveshell) | Generación de Mundo y Biomas | `class` | 209 | Generador de cavernas y bolsas de lava específico del Infram... |
| 227 | [`Material`](#class-material) | Bloques y Materiales | `class` | 71 | Definición de propiedades intrínsecas de la materia en el mu... |
| 228 | [`MaterialLiquid`](#class-materialliquid) | Bloques y Materiales | `class` | 28 | Especialización de Material para fluidos (agua y lava). |
| 229 | [`MaterialLogic`](#class-materiallogic) | Bloques y Materiales | `class` | 28 | Especialización de Material para componentes lógicos y circu... |
| 230 | [`MaterialTransparent`](#class-materialtransparent) | Bloques y Materiales | `class` | 28 | Especialización de Material para bloques transparentes que n... |
| 231 | [`MathHelper`](#class-mathhelper) | Matemáticas, Física y Estructuras de Datos | `class` | 86 | Tablas trigonométricas precalculadas y optimizaciones matemá... |
| 232 | [`MetadataChunkBlock`](#class-metadatachunkblock) | Mundo, Chunks, Iluminación y Guardado | `class` | 202 | Estructura auxiliar que empaqueta cambios de bloques en chun... |
| 233 | [`MinecartTrackLogic`](#class-minecarttracklogic) | Bloques y Materiales | `class` | 405 | Lógica geométrica de conexión y curvatura de raíles de vagon... |
| 234 | [`Minecraft`](#class-minecraft) | Núcleo y Ciclo del Juego | `class` | 1492 | El orquestador maestro y corazón del cliente de Minecraft. |
| 235 | [`MinecraftApplet`](#class-minecraftapplet) | Núcleo y Ciclo del Juego | `class` | 145 | Punto de entrada para ejecutar Minecraft dentro de un navega... |
| 236 | [`MinecraftAppletImpl`](#class-minecraftappletimpl) | Núcleo y Ciclo del Juego | `class` | 28 | Implementación concreta de Minecraft asociada a la ventana C... |
| 237 | [`MinecraftError`](#class-minecrafterror) | Núcleo y Ciclo del Juego | `class` | 13 | Error irrecuperable lanzado por el motor cuando ocurre un fa... |
| 238 | [`MinecraftException`](#class-minecraftexception) | Núcleo y Ciclo del Juego | `class` | 14 | Excepción general del juego para fallos de carga o guardado. |
| 239 | [`MinecraftImpl`](#class-minecraftimpl) | Núcleo y Ciclo del Juego | `class` | 27 | Implementación estándar de Minecraft para ejecución nativa e... |
| 240 | [`MobSpawnerBase`](#class-mobspawnerbase) | Entidades, Mobs y Físicas | `class` | 175 | Clase base para la ecología de spawn de criaturas en biomas. |
| 241 | [`MobSpawnerDesert`](#class-mobspawnerdesert) | Entidades, Mobs y Físicas | `class` | 13 | Definición ecológica para el bioma de Desierto. |
| 242 | [`MobSpawnerHell`](#class-mobspawnerhell) | Entidades, Mobs y Físicas | `class` | 17 | Definición ecológica para el Inframundo (Nether). |
| 243 | [`MobSpawnerSwamp`](#class-mobspawnerswamp) | Entidades, Mobs y Físicas | `class` | 13 | Definición ecológica para el bioma de Pantano. |
| 244 | [`ModelBase`](#class-modelbase) | Motor de Renderizado y Gráficos | `class` | 25 | Modelo geométrico cúbico 3D: 'ModelBase'. |
| 245 | [`ModelBiped`](#class-modelbiped) | Motor de Renderizado y Gráficos | `class` | 153 | Modelo geométrico cúbico 3D: 'ModelBiped'. |
| 246 | [`ModelBoat`](#class-modelboat) | Motor de Renderizado y Gráficos | `class` | 52 | Modelo geométrico cúbico 3D: 'ModelBoat'. |
| 247 | [`ModelChicken`](#class-modelchicken) | Motor de Renderizado y Gráficos | `class` | 75 | Modelo geométrico cúbico 3D: 'ModelChicken'. |
| 248 | [`ModelCow`](#class-modelcow) | Motor de Renderizado y Gráficos | `class` | 59 | Modelo geométrico cúbico 3D: 'ModelCow'. |
| 249 | [`ModelCreeper`](#class-modelcreeper) | Motor de Renderizado y Gráficos | `class` | 65 | Modelo geométrico cúbico 3D: 'ModelCreeper'. |
| 250 | [`ModelGhast`](#class-modelghast) | Motor de Renderizado y Gráficos | `class` | 55 | Modelo geométrico cúbico 3D: 'ModelGhast'. |
| 251 | [`ModelMinecart`](#class-modelminecart) | Motor de Renderizado y Gráficos | `class` | 57 | Modelo geométrico cúbico 3D: 'ModelMinecart'. |
| 252 | [`ModelPig`](#class-modelpig) | Motor de Renderizado y Gráficos | `class` | 19 | Modelo geométrico cúbico 3D: 'ModelPig'. |
| 253 | [`ModelQuadraped`](#class-modelquadraped) | Motor de Renderizado y Gráficos | `class` | 60 | Modelo geométrico cúbico 3D: 'ModelQuadraped'. |
| 254 | [`ModelRenderer`](#class-modelrenderer) | Motor de Renderizado y Gráficos | `class` | 206 | Modelo geométrico cúbico 3D: 'ModelRenderer'. |
| 255 | [`ModelSheep1`](#class-modelsheep1) | Motor de Renderizado y Gráficos | `class` | 33 | Modelo geométrico cúbico 3D: 'ModelSheep1'. |
| 256 | [`ModelSheep2`](#class-modelsheep2) | Motor de Renderizado y Gráficos | `class` | 20 | Modelo geométrico cúbico 3D: 'ModelSheep2'. |
| 257 | [`ModelSkeleton`](#class-modelskeleton) | Motor de Renderizado y Gráficos | `class` | 28 | Modelo geométrico cúbico 3D: 'ModelSkeleton'. |
| 258 | [`ModelSlime`](#class-modelslime) | Motor de Renderizado y Gráficos | `class` | 47 | Modelo geométrico cúbico 3D: 'ModelSlime'. |
| 259 | [`ModelSpider`](#class-modelspider) | Motor de Renderizado y Gráficos | `class` | 125 | Modelo geométrico cúbico 3D: 'ModelSpider'. |
| 260 | [`ModelZombie`](#class-modelzombie) | Motor de Renderizado y Gráficos | `class` | 32 | Modelo geométrico cúbico 3D: 'ModelZombie'. |
| 261 | [`MouseHelper`](#class-mousehelper) | Núcleo y Ciclo del Juego | `class` | 57 | Controlador de bajo nivel para el puntero del ratón. |
| 262 | [`MovementInput`](#class-movementinput) | Núcleo y Ciclo del Juego | `class` | 36 | Clase base de entrada de movimiento para el jugador. |
| 263 | [`MovementInputFromOptions`](#class-movementinputfromoptions) | Núcleo y Ciclo del Juego | `class` | 89 | Lee el estado físico del teclado según las opciones configur... |
| 264 | [`MovingObjectPosition`](#class-movingobjectposition) | Matemáticas, Física y Estructuras de Datos | `class` | 34 | Contenedor del resultado de una intersección por raycast (tr... |
| 265 | [`MusInputStream`](#class-musinputstream) | Motor de Audio y Sonido | `class` | 55 | Flujo de lectura con descifrado al vuelo para archivos de mú... |
| 266 | [`NBTBase`](#class-nbtbase) | Sistema NBT y Serialización | `class` | 149 | Nodo de almacenamiento serializable NBT: 'NBTBase'. |
| 267 | [`NBTTagByte`](#class-nbttagbyte) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagByte'. |
| 268 | [`NBTTagByteArray`](#class-nbttagbytearray) | Sistema NBT y Serialización | `class` | 44 | Nodo de almacenamiento serializable NBT: 'NBTTagByteArray'. |
| 269 | [`NBTTagCompound`](#class-nbttagcompound) | Sistema NBT y Serialización | `class` | 221 | Nodo de almacenamiento serializable NBT: 'NBTTagCompound'. |
| 270 | [`NBTTagDouble`](#class-nbttagdouble) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagDouble'. |
| 271 | [`NBTTagEnd`](#class-nbttagend) | Sistema NBT y Serialización | `class` | 32 | Nodo de almacenamiento serializable NBT: 'NBTTagEnd'. |
| 272 | [`NBTTagFloat`](#class-nbttagfloat) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagFloat'. |
| 273 | [`NBTTagInt`](#class-nbttagint) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagInt'. |
| 274 | [`NBTTagList`](#class-nbttaglist) | Sistema NBT y Serialización | `class` | 78 | Nodo de almacenamiento serializable NBT: 'NBTTagList'. |
| 275 | [`NBTTagLong`](#class-nbttaglong) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagLong'. |
| 276 | [`NBTTagShort`](#class-nbttagshort) | Sistema NBT y Serialización | `class` | 41 | Nodo de almacenamiento serializable NBT: 'NBTTagShort'. |
| 277 | [`NBTTagString`](#class-nbttagstring) | Sistema NBT y Serialización | `class` | 48 | Nodo de almacenamiento serializable NBT: 'NBTTagString'. |
| 278 | [`NetClientHandler`](#class-netclienthandler) | Redes y Protocolo Multijugador | `class` | 557 | Despachador central de paquetes de red en el cliente multiju... |
| 279 | [`NetHandler`](#class-nethandler) | Redes y Protocolo Multijugador | `class` | 185 | Manejador base de eventos del protocolo de paquetes de red. |
| 280 | [`NetworkManager`](#class-networkmanager) | Redes y Protocolo Multijugador | `class` | 239 | Gestor multihilo de sockets TCP para comunicaciones de red S... |
| 281 | [`NetworkMasterThread`](#class-networkmasterthread) | Redes y Protocolo Multijugador | `class` | 44 | Hilo maestro supervisor de la conexión de red TCP. |
| 282 | [`NetworkReaderThread`](#class-networkreaderthread) | Redes y Protocolo Multijugador | `class` | 36 | Hilo secundario para la recepción continua de paquetes de re... |
| 283 | [`NetworkWriterThread`](#class-networkwriterthread) | Redes y Protocolo Multijugador | `class` | 36 | Hilo secundario para la transmisión asíncrona de paquetes ha... |
| 284 | [`NextTickListEntry`](#class-nextticklistentry) | Mundo, Chunks, Iluminación y Guardado | `class` | 73 | Entrada individual en la cola de ticks programados de World. |
| 285 | [`NibbleArray`](#class-nibblearray) | Mundo, Chunks, Iluminación y Guardado | `class` | 54 | Array de memoria comprimida que almacena 2 valores de 4 bits... |
| 286 | [`NoiseGenerator`](#class-noisegenerator) | Matemáticas, Física y Estructuras de Datos | `class` | 13 | Clase base abstracta para generadores de ruido procedural. |
| 287 | [`NoiseGenerator2`](#class-noisegenerator2) | Matemáticas, Física y Estructuras de Datos | `class` | 157 | Variante de generador de ruido procedural. |
| 288 | [`NoiseGeneratorOctaves`](#class-noisegeneratoroctaves) | Matemáticas, Física y Estructuras de Datos | `class` | 68 | Generador de ruido fractal mediante suma de octavas de ruido... |
| 289 | [`NoiseGeneratorOctaves2`](#class-noisegeneratoroctaves2) | Matemáticas, Física y Estructuras de Datos | `class` | 58 | Generador fractal de octavas bidimensional. |
| 290 | [`NoiseGeneratorPerlin`](#class-noisegeneratorperlin) | Matemáticas, Física y Estructuras de Datos | `class` | 228 | Implementación pura del algoritmo de Ruido Perlin tridimensi... |
| 291 | [`OpenGlCapsChecker`](#class-openglcapschecker) | Motor de Renderizado y Gráficos | `class` | 23 | Verificador de capacidades y extensiones de la tarjeta gráfi... |
| 292 | [`OsMap`](#class-osmap) | Núcleo y Ciclo del Juego | `class` | 36 | Mapeo de opciones y rutas según el sistema operativo detecta... |
| 293 | [`Packet`](#class-packet) | Redes y Protocolo Multijugador | `class` | 152 | Paquete de datos de red: 'Packet' en el protocolo SMP. |
| 294 | [`Packet0KeepAlive`](#class-packet0keepalive) | Redes y Protocolo Multijugador | `class` | 31 | Paquete de datos de red: 'Packet0KeepAlive' en el protocolo ... |
| 295 | [`Packet10Flying`](#class-packet10flying) | Redes y Protocolo Multijugador | `class` | 49 | Paquete de datos de red: 'Packet10Flying' en el protocolo SM... |
| 296 | [`Packet11PlayerPosition`](#class-packet11playerposition) | Redes y Protocolo Multijugador | `class` | 48 | Paquete de datos de red: 'Packet11PlayerPosition' en el prot... |
| 297 | [`Packet12PlayerLook`](#class-packet12playerlook) | Redes y Protocolo Multijugador | `class` | 42 | Paquete de datos de red: 'Packet12PlayerLook' en el protocol... |
| 298 | [`Packet13PlayerLookMove`](#class-packet13playerlookmove) | Redes y Protocolo Multijugador | `class` | 56 | Paquete de datos de red: 'Packet13PlayerLookMove' en el prot... |
| 299 | [`Packet14BlockDig`](#class-packet14blockdig) | Redes y Protocolo Multijugador | `class` | 57 | Paquete de datos de red: 'Packet14BlockDig' en el protocolo ... |
| 300 | [`Packet15Place`](#class-packet15place) | Redes y Protocolo Multijugador | `class` | 57 | Paquete de datos de red: 'Packet15Place' en el protocolo SMP... |
| 301 | [`Packet16BlockItemSwitch`](#class-packet16blockitemswitch) | Redes y Protocolo Multijugador | `class` | 45 | Paquete de datos de red: 'Packet16BlockItemSwitch' en el pro... |
| 302 | [`Packet17AddToInventory`](#class-packet17addtoinventory) | Redes y Protocolo Multijugador | `class` | 42 | Paquete de datos de red: 'Packet17AddToInventory' en el prot... |
| 303 | [`Packet18ArmAnimation`](#class-packet18armanimation) | Redes y Protocolo Multijugador | `class` | 45 | Paquete de datos de red: 'Packet18ArmAnimation' en el protoc... |
| 304 | [`Packet1Login`](#class-packet1login) | Redes y Protocolo Multijugador | `class` | 55 | Paquete de datos de red: 'Packet1Login' en el protocolo SMP. |
| 305 | [`Packet20NamedEntitySpawn`](#class-packet20namedentityspawn) | Redes y Protocolo Multijugador | `class` | 70 | Paquete de datos de red: 'Packet20NamedEntitySpawn' en el pr... |
| 306 | [`Packet21PickupSpawn`](#class-packet21pickupspawn) | Redes y Protocolo Multijugador | `class` | 73 | Paquete de datos de red: 'Packet21PickupSpawn' en el protoco... |
| 307 | [`Packet22Collect`](#class-packet22collect) | Redes y Protocolo Multijugador | `class` | 39 | Paquete de datos de red: 'Packet22Collect' en el protocolo S... |
| 308 | [`Packet23VehicleSpawn`](#class-packet23vehiclespawn) | Redes y Protocolo Multijugador | `class` | 48 | Paquete de datos de red: 'Packet23VehicleSpawn' en el protoc... |
| 309 | [`Packet24MobSpawn`](#class-packet24mobspawn) | Redes y Protocolo Multijugador | `class` | 65 | Paquete de datos de red: 'Packet24MobSpawn' en el protocolo ... |
| 310 | [`Packet255KickDisconnect`](#class-packet255kickdisconnect) | Redes y Protocolo Multijugador | `class` | 41 | Paquete de datos de red: 'Packet255KickDisconnect' en el pro... |
| 311 | [`Packet28`](#class-packet28) | Redes y Protocolo Multijugador | `class` | 83 | Paquete de datos de red: 'Packet28' en el protocolo SMP. |
| 312 | [`Packet29DestroyEntity`](#class-packet29destroyentity) | Redes y Protocolo Multijugador | `class` | 36 | Paquete de datos de red: 'Packet29DestroyEntity' en el proto... |
| 313 | [`Packet2Handshake`](#class-packet2handshake) | Redes y Protocolo Multijugador | `class` | 41 | Paquete de datos de red: 'Packet2Handshake' en el protocolo ... |
| 314 | [`Packet30Entity`](#class-packet30entity) | Redes y Protocolo Multijugador | `class` | 43 | Paquete de datos de red: 'Packet30Entity' en el protocolo SM... |
| 315 | [`Packet31RelEntityMove`](#class-packet31relentitymove) | Redes y Protocolo Multijugador | `class` | 35 | Paquete de datos de red: 'Packet31RelEntityMove' en el proto... |
| 316 | [`Packet32EntityLook`](#class-packet32entitylook) | Redes y Protocolo Multijugador | `class` | 34 | Paquete de datos de red: 'Packet32EntityLook' en el protocol... |
| 317 | [`Packet33RelEntityMoveLook`](#class-packet33relentitymovelook) | Redes y Protocolo Multijugador | `class` | 40 | Paquete de datos de red: 'Packet33RelEntityMoveLook' en el p... |
| 318 | [`Packet34EntityTeleport`](#class-packet34entityteleport) | Redes y Protocolo Multijugador | `class` | 61 | Paquete de datos de red: 'Packet34EntityTeleport' en el prot... |
| 319 | [`Packet38`](#class-packet38) | Redes y Protocolo Multijugador | `class` | 39 | Paquete de datos de red: 'Packet38' en el protocolo SMP. |
| 320 | [`Packet39`](#class-packet39) | Redes y Protocolo Multijugador | `class` | 39 | Paquete de datos de red: 'Packet39' en el protocolo SMP. |
| 321 | [`Packet3Chat`](#class-packet3chat) | Interfaz de Usuario y Menús | `class` | 41 | Paquete de datos de red: 'Packet3Chat' en el protocolo SMP. |
| 322 | [`Packet4UpdateTime`](#class-packet4updatetime) | Redes y Protocolo Multijugador | `class` | 36 | Paquete de datos de red: 'Packet4UpdateTime' en el protocolo... |
| 323 | [`Packet50PreChunk`](#class-packet50prechunk) | Redes y Protocolo Multijugador | `class` | 42 | Paquete de datos de red: 'Packet50PreChunk' en el protocolo ... |
| 324 | [`Packet51MapChunk`](#class-packet51mapchunk) | Redes y Protocolo Multijugador | `class` | 76 | Paquete de datos de red: 'Packet51MapChunk' en el protocolo ... |
| 325 | [`Packet52MultiBlockChange`](#class-packet52multiblockchange) | Redes y Protocolo Multijugador | `class` | 63 | Paquete de datos de red: 'Packet52MultiBlockChange' en el pr... |
| 326 | [`Packet53BlockChange`](#class-packet53blockchange) | Redes y Protocolo Multijugador | `class` | 49 | Paquete de datos de red: 'Packet53BlockChange' en el protoco... |
| 327 | [`Packet59ComplexEntity`](#class-packet59complexentity) | Redes y Protocolo Multijugador | `class` | 69 | Paquete de datos de red: 'Packet59ComplexEntity' en el proto... |
| 328 | [`Packet5PlayerInventory`](#class-packet5playerinventory) | Redes y Protocolo Multijugador | `class` | 75 | Paquete de datos de red: 'Packet5PlayerInventory' en el prot... |
| 329 | [`Packet60`](#class-packet60) | Redes y Protocolo Multijugador | `class` | 75 | Paquete de datos de red: 'Packet60' en el protocolo SMP. |
| 330 | [`Packet6SpawnPosition`](#class-packet6spawnposition) | Redes y Protocolo Multijugador | `class` | 42 | Paquete de datos de red: 'Packet6SpawnPosition' en el protoc... |
| 331 | [`Packet7`](#class-packet7) | Redes y Protocolo Multijugador | `class` | 49 | Paquete de datos de red: 'Packet7' en el protocolo SMP. |
| 332 | [`Packet8`](#class-packet8) | Redes y Protocolo Multijugador | `class` | 36 | Paquete de datos de red: 'Packet8' en el protocolo SMP. |
| 333 | [`Packet9`](#class-packet9) | Redes y Protocolo Multijugador | `class` | 32 | Paquete de datos de red: 'Packet9' en el protocolo SMP. |
| 334 | [`PanelCrashReport`](#class-panelcrashreport) | Núcleo y Ciclo del Juego | `class` | 92 | Panel contenedor AWT para la presentación de informes de err... |
| 335 | [`Path`](#class-path) | Entidades, Mobs y Físicas | `class` | 144 | Encapsulador de una ruta de navegación calculada para criatu... |
| 336 | [`PathEntity`](#class-pathentity) | Entidades, Mobs y Físicas | `class` | 37 | Secuencia ordenada de puntos de ruta (PathPoint) asignada a ... |
| 337 | [`PathPoint`](#class-pathpoint) | Entidades, Mobs y Físicas | `class` | 58 | Nodo espacial tridimensional con coste de recorrido para el ... |
| 338 | [`Pathfinder`](#class-pathfinder) | Entidades, Mobs y Físicas | `class` | 218 | Motor de búsqueda de caminos sobre la malla de bloques naveg... |
| 339 | [`PlayerController`](#class-playercontroller) | Núcleo y Ciclo del Juego | `class` | 126 | Clase base abstracta para controladores de modo de juego del... |
| 340 | [`PlayerControllerMP`](#class-playercontrollermp) | Núcleo y Ciclo del Juego | `class` | 212 | Controlador del jugador para partidas multijugador. |
| 341 | [`PlayerControllerSP`](#class-playercontrollersp) | Núcleo y Ciclo del Juego | `class` | 147 | Controlador del modo Supervivencia para un solo jugador. |
| 342 | [`PlayerControllerTest`](#class-playercontrollertest) | Núcleo y Ciclo del Juego | `class` | 46 | Controlador de pruebas experimentales y modo creativo prelim... |
| 343 | [`PositionTexureVertex`](#class-positiontexurevertex) | Motor de Renderizado y Gráficos | `class` | 37 | Vértice espacial 3D con coordenadas UV de mapeado de textura... |
| 344 | [`RecipeSorter`](#class-recipesorter) | Objetos, Crafteo e Inventarios | `class` | 32 | Comparador que ordena las recetas en CraftingManager de mayo... |
| 345 | [`RecipesArmor`](#class-recipesarmor) | Objetos, Crafteo e Inventarios | `class` | 56 | Definición de recetas de piezas de armadura. |
| 346 | [`RecipesCrafting`](#class-recipescrafting) | Objetos, Crafteo e Inventarios | `class` | 26 | Definición de recetas básicas de construcción y utilidades. |
| 347 | [`RecipesFood`](#class-recipesfood) | Objetos, Crafteo e Inventarios | `class` | 23 | Definición de recetas de alimentos compuestos. |
| 348 | [`RecipesIngots`](#class-recipesingots) | Objetos, Crafteo e Inventarios | `class` | 40 | Definición de recetas de compactación y descompactación de l... |
| 349 | [`RecipesTools`](#class-recipestools) | Objetos, Crafteo e Inventarios | `class` | 56 | Definición de recetas de herramientas de trabajo. |
| 350 | [`RecipesWeapons`](#class-recipesweapons) | Objetos, Crafteo e Inventarios | `class` | 50 | Definición de recetas de espadas y armas. |
| 351 | [`RedstoneUpdateInfo`](#class-redstoneupdateinfo) | Bloques y Materiales | `class` | 22 | Registro de propagación para limitar actualizaciones cíclica... |
| 352 | [`Render`](#class-render) | Motor de Renderizado y Gráficos | `class` | 267 | Renderizador en espacio 3D para 'Render'. |
| 353 | [`RenderArrow`](#class-renderarrow) | Motor de Renderizado y Gráficos | `class` | 79 | Renderizador en espacio 3D para 'RenderArrow'. |
| 354 | [`RenderBiped`](#class-renderbiped) | Motor de Renderizado y Gráficos | `class` | 56 | Renderizador en espacio 3D para 'RenderBiped'. |
| 355 | [`RenderBlocks`](#class-renderblocks) | Motor de Renderizado y Gráficos | `class` | 2078 | Renderizador en espacio 3D para 'RenderBlocks'. |
| 356 | [`RenderBoat`](#class-renderboat) | Motor de Renderizado y Gráficos | `class` | 50 | Renderizador en espacio 3D para 'RenderBoat'. |
| 357 | [`RenderChicken`](#class-renderchicken) | Motor de Renderizado y Gráficos | `class` | 44 | Renderizador en espacio 3D para 'RenderChicken'. |
| 358 | [`RenderCow`](#class-rendercow) | Motor de Renderizado y Gráficos | `class` | 32 | Renderizador en espacio 3D para 'RenderCow'. |
| 359 | [`RenderCreeper`](#class-rendercreeper) | Motor de Renderizado y Gráficos | `class` | 68 | Renderizador en espacio 3D para 'RenderCreeper'. |
| 360 | [`RenderEngine`](#class-renderengine) | Motor de Renderizado y Gráficos | `class` | 469 | Renderizador en espacio 3D para 'RenderEngine'. |
| 361 | [`RenderEntity`](#class-renderentity) | Motor de Renderizado y Gráficos | `class` | 22 | Renderizador en espacio 3D para 'RenderEntity'. |
| 362 | [`RenderFallingSand`](#class-renderfallingsand) | Motor de Renderizado y Gráficos | `class` | 38 | Renderizador en espacio 3D para 'RenderFallingSand'. |
| 363 | [`RenderFireball`](#class-renderfireball) | Motor de Renderizado y Gráficos | `class` | 51 | Renderizador en espacio 3D para 'RenderFireball'. |
| 364 | [`RenderFish`](#class-renderfish) | Motor de Renderizado y Gráficos | `class` | 92 | Renderizador en espacio 3D para 'RenderFish'. |
| 365 | [`RenderGhast`](#class-renderghast) | Motor de Renderizado y Gráficos | `class` | 35 | Renderizador en espacio 3D para 'RenderGhast'. |
| 366 | [`RenderGlobal`](#class-renderglobal) | Motor de Renderizado y Gráficos | `class` | 1286 | Renderizador en espacio 3D para 'RenderGlobal'. |
| 367 | [`RenderHelper`](#class-renderhelper) | Motor de Renderizado y Gráficos | `class` | 63 | Renderizador en espacio 3D para 'RenderHelper'. |
| 368 | [`RenderItem`](#class-renderitem) | Motor de Renderizado y Gráficos | `class` | 218 | Renderizador en espacio 3D para 'RenderItem'. |
| 369 | [`RenderList`](#class-renderlist) | Motor de Renderizado y Gráficos | `class` | 87 | Renderizador en espacio 3D para 'RenderList'. |
| 370 | [`RenderLiving`](#class-renderliving) | Motor de Renderizado y Gráficos | `class` | 173 | Renderizador en espacio 3D para 'RenderLiving'. |
| 371 | [`RenderManager`](#class-rendermanager) | Motor de Renderizado y Gráficos | `class` | 134 | Renderizador en espacio 3D para 'RenderManager'. |
| 372 | [`RenderMinecart`](#class-renderminecart) | Motor de Renderizado y Gráficos | `class` | 95 | Renderizador en espacio 3D para 'RenderMinecart'. |
| 373 | [`RenderPainting`](#class-renderpainting) | Motor de Renderizado y Gráficos | `class` | 136 | Renderizador en espacio 3D para 'RenderPainting'. |
| 374 | [`RenderPig`](#class-renderpig) | Motor de Renderizado y Gráficos | `class` | 26 | Renderizador en espacio 3D para 'RenderPig'. |
| 375 | [`RenderPlayer`](#class-renderplayer) | Motor de Renderizado y Gráficos | `class` | 235 | Renderizador en espacio 3D para 'RenderPlayer'. |
| 376 | [`RenderSheep`](#class-rendersheep) | Motor de Renderizado y Gráficos | `class` | 26 | Renderizador en espacio 3D para 'RenderSheep'. |
| 377 | [`RenderSlime`](#class-renderslime) | Motor de Renderizado y Gráficos | `class` | 54 | Renderizador en espacio 3D para 'RenderSlime'. |
| 378 | [`RenderSnowball`](#class-rendersnowball) | Motor de Renderizado y Gráficos | `class` | 50 | Renderizador en espacio 3D para 'RenderSnowball'. |
| 379 | [`RenderSorter`](#class-rendersorter) | Motor de Renderizado y Gráficos | `class` | 50 | Renderizador en espacio 3D para 'RenderSorter'. |
| 380 | [`RenderSpider`](#class-renderspider) | Motor de Renderizado y Gráficos | `class` | 52 | Renderizador en espacio 3D para 'RenderSpider'. |
| 381 | [`RenderTNTPrimed`](#class-rendertntprimed) | Motor de Renderizado y Gráficos | `class` | 64 | Renderizador en espacio 3D para 'RenderTNTPrimed'. |
| 382 | [`RenderZombieSimple`](#class-renderzombiesimple) | Motor de Renderizado y Gráficos | `class` | 28 | Renderizador en espacio 3D para 'RenderZombieSimple'. |
| 383 | [`ScaledResolution`](#class-scaledresolution) | Motor de Renderizado y Gráficos | `class` | 32 | Calculador dinámico del factor de escala de la interfaz de u... |
| 384 | [`ScreenShotHelper`](#class-screenshothelper) | Interfaz de Usuario y Menús | `class` | 75 | Utilidad para capturar pantallas de juego y guardarlas en di... |
| 385 | [`Session`](#class-session) | Núcleo y Ciclo del Juego | `class` | 57 | Almacena las credenciales de la sesión activa del jugador. |
| 386 | [`SignModel`](#class-signmodel) | Motor de Renderizado y Gráficos | `class` | 26 | Modelo geométrico 3D del cartel (tablero y poste de soporte)... |
| 387 | [`Slot`](#class-slot) | TileEntities y Contenedores | `class` | 54 | Representación lógica y visual de una casilla individual en ... |
| 388 | [`SlotArmor`](#class-slotarmor) | TileEntities y Contenedores | `class` | 46 | Casilla especializada de inventario para piezas de armadura. |
| 389 | [`SlotCrafting`](#class-slotcrafting) | Objetos, Crafteo e Inventarios | `class` | 34 | Casilla especializada para el producto resultante de la fabr... |
| 390 | [`SlotInventory`](#class-slotinventory) | TileEntities y Contenedores | `class` | 30 | Casilla de inventario estándar sin restricciones de contenid... |
| 391 | [`SoundManager`](#class-soundmanager) | Motor de Audio y Sonido | `class` | 244 | Gestor del sistema de audio y enlace con Paulscode SoundSyst... |
| 392 | [`SoundPool`](#class-soundpool) | Motor de Audio y Sonido | `class` | 79 | Colección de clips de sonido indexados por categoría y nombr... |
| 393 | [`SoundPoolEntry`](#class-soundpoolentry) | Motor de Audio y Sonido | `class` | 19 | Entrada individual de un archivo de sonido en un SoundPool. |
| 394 | [`SpawnerAnimals`](#class-spawneranimals) | Entidades, Mobs y Físicas | `class` | 125 | Motor biológico de spawn periódico de criaturas. |
| 395 | [`StepSound`](#class-stepsound) | Bloques y Materiales | `class` | 40 | Efectos sonoros emitidos al caminar, colocar o destruir bloq... |
| 396 | [`StepSoundSand`](#class-stepsoundsand) | Bloques y Materiales | `class` | 19 | Variante de StepSound para arena, grava y nieve. |
| 397 | [`StepSoundStone`](#class-stepsoundstone) | Bloques y Materiales | `class` | 19 | Variante de StepSound para piedra, minerales y adoquín. |
| 398 | [`Teleporter`](#class-teleporter) | Mundo, Chunks, Iluminación y Guardado | `class` | 302 | Gestor cinemático de portales y translación de coordenadas e... |
| 399 | [`TerrainTextureManager`](#class-terraintexturemanager) | Motor de Renderizado y Gráficos | `class` | 268 | Administrador de atlas y texturas del terreno. |
| 400 | [`Tessellator`](#class-tessellator) | Motor de Renderizado y Gráficos | `class` | 355 | Motor de emisión geométrica de quads de alto rendimiento con... |
| 401 | [`TextureCompassFX`](#class-texturecompassfx) | Motor de Renderizado y Gráficos | `class` | 137 | Animación procedural para la esfera de la brújula en el inve... |
| 402 | [`TextureFX`](#class-texturefx) | Motor de Renderizado y Gráficos | `class` | 43 | Clase base para animaciones procedimentales de textura en me... |
| 403 | [`TextureFlamesFX`](#class-textureflamesfx) | Motor de Renderizado y Gráficos | `class` | 92 | Animación procedimental para las partículas y texturas de fu... |
| 404 | [`TextureLavaFX`](#class-texturelavafx) | Motor de Renderizado y Gráficos | `class` | 93 | Animación procedimental para el bloque de lava estacionaria. |
| 405 | [`TextureLavaFlowFX`](#class-texturelavaflowfx) | Motor de Renderizado y Gráficos | `class` | 97 | Animación procedimental para la lava en flujo continuo. |
| 406 | [`TexturePackBase`](#class-texturepackbase) | Motor de Renderizado y Gráficos | `class` | 45 | Clase base abstracta para paquetes de texturas (Texture Pack... |
| 407 | [`TexturePackCustom`](#class-texturepackcustom) | Motor de Renderizado y Gráficos | `class` | 138 | Paquete de texturas personalizado cargado desde un archivo .... |
| 408 | [`TexturePackDefault`](#class-texturepackdefault) | Motor de Renderizado y Gráficos | `class` | 55 | Paquete de texturas por defecto integrado en el cliente. |
| 409 | [`TexturePackList`](#class-texturepacklist) | Motor de Renderizado y Gráficos | `class` | 115 | Gestor del catálogo de paquetes de texturas disponibles en d... |
| 410 | [`TexturePortalFX`](#class-textureportalfx) | Motor de Renderizado y Gráficos | `class` | 100 | Animación procedimental para la lámina del portal al Inframu... |
| 411 | [`TextureWatchFX`](#class-texturewatchfx) | Motor de Renderizado y Gráficos | `class` | 105 | Animación procedural para la esfera del reloj en el inventar... |
| 412 | [`TextureWaterFX`](#class-texturewaterfx) | Motor de Renderizado y Gráficos | `class` | 99 | Animación procedimental para el bloque de agua estacionaria. |
| 413 | [`TexturedQuad`](#class-texturedquad) | Motor de Renderizado y Gráficos | `class` | 65 | Polígono cuadrilátero formado por 4 vértices PositionTexureV... |
| 414 | [`TexureWaterFlowFX`](#class-texurewaterflowfx) | Motor de Renderizado y Gráficos | `class` | 100 | Animación procedimental para el flujo horizontal del agua en... |
| 415 | [`ThreadConnectToServer`](#class-threadconnecttoserver) | Núcleo y Ciclo del Juego | `class` | 63 | Hilo asíncrono para negociar la conexión con un servidor mul... |
| 416 | [`ThreadDownloadImage`](#class-threaddownloadimage) | Núcleo y Ciclo del Juego | `class` | 55 | Hilo secundario para descargar imágenes de texturas en segun... |
| 417 | [`ThreadDownloadImageData`](#class-threaddownloadimagedata) | Núcleo y Ciclo del Juego | `class` | 23 | Contenedor y gestor de estado para imágenes descargadas asín... |
| 418 | [`ThreadDownloadResources`](#class-threaddownloadresources) | Núcleo y Ciclo del Juego | `class` | 163 | Hilo en segundo plano para descargar la música y efectos de ... |
| 419 | [`ThreadRunIsoClient`](#class-threadrunisoclient) | Núcleo y Ciclo del Juego | `class` | 29 | Hilo de procesamiento para la generación del mapa isométrico... |
| 420 | [`ThreadSleepForever`](#class-threadsleepforever) | Núcleo y Ciclo del Juego | `class` | 32 | Hilo utilitario que mantiene un hilo durmiendo indefinidamen... |
| 421 | [`TileEntity`](#class-tileentity) | TileEntities y Contenedores | `class` | 131 | Clase base abstracta para bloques con datos persistentes com... |
| 422 | [`TileEntityChest`](#class-tileentitychest) | TileEntities y Contenedores | `class` | 106 | Inventario de almacenamiento persistente para cofres (27 cas... |
| 423 | [`TileEntityFurnace`](#class-tileentityfurnace) | TileEntities y Contenedores | `class` | 293 | Lógica interna de cocinado y quemado de combustible del horn... |
| 424 | [`TileEntityMobSpawner`](#class-tileentitymobspawner) | TileEntities y Contenedores | `class` | 118 | Generador automático de monstruos con rotación visual intern... |
| 425 | [`TileEntityMobSpawnerRenderer`](#class-tileentitymobspawnerrenderer) | TileEntities y Contenedores | `class` | 51 | Renderizador 3D especial para el generador de monstruos (Mob... |
| 426 | [`TileEntityRenderer`](#class-tileentityrenderer) | TileEntities y Contenedores | `class` | 105 | Despachador central de renderizado para TileEntities. |
| 427 | [`TileEntitySign`](#class-tileentitysign) | TileEntities y Contenedores | `class` | 42 | Almacenamiento de las 4 líneas de texto del cartel. |
| 428 | [`TileEntitySignRenderer`](#class-tileentitysignrenderer) | TileEntities y Contenedores | `class` | 86 | Renderizador 3D para carteles y su texto. |
| 429 | [`TileEntitySpecialRenderer`](#class-tileentityspecialrenderer) | TileEntities y Contenedores | `class` | 34 | Clase base abstracta para renderizado 3D de TileEntities fue... |
| 430 | [`Timer`](#class-timer) | Núcleo y Ciclo del Juego | `class` | 68 | Sincronizador del reloj de juego (20 TPS) desacoplado de la ... |
| 431 | [`UnexpectedThrowable`](#class-unexpectedthrowable) | Núcleo y Ciclo del Juego | `class` | 18 | Envoltorio estructurado para excepciones imprevistas. |
| 432 | [`Vec3D`](#class-vec3d) | Matemáticas, Física y Estructuras de Datos | `class` | 206 | Vector espacial tridimensional para cálculos cinemáticos y d... |
| 433 | [`World`](#class-world) | Mundo, Chunks, Iluminación y Guardado | `class` | 2448 | El gestor integral del estado del mundo, físicas, iluminació... |
| 434 | [`WorldBlockPositionType`](#class-worldblockpositiontype) | Mundo, Chunks, Iluminación y Guardado | `class` | 28 | Tupla que asocia una posición tridimensional con el tipo de ... |
| 435 | [`WorldChunkManager`](#class-worldchunkmanager) | Mundo, Chunks, Iluminación y Guardado | `class` | 134 | Gestor climático y de biomas para el mundo Overworld. |
| 436 | [`WorldChunkManagerHell`](#class-worldchunkmanagerhell) | Mundo, Chunks, Iluminación y Guardado | `class` | 66 | Gestor de bioma único para el Inframundo (Nether). |
| 437 | [`WorldClient`](#class-worldclient) | Mundo, Chunks, Iluminación y Guardado | `class` | 262 | Implementación de World especializada para el cliente multij... |
| 438 | [`WorldGenBigTree`](#class-worldgenbigtree) | Generación de Mundo y Biomas | `class` | 450 | Generador de estructuras o recursos: 'WorldGenBigTree'. |
| 439 | [`WorldGenCactus`](#class-worldgencactus) | Generación de Mundo y Biomas | `class` | 39 | Generador superficial de columnas de cactus en biomas desért... |
| 440 | [`WorldGenClay`](#class-worldgenclay) | Generación de Mundo y Biomas | `class` | 69 | Generador de depósitos submarinos de arcilla. |
| 441 | [`WorldGenDungeons`](#class-worldgendungeons) | Generación de Mundo y Biomas | `class` | 214 | Generador de estructuras o recursos: 'WorldGenDungeons'. |
| 442 | [`WorldGenFire`](#class-worldgenfire) | Generación de Mundo y Biomas | `class` | 30 | Generador de fuegos eternos en el Inframundo (Nether). |
| 443 | [`WorldGenFlowers`](#class-worldgenflowers) | Generación de Mundo y Biomas | `class` | 33 | Generador de parches de flores amarillas y rosas rojas. |
| 444 | [`WorldGenHellLava`](#class-worldgenhelllava) | Generación de Mundo y Biomas | `class` | 79 | Generador de manantiales y cascadas de lava en los muros del... |
| 445 | [`WorldGenLakes`](#class-worldgenlakes) | Generación de Mundo y Biomas | `class` | 115 | Generador de lagos superficiales y subterráneos de agua o la... |
| 446 | [`WorldGenLightStone1`](#class-worldgenlightstone1) | Generación de Mundo y Biomas | `class` | 77 | Generador de estalactitas colgantes de piedra luminosa (Glow... |
| 447 | [`WorldGenLightStone2`](#class-worldgenlightstone2) | Generación de Mundo y Biomas | `class` | 77 | Segunda variante del generador de Glowstone en paredes y rec... |
| 448 | [`WorldGenLiquids`](#class-worldgenliquids) | Generación de Mundo y Biomas | `class` | 75 | Generador de pequeños manantiales de agua o lava que brotan ... |
| 449 | [`WorldGenMinable`](#class-worldgenminable) | Generación de Mundo y Biomas | `class` | 60 | Generador de estructuras o recursos: 'WorldGenMinable'. |
| 450 | [`WorldGenPumpkin`](#class-worldgenpumpkin) | Generación de Mundo y Biomas | `class` | 30 | Generador de calabazas silvestres en la superficie del Overw... |
| 451 | [`WorldGenReed`](#class-worldgenreed) | Generación de Mundo y Biomas | `class` | 39 | Generador de cañas de azúcar en orillas de ríos y costas. |
| 452 | [`WorldGenTrees`](#class-worldgentrees) | Generación de Mundo y Biomas | `class` | 96 | Generador de estructuras o recursos: 'WorldGenTrees'. |
| 453 | [`WorldGenerator`](#class-worldgenerator) | Generación de Mundo y Biomas | `class` | 20 | Generador de estructuras o recursos: 'WorldGenerator'. |
| 454 | [`WorldIso`](#class-worldiso) | Mundo, Chunks, Iluminación y Guardado | `class` | 23 | Mundo adaptado para la representación isométrica en el visor... |
| 455 | [`WorldProvider`](#class-worldprovider) | Mundo, Chunks, Iluminación y Guardado | `class` | 147 | Abstracción ambiental de dimensiones (Overworld vs Nether). |
| 456 | [`WorldProviderHell`](#class-worldproviderhell) | Mundo, Chunks, Iluminación y Guardado | `class` | 75 | Implementación ambiental específica del Inframundo (Nether). |
| 457 | [`WorldRenderer`](#class-worldrenderer) | Mundo, Chunks, Iluminación y Guardado | `class` | 282 | Compilador de fragmentos de terreno en listas de visualizaci... |
