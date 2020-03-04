# ultravuelo
Ultravuelo realista pokeemerald (tuto)


COMO FUNCIONA:

1) El sistema funciona en modo 2, pese a que solo he puesto el bg2. Esto es debido a que estube haciendo pruebas con dos bg (y por cierto se puede hacer perfectamente solo que el pokeemerald no tiene una funcion por defecto que haga rot/scale al bg3).

2) En mi ejemplo solo tengo 2 mapas insertados (1024x1024 cada uno y a escala 1:1), los mapas son de 256 colores en modo wrap. Los mapas estan en vertical (uno encima del otro).

3) El sistema de carga de mapas esta hecho para el movimiento en vertical, para el horizontal tambien pero tiene un pequeño bug debido a que la pantalla tiene muchos mas pixeles en horizontal que en vertical (tienes que cargar trozos de screenblocks y yo los cargo de uno en uno). La carga de mapas tanto en vertical como horizontal (los dos a la vez) NO LO TENGO HECHO.

4) Solo cargo un tileset. EL SISTEMA DE CARGA DE TILESETS NO LO TENGO HECHO.

5) El sistema para cargar los minis y destruirlos NO ESTA HECHO (por eso en la ruta 101 no aparece ningun mini).

6) El sistema de bajada (cuando terminas el ultravuelo) NO ESTA HECHO.

7) El sistema de reposicionamiento de minis esta completo (zoom, desplazamiento y movimiento) aunque con algun apanio.

8) El latios atraviesa todas las paredes, minis, warps y scripts (los pokemons salvajes tampoco pueden salir). Las animaciones de la hierba, huellas etc tampoco se activan.

9) Se baja pulsando el boton L y se sube pulsando el boton R. Cuando estas volando puedes volar de 0.5 de zoom a 0.25 de zoom 
(es decir 512 a 1024). Se sube y se baja con incrementos de 16. El Latios se mueve mas rapido como mas alto vuela.

10) La cinematica esta relantizada a posta, iba demasiado rapida y no me gustaba (aunque creo que me pase con lo lento jajaja).

11) Los minisprites son todos de 16 colores pero con el affine activado, es decir he puesto para los tres minis que salen sus
frames mirando a la derecha (si el affine esta activado no se puede hacer vhflip).

12) TODO EL CODIGO LO HE HECHO YO NO ESTA SACADO DE NINGUN SITIO.


-----


1) Programacion basica gba: modos de video.


Modos tiles:

Los modos tiles son los mas complicados: tienes una paleta (0x5000000), un tileset (hasta 4 charblocks, 0x4000 cada charblock) y un tilemap (hasta 32 screenblocks, 0x800 cada screenblock). Tanto el tileset como el tilemap se meten en la VRAM (0x6000000) (los charblocks y screenblocks comparten espacio es decir no puedes meter 4 charblock Y 32 screenblocks).


Modo 0: Este es el modo que usan los juegos de pokemon para los mapas.

	bg0, bg1, bg2 y bg3: no affine.


Modo 1:

	bg0, bg1: no affine.
	bg2: affine.
	bg3: no se puede usar.


Modo 2:

	bg0, bg1: no se pueden usar.
	bg2, bg3: affine.


Por lo tanto para el sistema de ultravuelo solo se puede usar modo 1 (un bg affine) o modo 2 (dos bg affine).

Los mapas affine solo pueden ser de 8bpp, no se puede usar vhflip y la forma como se meten en la VRAM es totalmente diferente que los mapas no affine.


Modos bitmap:

Los modos bitmap son los mas sencillos simplemente metes el pixel BGR de la imagen en la VRAM (0x6000000). La primera posicion es el pixel de arriba a la izquierda.


	Modo 3: 16bpp a pantalla completa (240x160).
	Modo 4: 8bpp a pantalla completa (240x160).
	Modo 5: 16bpp a "mitad pantalla" (160x128).



2) Funciones interesantes.


