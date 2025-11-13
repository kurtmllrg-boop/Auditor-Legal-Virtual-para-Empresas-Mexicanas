MARCO LÓGICO DEL MOTOR JURÍDICO 1.0

Proyecto: Auditoría Legal Automatizada para Empresas Mexicanas
Autor: Kurt Muller Gil

1. Propósito del motor jurídico

El motor jurídico es un sistema experto determinístico que hace tres cosas:

Recibe hechos sobre una empresa (tipo de sociedad, número de empleados, etc.).

Compara esos hechos con un conjunto de normas representadas en lógica.

Devuelve un conjunto de obligaciones aplicables y su nivel de riesgo (cumplidas / no cumplidas / no aplica).

Este documento define el lenguaje interno del motor:

cómo se representa una norma,

cómo se representa una empresa,

y cómo decide qué norma manda cuando hay conflicto.

2. Objeto NORMA — Modelo lógico de una norma

Una NORMA en el sistema no es un texto; es un objeto lógico con campos fijos.

2.1. Campos mínimos de una NORMA

Propuesta de estructura:

id

Identificador único interno.

Ej.: "NORM_LAB_CONTRATO_ESCRITO".

materia

Rama del derecho: "laboral", "societario", "datos_personales", "comercial", etc.

jerarquia

Nivel normativo: "constitucion", "ley", "reglamento", "norma_oficial", "circular", etc.

especialidad

Qué tan específica es: "general", "especial", "ultraespecial".

Sirve para aplicar el principio de lex specialis.

ambito_espacial

"federal", "estatal", "municipal", o incluso nombre del estado.

Ej.: "federal", "Jalisco".

ambito_material

Tema concreto: "relaciones_laborales", "constitucion_sociedades", "proteccion_datos", etc.

sujeto_obligado

A quién va dirigida: "empresa", "patron", "responsable_datos", "proveedor_servicios", etc.

condicion_aplicacion

Regla lógica que dice cuándo aplica la norma.

Ej.: "empresa.tiene_empleados == true".

conducta_exigida

Acción u obligación concreta.

Ej.: "formalizar_contrato_escrito", "elaborar_aviso_privacidad".

tipo_norma

"obligacion", "prohibicion", "facultad", "definicion", etc.

Para el motor de cumplimiento, nos centramos en "obligacion" y "prohibicion".

consecuencia_incumplimiento

Tipo general: "multa", "nulo", "responsabilidad_laboral", "inspeccion".

severidad_riesgo

"alto", "medio", "bajo" (valor jurídico-técnico, no probabilístico).

fuente_formal

Estructura de referencia legal:

{
  "ordenamiento": "LFT",
  "articulo": "24",
  "fraccion": null
}


vigencia_desde / vigencia_hasta

Fechas de inicio y fin de vigencia (si aplica).

notas_interpretativas (opcional)

Comentarios para juristas y futuras mejoras del motor.

2.2. Ejemplo de NORMA en formato lógico
{
  "id": "NORM_LAB_CONTRATO_ESCRITO",
  "materia": "laboral",
  "jerarquia": "ley",
  "especialidad": "general",
  "ambito_espacial": "federal",
  "ambito_material": "relaciones_laborales",
  "sujeto_obligado": "patron",
  "condicion_aplicacion": "empresa.tiene_empleados == true",
  "conducta_exigida": "formalizar_contrato_escrito",
  "tipo_norma": "obligacion",
  "consecuencia_incumplimiento": "multa_STPS",
  "severidad_riesgo": "alto",
  "fuente_formal": {
    "ordenamiento": "LFT",
    "articulo": "24",
    "fraccion": null
  },
  "vigencia_desde": "1970-05-01",
  "vigencia_hasta": null,
  "notas_interpretativas": "La falta de contrato escrito no elimina la relación laboral, pero genera riesgos probatorios y multas."
}


Este es el “átomo” jurídico con el que va a trabajar el motor.

3. Objeto EMPRESA / CASO — Modelo de hechos

El motor no trabaja con “empresas” vagas, sino con un objeto EMPRESA que contiene hechos relevantes.

3.1. Campos mínimos del objeto EMPRESA (V1)

Propuesta inicial (se puede ampliar después):

tipo_sociedad

Ej.: "SA_DE_CV", "SAPI", "SAS", "persona_fisica", etc.

num_empleados

Número entero.

tiene_empleados

true / false. (Derivado de num_empleados > 0, pero puedes guardarlo ya calculado.)

ubicacion_fiscal

Estado: "Jalisco", "CDMX", etc.

ambito_operacion

"local", "nacional", "internacional".

giro_principal

"comercio", "servicios_profesionales", "manufactura", etc.

trata_datos_personales

true / false.

tiene_sitio_web

true / false.

