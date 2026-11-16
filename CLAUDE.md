# Coffit App: memoria del proyecto

> **Para Claude:** lee este archivo al empezar cualquier chat del proyecto "CREAR APPS". Cuando cambies algo de la app, actualiza la sección que corresponda, agrega una línea al **Registro de cambios** con la fecha, y guarda el archivo en los tres lugares (ver "Dónde vive este archivo").
>
> Última actualización: 16 nov 2026, versión V6 (conexión con Supabase).

---

## 1. Qué es

La app de pedidos propia de **Coffit Cafetería** (Calle Libertad 316, Virú, La Libertad, Perú). El dueño es David, que trabaja en español, en Windows y sin terminal (usa Claude Code desde la app de escritorio).

**Objetivo de negocio:** +15–20 % de ventas en el próximo trimestre. Se busca con más ticket (combos y agregados), más frecuencia (repetir pedido, sellos) y más clientes (QR en mesa, recojo, delivery). Venta propia, sin las comisiones de 22–28 % de PedidosYa.

**Estado actual:** borrador público en prueba. Se mejorará de forma continua junto con el negocio antes de lanzarlo a clientes.

---

## 2. Accesos y servicios

| Servicio | Dato |
|---|---|
| App pública (Vercel) | https://coffit-app.vercel.app |
| Repositorio GitHub | `dvidvigo-spec/coffit-app` (público, rama `main`, un solo archivo `index.html`) |
| Vercel | Equipo "dvidvigo" (plan Hobby gratis). Proyecto `coffit-app`, preset "Other". Publica solo con cada commit a `main`. |
| Supabase | Organización "Coffit", proyecto `coffit-app`, ref `curlxsugmcjurewfngqm`, URL `https://curlxsugmcjurewfngqm.supabase.co`, plan gratis |
| Clave Supabase en la app | `sb_publishable_tJNmh5TnRt0gNB2Zo4C-kQ_Jnawdy9J` (clave publicable, segura en el navegador; la seguridad la da RLS). **Nunca** poner la `service_role`/secret key en la app. |
| WhatsApp oficial | **944 667 516** (`wa.me/51944667516`). Recibe los pedidos y el botón de contacto. El 941 112 979 de la carta ya NO es el oficial. |
| Vista previa en Claude | Artifact "Coffit Pide Aquí", https://claude.ai/artifact/C1Ec74V8fhmQCLt61ctMwL (ahí Supabase no conecta; usa la carta incluida). |

**Cómo publicar un cambio:**
- **Hoy:** David sube `index.html` a mano en GitHub (Add file → Upload files). El archivo está en `D:\POKERSTUDY\DvidVigo\DOCUMENTOS DE CLAUDE\CLAUDA_APP_VIRU\coffit-app\index.html`. Vercel publica solo.
- **Para que Claude suba los cambios directamente,** falta instalar la Claude GitHub App en el repo: https://github.com/apps/claude/installations/select_target. Con eso, Claude clona con `add_repo`, hace commit y push.
- Por seguridad, la subida de archivos desde Chrome la tiene que hacer David. Los códigos de verificación y las aprobaciones de permisos (OAuth) también los hace él.

---

## 3. Base de datos (Supabase)

**`public.productos`**: la carta. Hoy tiene 188 filas.
`id int PK · categoria · nombre · precio numeric · precio_grande numeric (16 oz o torta entera) · descripcion · orden · activo bool · actualizado`

**`public.pedidos`**: cada pedido enviado desde la app.
`id identity · creado · modo ('mesa'|'recojo'|'delivery') · nombre · celular · mesa · hora_recojo · direccion · referencia · pago · paga_con · subtotal · delivery · descuento · total · items jsonb [{id,nombre,detalle,nota,precio,cantidad}] · estado ('nuevo'|'preparando'|'en camino'|'entregado'|'cancelado')`

**Seguridad (RLS activado):**
- `productos`: la clave anónima puede leer solo los productos con `activo = true`.
- `pedidos`: la clave anónima solo puede **insertar** (con total entre 0 y 5000, y de 1 a 60 ítems). **Nadie de afuera puede leer los pedidos**; se ven en el panel de Supabase, en Table Editor → pedidos.

**Para cambiar precios u ocultar productos sin tocar código:** en Supabase, Table Editor → `productos`, editar `precio` o poner `activo = false`. La app toma los cambios al abrirse.

El SQL inicial está guardado en el editor SQL de Supabase. Lo probé el 16 nov: leer la carta y crear un pedido funcionan, leer pedidos queda bloqueado, y el pedido de prueba se borró.

---

## 4. Cómo está hecha la app (`index.html`, ~1.5 MB)