Primero tengo que decir que tengo el pokeemerald desactualizado espero que no haya cambios de nombres xd.


1.

	-bg.c
	-void SetBgAffine(u8 bg, s32 srcCenterX, s32 srcCenterY, s16 dispCenterX, s16 dispCenterY, s16 scaleX, s16 scaleY, u16 rotationAngle).


	Esta funcion sirve para el rot/scale del mapa affine.


	-bg: el bg que le haremos rot/scale (solo puede ser el dos, para hacerlo con el tres hay que hacer un pequeño cambio en la funcion).
	-srcCenterX: es el desplazamiento eje X del mapa (cada 256 es un pixel en zoom x1).
	-srcCenterY: es el desplazamiento eje Y del mapa (cada 256 es un pixel en zoom x1).
	-dispCenterX: es el pixel de anclaje de la pantalla eje X (120 es el centro de la pantalla, en el ultravuelo ese valor siempre es 120).
	-dispCenterY: es el pixel de anclaje de la pantalla eje Y (80 es el centro de la pantalla, en el ultravuelo ese valor siempre es 80).
	-scaleX: escalado del mapa eje X (64=x4, 128=x2, 256=x1, 512=x0.5, 1024=x0.25).
	-scaleY: escalado del mapa eje Y (64=x4, 128=x2, 256=x1, 512=x0.5, 1024=x0.25).
	-rotationAngle: angulo de rotacion (no me acuerdo exactamente de los numeros XDDD, pero es facil de comprobar).


2.

	-No me acuerdo el fichero donde estaba xd.
	-void SetAffineData(struct Sprite *sprite, s16 xScale, s16 yScale, u16 rotation).


	Esta funcion sirve para el rot/scale de los sprites en modo affine. Es una funcion static cuidado con esto (desconozco si hay una funcion parecida a esta que no sea static).


	-sprite: el sprite en modo affine.
	-xScale: escalado del sprite eje X (64=x4, 128=x2, 256=x1, 512=x0.5, 1024=x0.25).
	-yScale: escalado del mapa eje Y (64=x4, 128=x2, 256=x1, 512=x0.5, 1024=x0.25).
	-rotation: angulo de rotacion (no me acuerdo exactamente de los numeros XDDD, pero es facil de comprobar).


!!!Vigilad con el tamanio del oam (max 64x64) si haceis escalado x2 o x4 se puede ver cortado el sprite (en el caso del ultravuelo no problem xd).!!!

!!!El punto de anclaje del sprite creo que es el centro (facil de comprobar)!!!


3.

	-field_camera.c
	-void CameraUpdate(void).


	La utilidad de esta funcion es para saber hacia donde conio tenes que mover el mapa affine (srcCenterX y srcCenterY, los mapas affine usan unos registros diferentes para mover el mapa).


	-deltaX=-1 -> izquierda.
	-deltaX=1 -> derecha.
	-deltaY=-1 -> arriba.
	-deltaY=1 -> abajo.


!!!No se si hay una forma mejor de hacer esto (yo creo que si xd), pero de esta forma funciona.!!!


4.

	-field_camera.c
	-void FieldUpdateBgTilemapScroll(void).


	Esta funcion es la que se encarga de mover el mapa no affine, yo meti el sistema de ultravuelo aqui (con un bool para comprobar si estamos en modo ultravuelo), evidentemente se puede meter tambien en el fichero overworld.c


5.

	-sprite.c
	-void UpdateOamCoords(void).


	Actualiza la posicion de los sprites.

	Esta funcion es util para el reposicionamiento (desplazamiento) de los minis (lo explicare mas adelante con mas detalle).


6.

	-event_object_movement.c
	-void UpdateEventObjectCurrentMovement(struct EventObject *eventObject, struct Sprite *sprite, bool8 (*callback)(struct EventObject *, struct Sprite *)).
	-bool8 obj_npc_ministep(struct Sprite *sprite).


	Calcula los mini pasos que tiene que dar el mini.

	Estas dos funciones son utiles para el reposicionamiento (movimiento) de los minis (lo explicare mas adelante con mas detalle).