tiene_establecimientos_fisicos

true / false.

usa_subcontratacion

true / false.

3.2. Representación simple de EMPRESA
{
  "tipo_sociedad": "SA_DE_CV",
  "num_empleados": 25,
  "tiene_empleados": true,
  "ubicacion_fiscal": "Jalisco",
  "ambito_operacion": "nacional",
  "giro_principal": "servicios_profesionales",
  "trata_datos_personales": true,
  "tiene_sitio_web": true,
  "tiene_establecimientos_fisicos": true,
  "usa_subcontratacion": false
}


El motor cruza este objeto con el conjunto de NORMA.

4. Principios de decisión del motor (conflictos entre normas)

Aquí metemos los principios jurídicos universales que tu motor va a respetar.

Los escribimos de forma operativa, no doctrinal.

4.1. Lex superior (jerarquía normativa)

Regla interna:

Si dos normas se contradicen y tienen el mismo ámbito (materia, espacio, sujeto), prevalece la que tenga mayor jerarquia.

Orden sugerido de jerarquía (de mayor a menor):

Constitución

Tratado internacional

Ley federal / ley local (según caso)

Reglamento

Norma oficial mexicana

Lineamientos / circulares / criterios administrativos

En pseudocódigo:

si conflicto(norma_A, norma_B):
    si norma_A.jerarquia > norma_B.jerarquia:
        aplica norma_A
    si norma_B.jerarquia > norma_A.jerarquia:
        aplica norma_B


(“>” aquí representa “mayor jerarquía”, no numérico literal.)

4.2. Lex specialis (especialidad)

Regla interna:

Si dos normas tienen la misma jerarquía pero una es más específica, gana la más específica.

Implementación simple:

especialidad → "general" < "especial" < "ultraespecial".

si conflicto(norma_A, norma_B) y misma_jerarquia:
    si norma_A.especialidad > norma_B.especialidad:
        aplica norma_A


Ejemplo:
Norma general: obligaciones laborales de todo patrón.
Norma especial: obligaciones de patrones en construcción → gana la especial para empresas de construcción.

4.3. Lex posterior (ley posterior)

Regla interna:

Si dos normas tienen la misma jerarquía y el mismo nivel de especialidad, prevalece la más reciente en vigencia.

Usando vigencia_desde:

si conflicto(norma_A, norma_B) y misma_jerarquia y misma_especialidad:
    aplica la de mayor vigencia_desde

4.4. Competencia y ámbito espacial

Antes de aplicar una norma, el motor verifica:

que ambito_espacial de la norma sea compatible con ubicacion_fiscal / ambito_operacion de la empresa;

que materia coincida con el módulo activo (laboral, societario, etc.).

Regla:

norma_aplicable = (
    coincide_ambito_espacial(norma, empresa) &&
    coincide_materia(norma, modulo_actual) &&
    evalua_condicion_aplicacion(norma.condicion_aplicacion, empresa)
)


Si norma_aplicable es false, esa norma no entra al juego.

4.5. Temporalidad (vigencia)

El motor solo considera aplicables las normas cuya vigencia cubra la fecha de análisis.

fecha_analisis = hoy()

norma_vigente = (
   norma.vigencia_desde <= fecha_analisis &&
   (norma.vigencia_hasta == null || norma.vigencia_hasta >= fecha_analisis)
)


Si norma_vigente es false, la norma queda excluida (o archivada para histórico).

5. Output del motor: qué devuelve

Para cada EMPRESA, una vez procesadas las normas aplicables, el motor genera:

5.1. Lista de obligaciones evaluadas

Cada entrada podría tener:

id_norma

aplica (true/false)

cumplida (true/false/desconocido)

severidad_riesgo

recomendacion

Ejemplo:

{
  "id_norma": "NORM_LAB_CONTRATO_ESCRITO",
  "aplica": true,
  "cumplida": false,
  "severidad_riesgo": "alto",
  "recomendacion": "Formalizar por escrito todos los contratos laborales actuales."
}

5.2. Agregación tipo “semáforo legal”

El motor puede calcular un índice global:

% de obligaciones críticas incumplidas,

número de riesgos altos, medios, bajos,

una calificación global (0–100 o rojo/amarillo/verde).

6. Qué sigue después de este marco lógico

Con este documento tú ya tienes:

el modelo de NORMA,

el modelo de EMPRESA,

las reglas de decisión que el motor debe respetar.

Los siguientes pasos naturales serían:

Definir esto en forma más técnica (por ejemplo, un esquema JSON o clases en Python).

Implementar un pequeño prototipo de motor que:

cargue normas de prueba,

reciba un objeto EMPRESA de prueba,

y devuelva la lista de obligaciones aplicables.

Sólo después, empezar a alimentar el sistema con normas reales, poco a poco.