Es un solo archivo HTML con CSS y JS propios, sin frameworks. Trae dentro:
- `MENU`: la carta en JSON, que sirve de respaldo si Supabase no responde. Al abrir, `syncMenu()` la reemplaza con los datos de `productos`.
- `PHOTOS`: 51 fotos en base64 (360 px). `PIDX` asigna el nombre del producto a su foto. Hoy 62 productos tienen foto.
- El logo real (wordmark y sello) en base64, en el color espresso.

**Pantallas:**
- **Inicio:** modo (En mesa / Para recoger / Delivery), buscador, promo del día, grilla de 9 secciones con foto y "Lo más pedido".
- **Carta:** pestañas fijas por sección, subsecciones que se marcan al bajar, y "+" que agrega directo o se vuelve contador.
- **Club:** sellos (8 sellos = café gratis; hoy solo se guardan en el celular), tortas enteras, Miércoles Coffit y pedidos corporativos.
- **Pedido:** ofertas sugeridas, barra de delivery gratis, datos del cliente, pago Yape/Plin o efectivo. Al enviar, el pedido se guarda en Supabase (`saveOrder`, sin esperar respuesta) y se abre WhatsApp con el pedido.
- **Botón flotante de WhatsApp** (abajo a la izquierda, con rebote).

**Diseño:** color caramelo #B98D62 (protagonista), espresso #2A1D16 (texto y botones), leche #FBF7F2 (fondo) y verde hoja #5E7B4C (solo para ahorro, pedido y logros). Fuentes Marcellus (títulos) y Figtree (texto). Se piensa primero en el celular, con botones de al menos 44 px.

**Reglas de negocio actuales** (son propuestas y falta que el dueño las confirme):
- Delivery S/ 4, gratis desde S/ 45.
- Combo almuerzo: plato de fondo + limonada clásica (+S/ 6, ahorra S/ 3).
- Combo tarde: café caliente + porción de torta, S/ 22.
- Combo antojo: wrap o sándwich + frappé 12 oz (−S/ 3).
- Promo por día; el miércoles, el 2.º frappé de 12 oz sale a mitad de precio (se calcula sola en el carrito).
- Agregados: 16 oz, versión fría +2, helado +3, fruta +1, masa madre +2, papas +2, adicionales +1, extra carne +8, matcha +3, jarabes +2.

**Fuentes de código en el chat** (en `/home/claude` del entorno de Claude; no persisten entre chats): `coffit_v6_template.html` (plantilla con los marcadores `__MENU__`, `__PHOTOS__`, `__PIDX__`, `__LOGO_WORD__`, `__LOGO_SEAL__`), `menu.json`, `img/`, `build_img.py`. **En un chat nuevo, partir del `index.html` del repo o de la carpeta**, que ya trae todo incluido.

---

## 5. Datos del negocio (base para decidir)

**TumiSoft** (caja), del 4 mar al 15 nov 2026:
- **Ventas:** 6,949 comprobantes por **S/ 402,650**. Ticket promedio S/ 58 (mediana S/ 48). Unos 27 comprobantes y ~S/ 1,600 por día.
- **Por día de la semana:** domingo S/ 2,100, sábado 1,768, viernes 1,682, lunes 1,608, jueves 1,517, martes 1,499 y **miércoles 1,040** (el día más flojo).
- **Pago:** Yape 59 %, mixto 21 %, efectivo 17 %, tarjeta menos de 3 %.
- **Mesa vs. para llevar:** desde julio, ~93 % de los comprobantes son de mesa, pero se cobraron 5,926 tappers (mucho para llevar dentro de cuentas de mesa).
- **Más vendidos:** Waffle tropical, Wrap de pollo, Pollo a la plancha, Chicken katsu, Club sándwich, Ensalada de frutas, Jugo de fresa (el que más unidades vende), Desayuno al paso, Jugo surtido, Frapuccino caramel, Crepé Nutella, Tequeños y Torta de chocolate.
- **Concentración:** de 275 productos, 31 hacen el 50 % de la venta y 80 hacen el 80 %. Hay 87 productos con menos de S/ 200 en 8 meses.
- **Límite:** la hora de TumiSoft es la de emisión del comprobante (53 % entre las 22:00 y las 23:59), así que no sirve para saber las horas pico reales.

**Excel de gastos** (`GASTOS COFFIT 2026.xlsx`):
- Ventas de ~S/ 1,780 por día en nov 2026 (+32 % frente a nov 2025).
- En 7 meses, los gastos (~S/ 245k) son iguales a los ingresos (~S/ 244k): punto de equilibrio. Los gastos personales salen de la misma caja.
- Insumos ~28–40 % y personal ~28 % de la venta. El 24 % de los gastos no tiene detalle.
- **Meta +15–20 %** = +S/ 270–360 por día, unos S/ 8,000–10,700 más al mes en ventas y ~S/ 4,800–6,900 más de utilidad al mes.