7.

	-overworld.c
	-void OverworldBasic(void).


	Se encarga de muchas tareas principales del juego (runscripts, runtasks, camaraupdate, buildoambuffer...).


8.

	-tileset_anims.c
	-void QueueAnimTiles_General_Flower(u16 timer).

	Animacion de la flor :)


3) Codigo: insertar minisprite affine (sustitucion).

Pequeño tuto para insertar los minisprites affine :)


	INFO. Los minisprites affine no pueden hacer uso del hvflip, es decir hay que insertar los frames (cuando mira a la derecha).

	INFO. Lon minis affine pueden ser de 16 colores sin problemas (al contrario de los bg affine).

	INFO. Los sprites que hay que modificar estan en \graphics\event_objects\pics\people\

	INFO. Yo los 3 frames los he insertado al final del spritesheet (esto es importante porque luego no os ira bien).

	INFO. Lo que haremos es SUSTITUIR un mini cualquiera (por ejemplo el fat_man) por el fat_man affine (no hay ningun problema en hacer esto ya que los minis affine tambien pueden ser de 16 colores).


!!!El ejemplo lo hare con el fat_man (16x32) si el mini tiene otro tamanio entonces el tuto varia un poco (no hace falta ser un lince para ver que es lo que hay que cambiar)!!!



1. Insertad los frames que faltan (los que miran hacia la izquierda les haceis hflip y los poneis al final) con el programa de edicion que querais y lo indexais (con la misma paleta que tenia!!!).

El spritesheet tiene que medir 192 pixel de largo. Lo meteis en la carpeta donde estaba (borrad el .4bpp para que al compilar (make) lo vuelva a generar).



2. Abrimos archivo event_object_graphics_info.h

Donde esta el fat_man (fila 19) sustituimos toda la fila por esto:

	const struct EventObjectGraphicsInfo gEventObjectGraphicsInfo_FatMan = {0xFFFF, EVENT_OBJ_PAL_TAG_0, EVENT_OBJ_PAL_TAG_NONE, 256, 16, 32, 2, SHADOW_SIZE_M, FALSE, FALSE, TRACKS_FOOT, &gEventObjectBaseOamAffine_16x32, gEventObjectSpriteOamTables_16x32, gEventObjectImageAnimTable_StandardAffine, gEventObjectPicTable_FatMan, gDummySpriteAffineAnimTable};

Lo que hemos hecho con esto es sustituir los structs por defecto por otros que vamos a crear a continuacion. 



3. Abrimos archivo event_object_pic_tables.h

Buscamos el struct "const struct SpriteFrameImage gEventObjectPicTable_FatMan[]" y lo sustituimos por esto:

	const struct SpriteFrameImage gEventObjectPicTable_FatMan[] = {

	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 0),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 1),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 2),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 3),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 4),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 5),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 6),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 7),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 8),
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 9),  //frame mira derecha
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 10), //frame anda derecha1
	    overworld_frame(gEventObjectPic_FatMan, 2, 4, 11), //frame anda derecha2

	};

Lo que hemos hecho aqui simplemente es anadir los frames.



	INFO: A PARTIR DE AQUI TODO LO QUE HAREMOS SOLO HACE FALTA HACERLO UNA VEZ.

4. Abrimos archivo base_oam.h

Nos vamos al final del archivo y copiamos esto:

	const struct OamData gEventObjectBaseOamAffine_16x32 = {

	    .affineMode = ST_OAM_AFFINE_NORMAL, //modo affine
	    .shape = SPRITE_SHAPE(16x32),
	    .size = SPRITE_SIZE(16x32),
	    .priority = 2

	};

Aqui lo que hacemos es decirle que el sprite sera affine.



5. Abrimos archivo event_object_anims.h

