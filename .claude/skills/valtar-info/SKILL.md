---
name: valtar-info
description: Muestra los datos oficiales de VALTAR SECURITY SYSTEMS S.A.C. (razón social, RUC, gerente, dirección, contacto, cuentas bancarias, partida registral) en formato tabla. También rellena los espacios vacíos de documentos adjuntos (Word/PDF/texto) con esta información cuando el usuario lo solicita. Úsalo cuando el usuario escriba "/valtar.info", "/valtar-info", pida los datos de Valtar, o pida completar un documento con la información de la empresa Valtar.
---

# Valtar Info

Skill que centraliza la información oficial de **VALTAR SECURITY SYSTEMS S.A.C.** para dos casos de uso:

1. **Mostrar los datos** en una tabla Markdown clara.
2. **Rellenar documentos** (Word `.docx`, PDF, texto) sustituyendo los campos vacíos por los datos correspondientes de la empresa y devolviendo el archivo resultante en el mismo formato que el original.

---

## 1. Datos oficiales de la empresa

Cuando el usuario invoque el skill sin un documento adjunto (por ejemplo escribe solo `/valtar.info` o "dame los datos de Valtar"), responde con esta tabla exacta:

| Campo | Valor |
|---|---|
| Razón social | VALTAR SECURITY SYSTEMS S.A.C. |
| CE | 003777841 |
| RUC | 20612247031 |
| Gerente | Roberto Carlos Tarazona Francisco |
| Correo | Roberto.tarazona@valtarsecurity.com |
| Dirección | Av. Javier Prado Este 1166, Ofi. 601, San Isidro, Lima – Lima |
| Código postal | 15036 |
| Celular | 902156356 |
| Página web | www.valtarsecurity.com |
| Partida registral | 15563045 |
| Asiento | A00001 |

### Cuentas bancarias

| Banco | Moneda | Cuenta | CCI |
|---|---|---|---|
| BBVA | Soles | 00110112020039369303 | 01111200020039369303 |
| BCP | Soles | 1914213864081 | 00219100421386408150 |
| BCP | Dólares | 1937135513197 | 00219300713551319711 |
| Banco de la Nación | Soles (Detracciones) | 00002220571 | — |

> Fuente canónica de los datos: `data.md` en este mismo skill. Si el usuario corrige algún dato, **actualiza `data.md`** y vuelve a generar la respuesta desde ahí — nunca inventes ni asumas valores distintos.

---

## 2. Rellenar documentos con los datos de la empresa

Cuando el usuario adjunte un archivo (Word, PDF, TXT, Markdown, etc.) y pida llenar los espacios vacíos con `/valtar.info` (o equivalente), sigue este procedimiento:

### Paso 1 — Identificar el formato del archivo
- `.docx` → Word (usa `python-docx`).
- `.pdf` → PDF (usa `pypdf` para leer; si hay que rellenar campos de formulario AcroForm usa `pypdf` o `pdfrw`; si es PDF plano sin campos, conviértelo a texto, rellena, y regenera el PDF con `reportlab` **solo si el usuario acepta perder formato visual**; si no, pide confirmación).
- `.txt` / `.md` → edición directa de texto.
- Otros → pregunta al usuario en qué formato quiere la salida.

### Paso 2 — Detectar los espacios a rellenar
Busca marcadores típicos en el documento, en este orden de prioridad:

1. **Placeholders explícitos**: `{{razon_social}}`, `{{ruc}}`, `<RUC>`, `[RAZÓN SOCIAL]`, etc.
2. **Campos de formulario** (en Word: content controls / campos; en PDF: AcroForm fields).
3. **Líneas en blanco rotuladas**: `Razón social: ________`, `RUC: .................`, `Gerente: ____`.
4. **Etiquetas seguidas de espacio vacío** en tablas (celda derecha vacía junto a una celda etiquetada).

Mapea cada etiqueta detectada a un campo de la empresa usando sinónimos:

| Etiqueta posible en el documento | Campo a insertar |
|---|---|
| Razón social, Empresa, Nombre de la empresa, Contratista, Proveedor | VALTAR SECURITY SYSTEMS S.A.C. |
| RUC, N° RUC, R.U.C. | 20612247031 |
| CE, Código empresa | 003777841 |
| Representante legal, Gerente, Gerente general, Apoderado | Roberto Carlos Tarazona Francisco |
| Correo, Email, E-mail, Correo electrónico | Roberto.tarazona@valtarsecurity.com |
| Dirección, Domicilio, Domicilio fiscal | Av. Javier Prado Este 1166, Ofi. 601, San Isidro, Lima – Lima |
| Código postal, CP, Zip | 15036 |
| Celular, Teléfono, Móvil, Contacto | 902156356 |
| Página web, Web, Sitio web, URL | www.valtarsecurity.com |
| Partida, Partida registral, Partida electrónica | 15563045 |
| Asiento | A00001 |
| Cuenta BBVA soles | 00110112020039369303 (CCI 01111200020039369303) |
| Cuenta BCP soles | 1914213864081 (CCI 00219100421386408150) |
| Cuenta BCP dólares | 1937135513197 (CCI 00219300713551319711) |
| Cuenta Banco de la Nación / Detracción | 00002220571 |

Si aparece una etiqueta que **no** mapea a ningún campo conocido, déjala sin llenar y repórtala al final.

### Paso 3 — Rellenar preservando el formato
- No cambies fuentes, tamaños, colores, encabezados, pies de página ni tablas del archivo original.
- Solo reemplaza el texto del placeholder/campo vacío.
- En Word, reemplaza a nivel de `run` cuando sea posible para no romper el estilo.
- En PDF con AcroForm, establece el valor del field y mantén el resto intacto.

### Paso 4 — Devolver el archivo
- Guarda el resultado con el sufijo `_valtar` antes de la extensión (ej.: `contrato.docx` → `contrato_valtar.docx`).
- Mantén el mismo formato/extensión que el original salvo que el usuario pida otro.
- Al final, muestra un resumen:
  - Campos rellenados (lista).
  - Campos no mapeados que quedaron vacíos (si los hubo), para que el usuario decida.

### Paso 5 — Verificación final
Antes de entregar el archivo, confirma que:
- No quedaron placeholders originales sin reemplazar (salvo los que avisaste).
- Los valores insertados coinciden exactamente con los de `data.md`.
- El archivo abre correctamente (si hay entorno para validarlo).

---

## Archivos del skill

- `SKILL.md` — este archivo, las instrucciones.
- `data.md` — fuente única de verdad con los datos de la empresa. Edita aquí si algún dato cambia.