**Docs de análisis en claude.ai:**
- Estudio de mercado de delivery en Virú: https://claude.ai/artifact/VUtk6e3MvM4PxxfSr4ogU4
- Análisis financiero y plan de la app: https://claude.ai/artifact/TQct6Dmd8RQjpS96zmB3pg

---

## 6. Archivos en la PC de David

Carpeta: `D:\POKERSTUDY\DvidVigo\DOCUMENTOS DE CLAUDE\CLAUDA_APP_VIRU\`
- `CARTA_COFFIT_NOVIEMBRE_2026.pdf`: carta oficial. El PDF tiene precios antiguos escondidos bajo los nuevos; valen los precios visibles.
- `carta_coffit_app.csv`: los 188 productos con los precios visibles.
- `GASTOS COFFIT 2026.xlsx`: gastos e ingresos diarios por mes.
- `FOTOS_CARTA\`: 51 fotos originales en PNG, más `fotos_app_cuadradas.zip`.
- `PNG LOGOS_LP.WB.png`: logo real (negro, fondo transparente).
- `coffit-app\index.html`: **la versión para subir a GitHub**.
- `coffit_app_v1…v6.html`: historial de versiones.
- `COFFIT_APP_PROYECTO.md`: este archivo.

Git está instalado en `C:\Program Files\Git`.

---

## 7. Pendientes

**Datos que debe confirmar el dueño:**
- [ ] Número de Yape/Plin para pagar (en la app quedó sin número a propósito).
- [ ] Precios de combos y delivery (S/ 4, gratis desde S/ 45).
- [ ] Promos por día, sobre todo la del miércoles.
- [ ] Horario real por día (hoy dice "abierto hasta las 11 p. m.", que es un supuesto).
- [ ] Fotos de los productos muy vendidos que no tienen foto: pollo a la plancha, ensalada de frutas, jugo surtido, frapuccino caramel, tequeños, batido de fresa, lomo saltado, ensalada César, cheesecakes y tiramisú.
- [ ] Confirmar si la foto turquesa es "Espuma de mar" y la roja es "Iced tea frutos rojos".
- [ ] Decidir qué hacer con los 87 productos de poca venta.
- [ ] Costo por producto (para armar combos sin perder margen).

**Técnicos:**
- [ ] **Subir la V6 a GitHub:** hoy https://coffit-app.vercel.app sigue con la V5, sin Supabase. El archivo correcto ya está en `coffit-app\index.html`.
- [ ] Instalar la Claude GitHub App en el repo para que Claude pueda hacer push.
- [ ] Panel interno para Coffit: ver pedidos nuevos y cambiar su estado.
- [ ] Club Coffit en Supabase (sellos por celular, no en el celular del cliente).
- [ ] Separar las fotos del HTML (Supabase Storage o carpeta `/img` en el repo) para que cargue más rápido.
- [ ] Dominio propio (opcional) y revisar si pasar a un plan pagado de Vercel cuando haya uso comercial.

---

## 8. Registro de cambios

| Fecha | Versión | Cambio |
|---|---|---|
| 16 nov 2026 | V1 | Primer borrador: carta del PDF, más vendidos según TumiSoft, combos, agregados, carrito, envío por WhatsApp. |
| 16 nov 2026 | V2 | Navegación por secciones: grilla de 9 secciones, pestañas fijas, "+" directo con contador, menú inferior (Inicio · Carta · Club · Pedido). |
| 16 nov 2026 | V3 | 51 fotos sacadas del PDF a máxima resolución (62 productos con foto). |
| 16 nov 2026 | V4 | Logo real de Coffit arriba y sello en el Club. |
| 16 nov 2026 | V5 | Botón flotante de WhatsApp; número oficial 944 667 516; se quitó el número del pago con Yape. Publicada en https://coffit-app.vercel.app (GitHub + Vercel). |
| 16 nov 2026 | V6 | Supabase: tablas `productos` (188) y `pedidos` con RLS; la app lee la carta de Supabase y guarda cada pedido. **Pendiente de subir a GitHub.** |

---

## Dónde vive este archivo
1. **Proyecto de claude.ai "CREAR APPS"**, como documento `COFFIT_APP_PROYECTO.md`. Se lee en cada chat nuevo del proyecto.
2. **PC de David:** `CLAUDA_APP_VIRU\COFFIT_APP_PROYECTO.md`.
3. **Repo GitHub,** como `CLAUDE.md` (cuando Claude tenga permiso de push o David lo suba).