Os vais a la fila que querais (yo lo he puesto en la fila 798, pero podeis ponerlo antes del ultimo struct) y copiais esto:

	const union AnimCmd gEventObjectImageAnim_FaceEastAffine[] =
	{

	    ANIMCMD_FRAME(9, 16),
	    ANIMCMD_JUMP(0),

	};

	const union AnimCmd gEventObjectImageAnim_GoEastAffine[] =
	{

	    ANIMCMD_FRAME(10, 8),
	    ANIMCMD_FRAME(9, 8),
	    ANIMCMD_FRAME(11, 8),
	    ANIMCMD_FRAME(9, 8),
	    ANIMCMD_JUMP(0),

	};

	const union AnimCmd gEventObjectImageAnim_GoFastEastAffine[] =
	{

	    ANIMCMD_FRAME(10, 4),
	    ANIMCMD_FRAME(9, 4),
	    ANIMCMD_FRAME(11, 4),
	    ANIMCMD_FRAME(9, 4),
	    ANIMCMD_JUMP(0),

	};

	const union AnimCmd gEventObjectImageAnim_GoFasterEastAffine[] =
	{

	    ANIMCMD_FRAME(10, 2),
	    ANIMCMD_FRAME(9, 2),
	    ANIMCMD_FRAME(11, 2),
	    ANIMCMD_FRAME(9, 2),
	    ANIMCMD_JUMP(0),

	};

	const union AnimCmd gEventObjectImageAnim_GoFastestEastAffine[] =
	{

	    ANIMCMD_FRAME(10, 1),
	    ANIMCMD_FRAME(9, 1),
	    ANIMCMD_FRAME(11, 1),
	    ANIMCMD_FRAME(9, 1),
	    ANIMCMD_JUMP(0),

	};

	const union AnimCmd *const gEventObjectImageAnimTable_StandardAffine[] = {

	    gEventObjectImageAnim_FaceSouth,
	    gEventObjectImageAnim_FaceNorth,
	    gEventObjectImageAnim_FaceWest,
	    gEventObjectImageAnim_FaceEastAffine,
	    gEventObjectImageAnim_GoSouth,
	    gEventObjectImageAnim_GoNorth,
	    gEventObjectImageAnim_GoWest,
	    gEventObjectImageAnim_GoEastAffine,
	    gEventObjectImageAnim_GoFastSouth,
	    gEventObjectImageAnim_GoFastNorth,
	    gEventObjectImageAnim_GoFastWest,
	    gEventObjectImageAnim_GoFastEastAffine,
	    gEventObjectImageAnim_GoFasterSouth,
	    gEventObjectImageAnim_GoFasterNorth,
	    gEventObjectImageAnim_GoFasterWest,
	    gEventObjectImageAnim_GoFasterEastAffine,
	    gEventObjectImageAnim_GoFastestSouth,
	    gEventObjectImageAnim_GoFastestNorth,
	    gEventObjectImageAnim_GoFastestWest,
	    gEventObjectImageAnim_GoFastestEastAffine,

	};

Fijaos que esto sirve para la mayoria de minis (pero no todos).

En el ultimo struct del archivo "const struct UnkStruct_085094AC gUnknown_085094AC[]" (en mi pokeemerald se llama asi xd) lo sustituimos por esto:

	const struct UnkStruct_085094AC gUnknown_085094AC[] = {

	    {
		.anims = gEventObjectImageAnimTable_QuintyPlump,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_Standard,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_BrendanMayNormal,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_AcroBike,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_Surfing,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_Nurse,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_Fishing,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		.anims = gEventObjectImageAnimTable_StandardAffine,
		.animPos = {1, 3, 0, 2},
	    },
	    {
		NULL,
		{0, 0, 0, 0},
	    },

	};

Y con esto termina el tutorial.

La verdad es que no me acuerdo si era necesario hacer mas cosas (yo diria que no).




4) Codigo: reposicionamiento de minis.

Esto sin lugar a dudas es lo mas dificil.

	INFO. El sistema este lo he pensado yo (no lo he sacado de ningun sitio), por lo tanto no puedo asegurar que sea la mejor forma de hacerlo.

	INFO. Para hacer este sistema creo que se puede hacer con numeros decimales (ESTA OPCION NO LA HE EXPLORADO).

	INFO. En vez de numeros decimales yo lo he hecho con el resto de la division (a%b) a partir de ahora lo llamare modulo.

	INFO. Las formulas hay que aplicarlas tanto para el eje X como el eje Y. Yo solo pondre los ejemplos con el eje X.

	INFO IMPORTANTE: El reposicionamiento que yo he hecho funciona cuando el desplazamiento es mas rapido en funcion de la altura (cuanto mas alto vuelas mas rapido vas).


Formula final (no hare el desarrollo de todas las formulas):


	X = (D+M)/Z


	X: la distancia que hay que sumar (o restar) a la posicion actual del mini (es decir reposicionarlo).
	Cuando bajas: gSprites[i].pos1.x -= x
	Cuando subes: gSprites[i].pos1.x += x


	D: la distancia que hay entre el punto de anclaje del bg (120 en el caso de las X y 80 en el caso de las Y) y el punto de anclaje del mini (centro es decir +8 para las X y +16 para las Y).


	M: el modulo ANTERIOR (de la formula final).
	Para esto vamos a tener que guardar para cada mini un par de u8 (x,y).
	Para calcular M siguiente hay que hacer muchas operaciones mas debido a que tienen que cuadrar los modulos tanto cuando subes como cuando bajes.
	Esto hay que hacerlo antes de aplicar la formula:
	-Si estabas bajando y ahora subes cambias el signo del modulo.
	-Si estabas subiendo y ahora bajas cambias el signo del modulo.


	Z: zoom/incremento
	Es el numero de partes que hay entre pixeles.
	En mi caso el incremento es de 16:
	-porque queda bien xd.
	-porque luego al calcular los ministeps el calculo se simplifica mucho.
	
	

Codigo para subir y bajar (reposicionamiento zoom):


	//Primero:

	u8 i; //para el bucle minis affine
	s16 x; //X
	s16 y; //Y
	s16 num; //X+M o Y+M
	u8 den; //Z
	//esto es por la distinta velocidad en funcion de la altura
	bool8 L = FALSE;
	bool8 R = FALSE;

	//Si bajas (gMain.heldKeys & L_BUTTON) y zoom > 512:
	L = TRUE;
	zoom -= 16; //16 es el incremento
	den = zoom/16; //den es Z
	//aqui empieza el bucle que recorre todos los minis affine
	SetAffineData(&gSprites[i], zoom, zoom, 0);
	if (!bajando) {
		u8 i;
		//Cambiamos signo de los modulos (16 se supone que es el numero de minis pero yo solo tenia 3 XD???)
		for (i = 0; i<16; i++) {
			modX[i] = -modX[i];
			modY[i] = -modY[i];
		}
		bajando = TRUE;
	}

	//Si subes (gMain.heldKeys & R_BUTTON) y zoom < 1024:
	R = TRUE;
	zoom += 16;
	den = zoom/16;
	//bucle minis affine
	SetAffineData(&gSprites[i], zoom, zoom, 0);
	if (bajando) {
		u8 i;
		for (i = 0; i<16; i++) {
			modX[i] = -modX[i];
			modY[i] = -modY[i];
		}
		bajando = FALSE;
	}

	//A partir de aqui es lo mismo tanto si subes como si bajas

	//X
	x = ((gSprites[i].oam.x + 8));
	if (x > 380) x -= 512; //esto es por si quieres que el mini se siga reposicionando incluso fuera de pantalla (hasta un cierto margen)

	//Esta parte es debido a que al tener velocidad mayores a mas altura hay que hacer un apanio (luego explico)
	//Si no pones las distintas velocidades entonces esta parte no es necesaria.
	//ultravueloD es la direccion
	//ultravueloI son los ministeps
	if (ultravueloD > 0) {
		if (ultravueloD == 1) x += (ultravueloI*2 - 32);
		else if (ultravueloD == 2) x -= (ultravueloI*2 - 32);
	}

	//num es el numerador(D+M), den es el denominador(Z) y mod es el modulo(M)
	num = (120 - x) + modX[i];
	x = num/den; //aplicamos la formula
	//Redondeamos el numero Ej. 140px + 7(m) --> 141px - 3(m), donde Z=10 (es un ejemplo ficticio)
	//diria que se puede hacer mejor
	if ((num > 0) && ((num%den) >= ((den/2) + (den%2)))) {
		modX[i] = ((num%den) - den);
		x++;
	}
	else if ((num < 0) && ((-num%den) >= ((den/2) + (den%2)))) {
		modX[i] = ((num%den) + den);
		x--;
	}
	else modX[i] = num%den;                    

	//Y
	y = ((gSprites[i].oam.y + 16));
	if (y > 200) y -= 256;

	//Esto podria ir junto al x
	if (ultravueloD > 0) {
		if (ultravueloD == 3) y += (ultravueloI*2 - 32);
		else if (ultravueloD == 4) y -= (ultravueloI*2 - 32);
	}

	num = (80 - y) + modY[i];
	y = num/den;
	//Se repite el codigo xd
	if ((num > 0) && ((num%den) >= ((den/2) + (den%2)))) {
		modY[i] = ((num%den) - den);
		y++;
	}
	else if ((num < 0) && ((-num%den) >= ((den/2) + (den%2)))) {
		modY[i] = ((num%den) + den);
		y--;
	}
	else modY[i] = num%den;

	//Finalmente:

	//Si bajas (L):
	gSprites[i].pos1.x -= x;
	gSprites[i].pos1.y -= y;

	//Si subes (R):
	gSprites[i].pos1.x += x;
	gSprites[i].pos1.y += y;



Codigo desplazamiento (reposicionamiento desplazamiento):

	//se entra si ultravueloD > 0
	s16 correccion = 0;
	velocidad = zoom/16;
	//subes y bajas mientras te mueves (hay que corregir la velocidad para que los minis se posicionen de forma correcta)
	//ESTO ES UN APANIO ya que no supe como reposicionar los minis con distintas velocidades en funcion de la altura :(
	if (L || R) {
		//esto podria ser switch
		//esto se hace porque la velocidad es diferente en funcion de la altura
		//creo que seria mejor quitar lo de distintas velocidades en funcion de la altura ya que con este codigo lo que 		//hago es modificar la velocidad para que los minis se reposicionen bien
		//esta es la parte que se puede mejorar seguro
		if (ultravueloD == 1) correccion = (zoom*2*(15 - ultravueloI)) + (srcCenterX - srcCenterXAnt);
		else if (ultravueloD == 2) correccion = (zoom*2*(15 - ultravueloI)) - (srcCenterX - srcCenterXAnt);
		else if (ultravueloD == 3) correccion = (zoom*2*(15 - ultravueloI)) + (srcCenterY - srcCenterYAnt);
		else correccion = (zoom*2*(15 - ultravueloI)) - (srcCenterY - srcCenterYAnt);
	}

	//switch
	//se hacen un monton de operaciones innecesarias (se divide la velocidad entre 16 al principio y ahora se multiplica por 32 WTF XD????)
	//la variable velocidad ni es necesaria (podria se zoom*2 como hago arriba xd)
	if (ultravueloD == 1) srcCenterX -= (32*velocidad + correccion);
	else if (ultravueloD == 2) srcCenterX += (32*velocidad + correccion);
	else if (ultravueloD == 3) srcCenterY -= (32*velocidad + correccion);
	else srcCenterY += (32*velocidad + correccion);


	if (ultravueloI > 0) --ultravueloI;
	else {
		ultravueloI = 15;
		ultravueloD = 0;
		//esto es el apanio para que el reposicionamiento funcione cuando te mueves mientras subes y bajas
		//si no hubiera distintas velocidades no sera necesario
		srcCenterXAnt = srcCenterX;
		srcCenterYAnt = srcCenterY;
	}

	//esto va en la funcion UpdateOamCoords
	//si modo ultravuelo activado y el sprite es affine...
	
	sprite->oam.x = sprite->pos1.x + sprite->pos2.x + sprite->centerToCornerVecX + (gSpriteCoordOffsetX + gTotalCameraPixelOffsetX); //esto no tengo ni idea de porque funciona (puede que haya algun bug)

	sprite->oam.y = sprite->pos1.y + sprite->pos2.y + sprite->centerToCornerVecY + (gSpriteCoordOffsetY + gTotalCameraPixelOffsetY);


Codigo movimiento (reposicionamiento movimiento):

	//esta funcion es nueva (basada en obj_npc_ministep)
	//solo sirve si el incremento es 16
	bool8 obj_npc_ministep_affine(struct Sprite *sprite, u8 spriteId) {

	    s16 direccion = sprite->data[3]; //0-> nada, 1-> abajo, 2-> arriba, 3-> izquierda, 4-> derecha
	    s8 num = 16;
	    s8 modulo;

	    if (bajando) num = -16;

	    if (sprite->data[5] >= gUnknown_0850E768[sprite->data[4]]) return FALSE;

	    if ((direccion == 1) || (direccion == 2)) {
		if (direccion == 1) modY[spriteId] += num;
		else modY[spriteId] -= num;
		      //Aqui hago una cosa un poco rara, pero funciona :)
		modulo = modY[spriteId]%(zoom/16);
		if (modulo != modY[spriteId]) {
		    modY[spriteId] = modulo;
		    gUnknown_0850E754[sprite->data[4]][sprite->data[5]](sprite, sprite->data[3]);
		}
	    }
	    else if ((direccion == 3) || (direccion == 4)) {
		if (direccion == 3) modX[spriteId] -= num;
		else modX[spriteId] += num;
		modulo = modX[spriteId]%(zoom/16);
		if (modulo != modX[spriteId]) {
		    modX[spriteId] = modulo;
		    gUnknown_0850E754[sprite->data[4]][sprite->data[5]](sprite, sprite->data[3]);
		}
	    }

	    sprite->data[5]++;

	    if (sprite->data[5] < gUnknown_0850E768[sprite->data[4]]) return FALSE;

	    return TRUE;

	}

	//la funcion anterior la llamamos desde aqui
	//en la funcion npc_obj_ministep_stop_on_arrival
	//si modo ultravuelo activado y mini affine...
	if (obj_npc_ministep_affine(sprite, eventObject->spriteId)) {

		ShiftStillEventObjectCoords(eventObject);
		eventObject->triggerGroundEffectsOnStop = TRUE;
		sprite->animPaused = TRUE;
		return TRUE;

	}
	//si el modo ultravuelo no esta activado entonces haces lo mismo que ya habia en la funcion


Aplicar bg affine:

	//Finalmente ponemos esto (fuera del if ultravueloD > 0)
	SetBgAffineUltravuelo(2, srcCenterX, srcCenterY, 120, 80, zoom, zoom, 0); //solo el bg2

-------------------

Espero que se entienda el codigo de reposicionamiento.

Faltan muchas cosas como la flauta, insertar bg, cinematica, latios, eliminar scripts cuando vuelas, animaciones bg, la carga de mapas ETC.

Me gustaria decir que YO NO RECOMIENDO HACER EL SISTEMA DE ULTRAVUELO pero si hay alguien igual de zumbado que yo y quiere continuar esta mierda pues que lo haga :)
